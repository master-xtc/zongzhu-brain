# 技能_开发建站_小程序canvas绘制SVG贴纸转PNG管线_v1

## 触发场景
小程序页面需要把 SVG 贴纸/图标画进 canvas 2d（分享卡、海报合成、打卡图），或任何"view 能渲染 SVG 但 canvas drawImage 画不了"的场景。

## 结论先行
- 小程序 canvas 2d（wx.createImage + ctx.drawImage）**不支持 SVG 数据/文件**；view 的 background-image 支持 SVG data URI，两者不能混用。
- 正解：离线把 SVG 栅格化为 PNG 资产（192×192 足够清晰），板面预览与 canvas 绘制共用同一 PNG，保证图文一致。

## 步骤
1. 检查本机 SVG 栅格化工具：node sharp / @resvg/resvg-js / python cairosvg / ImageMagick；无则 `npm i @resvg/resvg-js`（预编译，3 秒装完，无原生编译）。
2. 独立工作目录建脚本（勿把 node_modules 放进小程序工程）：读 data/*.json 的 svg 字段 → `new Resvg(svg, { fitTo:{mode:'width',value:192}, background:'rgba(255,255,255,0)' })` → `render().asPng()` → 写 `assets/stickers/{id}.png`。
3. 页面 view 预览：inline style `background-image:url('/assets/stickers/{{id}}.png')`（根绝对路径）。
4. canvas 绘制：`wx.createImage()` 加载同一 PNG 路径 → `ctx.drawImage(img, ...)`；多张贴纸用 Promise 链串行加载。

## 验收标准
- 30 张贴纸 PNG 与源 JSON id 一一对应（脚本断言）；PNG 透明背景（无白底矩形）；总计 ≤150KB 可接受。
- canvas 导出图与板面预览视觉一致（同一 PNG 源）。
- 静态扫描含"贴纸 PNG 资产数 = 源清单数 + id 一一对应"检查项。

## 适用模型
任意 Codex/下属窗口；resvg-js 需本机 node ≥14。

## 踩坑记录
- canvas 2d 的隐藏节点 CSS 尺寸压小（10px 离屏）即可，背板用 node.width/height 设置；CSS 大尺寸会触发"375 无超宽"扫描误报。
- 贴纸 view 自带 transform（translate+rotate）时禁用 .press（:active transform 顶掉定位），改自定义 hover-class 调 opacity。
- 换行/重复属性事故：批量改 WXML 用行级 List 手术并立即 Get-Content 目检，勿依赖幂等 Replace。