---
name: query-weather
description: >-
  当用户提到天气并希望查询天气时触发此 Skill；
  触发关键词：天气（必须包含）、weather、查天气、问天气、看天气、今日天气、明天天气、后天天气；
  输入需要：地点名称（可含错别字，高德地图自动纠错）；输出为：实时天气+24小时预报+7天预报+日出日落；
  不适用于：非天气查询场景。
  典型用户表达："xx天气" "北京天气" "xx山天气怎么样" "帮我查天气" "明天杭州天气"
version: 8.0.0
metadata:
  hermes:
    tags: [weather]
    category: utility
required_commands: [curl]
---

# 通用天气查询

## 前置条件
1. 安装 curl
2. 已配置 `references/keys.json`

## 流程

### 第一步：加载密钥
读取 `references/keys.json`，获取：
- `amap.key`：高德地图 API Key
- `qweather.api_key`、`qweather.host`：和风天气凭证

### 第二步：地名纠错与坐标查询
使用高德地图地理编码 API：
```
GET https://restapi.amap.com/v3/geocode/geo
  ?key={amap.key}
  &address={用户输入的地名}
  &city={城市名（可选）}
```

**处理逻辑：**
- count == 0：无法识别，反问用户"请问您说的是哪个地方？"
- count == 1：使用返回的坐标（格式：经度,纬度）
- count > 1：返回候选列表，反问用户选择

### 第三步：并行查询多个天气数据源
**同时调用** 以下接口：

#### 1. 城市实时天气
```
GET https://{qweather.host}/v7/weather/now
  ?location={经度},{纬度}
```

#### 2. 格点实时天气（精确位置）
```
GET https://{qweather.host}/v7/grid-weather/now
  ?location={经度},{纬度}
```

#### 3. 24小时预报
```
GET https://{qweather.host}/v7/weather/24h
  ?location={经度},{纬度}
```

#### 4. 7天预报（含日出日落）
```
GET https://{qweather.host}/v7/weather/7d
  ?location={经度},{纬度}
```

#### 5. 空气质量（可选，失败不中断）
```
GET https://{qweather.host}/v7/air-quality/now
  ?location={经度},{纬度}
```

#### 6. 天气预警（可选，失败不中断）
```
GET https://{qweather.host}/v7/weather/warning
  ?location={经度},{纬度}
```

**认证方式：** 请求头 `X-QW-Api-Key: {qweather.api_key}`

**⚠️ curl 必须加 `--compressed` 参数：**
```bash
curl -s --compressed "https://{host}/v7/weather/now?location=..." -H "X-QW-Api-Key: ..."
```

### 第四步：空气质量/天气预警（可选，失败不中断）

## 输出格式

```
📍 {地名}（{经度},{纬度}）
🕐 更新时间：{updateTime}

━━━ 🌡️ 实时天气 ━━━
  ━━ 格点（精确位置）━━
  🌡️ {grid.temp}℃（体感 {grid.feelsLike}℃）
  🌤️ {grid.text} | 💨 {grid.windDir} {grid.windScale}级
  💧 {grid.humidity}% | 🌧️ 降水 {grid.precip}mm
  ☁️ 云量 {grid.cloud}% | 👁️ 能见度 {grid.vis}km

  ━━ 城市（最近城市）━━
  🌡️ {city.temp}℃（体感 {city.feelsLike}℃）
  🌤️ {city.text} | 💨 {city.windDir} {city.windScale}级

━━━ ⏰ 24小时预报 ━━━
{next24h_summary}

━━━ 📅 7天预报 ━━━
{daily_forecast_table}

━━━ 🌅 日出日落 ━━━
今天：🌅 {today.sunrise} 🌇 {today.sunset}
明天：🌅 {tomorrow.sunrise} 🌇 {tomorrow.sunset}

{f 空气质量（如有）}
{f 天气预警（如有）}

⚠️ 天气多变，仅供参考
```

### 24小时预报格式
取下一个整点时刻的数据，格式：
```
{time} | {temp}℃ | {text} | {windDir} {windScale}级 | 降水{pop}%
```

### 7天预报表格格式
| 日期 | 天气 | 最高/最低 | 风力 |
|------|------|-----------|------|
| 4/29 | 阴→多云 | 16°/11° | 北风1-3级 |
| ... | ... | ... | ... |

## 失败处理

| 场景 | 处理方式 |
|------|---------|
| 高德地图无法识别地名 | "抱歉，我无法识别「{输入地名}」，请确认地名或提供更详细的地址" |
| 格点天气失败 | 只显示城市天气，标注"格点数据获取失败" |
| 城市天气失败 | 只显示格点天气，标注"城市数据获取失败" |
| 空气质量/预警失败 | 跳过该模块，不显示 |
| 两个都失败 | "天气数据获取失败，请稍后重试" |
| 用户未提供地点 | "请告诉我您要查询的地点名称" |

## 常用地点预设坐标

| 地点 | 坐标 |
|------|------|
| 真宝顶 | 110.830974, 26.142621 |
