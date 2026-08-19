# 开发建站_PixsoMCP设计转代码链路

## 技能名
Pixso MCP 设计转代码链路（设计稿数据级转码）

## 触发场景
- 建站/接单时 UI 反复改样式、还原度低、被"编程AI直接写UI"拖进度
- 需要"设计稿 → 高还原前端代码"的完整链路
- 习总做海报/网站/小程序交付，想提升设计转码效率与质量

## 步骤
1. 分工定位：编程 AI（Claude Code/Cursor）逻辑强审美弱；设计 AI（Pixso AI）审美强——UI 交给设计 AI，代码交给编程 AI
2. 出稿：给 Pixso AI 一句提示词（示例："复刻XX App + 新野兽主义风格"），生成带完整图层数据的设计稿
3. 启用 Pixso MCP：Pixso 客户端（≥2.2.0）文件菜单开启 Pixso MCP，本地服务器 http://localhost:3667/mcp
4. 挂载到编程 AI：`claude mcp add --transport http pixso-desktop http://127.0.0.1:3667/mcp`（Cursor/VS Code/Windsurf 同理）
5. 生成：丢设计稿链接 + 一句"生成HTML代码"，得到高还原前端（圆角/间距/颜色变量对齐）
6. 验证：Web 后台/移动端 APP 各跑一遍还原度；样式变量抽查

## 验收标准
- 生成界面还原度高：圆角、间距、颜色变量与设计稿对得上
- 全程不靠"截图+AI看图猜"，走结构化数据通道
- 挂载命令写入本地配置（~/.claude.json）成功并有端口监听确认

## 适用模型/工具
- Pixso AI + Pixso MCP（官方）；Claude Code / Cursor / VS Code / Windsurf
- 替代品注意：Stitch（评论区称免费）可对比实测；价格与效果争议需自测
- 方法论同构：小说/内容 MCP 工程同样优先做"结构化数据通道"

## 使用记录
- 2026-08-14：抖音视频《独立开发最大的坑：逻辑写得飞起，UI丑得想哭？》拆解产出（孙同学玩AI），已写入分析报告 分析_PixsoMCP设计转代码UI工作流_20260814.md
