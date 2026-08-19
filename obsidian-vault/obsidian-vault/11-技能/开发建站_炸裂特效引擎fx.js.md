# 开发建站_炸裂特效引擎fx.js

## 技能名
炸裂特效引擎（fx.js · 动漫式能量感动效）

## 触发场景
- 网站需要"无与伦比的吸引力"：高光时刻炸裂动效（开屏/打卡/成就/视图切换）
- 从含蓄动效升级到撕裂/碎裂/粒子级特效
- 任何品牌叙事站、作品集、私用工具兼展示站

## 步骤
1. 新建 `assets/js/fx.js`，暴露 `window.FX`：init/burst/shockwave/speedLines/shake/flash/rippedScreen/shatter/textBurst/isDesktop
2. 引擎自注入样式（style 标签）：冲击波环/速度线/闪白/撕裂层/碎块/震动 keyframes
3. 全局粒子层：fixed canvas（z-index 90），粒子池上限 150，DPR 上限 2，rAF 循环，仅在有粒子/光轨时运行
4. 高光点接入：boot 首屏 `v4FxOnce("home")`（首屏不走 switchView！）+ switchView 内 `v4FxOnce(name)`；celebrate 增强（burst+shockwave+速度线+PERFECT 判定）；splash 撕裂
5. 数据卡碎裂：v4FxOnce 对 `.stats-grid .stat-card` 延迟 420ms 触发，`el._fxShattered` 防重
6. 文字爆破：Canvas fillText 取样（字体取 computedStyle），每 3px 采样，粒子飞散→GSAP 弹性聚合
7. 移动端降级：`pointer: fine and min-width 768px` 判定，窄屏/触屏只保留辉光冲击波弹性
8. 性能护栏：prefers-reduced-motion 全关；burst 后 2.2s 自动停循环

## 验收标准
- 控制台 0 错误；390px 无横向溢出
- 桌面首屏数据卡碎裂触发（检查 `_fxShattered` 持久标记，瞬时 DOM 已自清理）
- 移动端不卡顿（粒子/光轨关闭）
- 数据层零改动；SW 缓存版本号升级并注册 fx.js
- 特效元素 0.5-0.9s 自动清理（无 DOM 泄漏）

## 适用模型
Codex 直接能力（GSAP/Canvas 均原生掌握）；视觉终验须视觉下属/习总目视

## 踩坑记录
- 首屏不走 switchView → boot 必须补 `v4FxOnce("home")`，否则首屏动效缺失
- 多步批量替换脚本锚点链式依赖会互相破坏（A 步骤插入改变了 H 锚点）→ 用独立锚点或先 dry-run
- 移动端 Playwright 390px 实际 clientWidth 380（滚动条），溢出判断用 scrollWidth>clientWidth
