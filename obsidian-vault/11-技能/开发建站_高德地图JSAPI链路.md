# 技能：开发建站_高德地图JSAPI链路

## 技能名
高德地图 JS API 2.0 建站链路（场景A：门店地图展示页）

## 触发场景
- 网站需要真实地图：门店/场馆定位、服务范围圈、地址解析、一键导航
- 习总健身站"找到我们"地图页及其升级（场景B 调度台）
- 接单类地图展示需求

## 步骤
1. 查库：本技能 + 提示词库《提示词_建站_健身地图调度台_v1.md》+ 教程《教程_健身网站地图调度台_从零到一_20260815.md》。
2. Key：高德控制台创建"浏览器端(JS API)"应用拿 Key；Key 只写 config.js，不写文档/记忆。
3. 三文件结构：config.js（key + securityJsCode 预留位）、data.js（门店数据，换店只改它）、index.html（页面）。
4. 动态加载：`new AMap.Map` 前先动态插入 script（`https://webapi.amap.com/maps?v=2.0&key=KEY&plugin=AMap.Scale,AMap.ToolBar,AMap.Geocoder`）；config.js 先于页面加载。
5. 必做降级：Geocoder 回调失败/超时 8s → 用 data.js 兜底坐标渲染，页面永不白屏。
6. 交互：Marker(content 自定义荧光绿图钉) + Circle(服务半径) + InfoWindow(店名/地址/电话/时间+开始导航) + 导航链接 `https://uri.amap.com/navigation?to=lng,lat,店名&mode=car&coordinate=gaode`。
7. 验证：本地 `python -m http.server 8123` + CDP 真实浏览器（headless chrome --remote-debugging-port + Node WebSocket）截图 OCR；移动端 390px 与桌面端各一次。

## 验收标准
- 移动端 390px 无横向滚动、地图不黑屏、POI 文字可 OCR 到（证明瓦片渲染）；
- 点击 Marker 弹 InfoWindow；导航链接正确（to=纬度,经度,店名）；
- 无 console 异常；Key 只出现在 config.js。

## 踩坑记录（2026-08-15 实测）
- Geocoder 是插件：不写进 plugin 参数会报 "AMap.Geocoder is not a constructor"。
- Key 开启"安全密钥"校验后：地图瓦片正常，但服务接口（Geocoder）报 INVALID_USER_SCODE(10008) 且回调不触发（无报错，只挂起）——排查法：CDP Network.getResponseBody 抓 geocode 响应体即见错误 JSON；解法=config.js 填 securityJsCode 或前端 window._AMapSecurityConfig。
- JS 平台 Key 调 REST 接口（restapi.amap.com/v3/geocode/geo）返回 USERKEY_PLAT_NOMATCH，属正常，别误判为 Key 失效。
- headless chrome 截图时机坑：Geocoder 是真实网络回调，--virtual-time-budget 截图会截到"解析中"；要用 CDP 轮询 DOM 等回调/兜底后再截图。

## 使用记录
- 2026-08-15 首建：健身站"找到我们"地图页场景A交付（D:\健身网站\map-dispatch），移动端+桌面端验证通过，截图与复盘已回填。
