# 提示词_视觉能力接入_通用MCP配置_v1

## 名称
视觉能力接入 · Qwen-MM-Plugins 通用 MCP 配置（Claude Code / Hermes 双通道）

## 用途
给任意大模型外挂多模态"眼睛"：读图 / 读视频 / 读文档 / 3D / OCR 文字提取。核心链路 = MCP stdio 注册（qwen-mm-plugins-core，零 API key）+ skill 说明 + 底座模型能力兜底判断。

## 适用场景
- 给 Claude Code 或 Hermes 等 Agent 装视觉能力
- DeepSeek 等纯文本模型需要"看到"图片/视频内容（走 OCR 兜底）
- 视频抽帧 OCR、图片文字提取、图文一致性核对
- 新员工/新 Agent 视觉通道配置

## 变量位
- `{AGENT}`：目标 Agent（claude / hermes）
- `{CONFIG_PATH}`：配置文件路径（Claude: `~/.claude.json`；Hermes: `$HERMES_HOME/config.yaml`）
- `{SKILL_DIR}`：skill 落点（Claude: `~/.claude/skills/`；Hermes: `$HERMES_HOME/skills/`）
- `{UV_BIN}`：uv 安装路径（本机 `D:\AI工具\uv`）

## 正文模板
1. 确认 uv 已装（装 D 盘，PATH + UV_CACHE_DIR 配好）。
2. 注册 MCP stdio（三要素缺一不可）：
   - command: `uvx`
   - args: `["--from", "qwen-mm-plugins[core] @ git+https://github.com/QwenLM/Qwen-MM-Plugins.git@qwen-mm-plugins-core-v1.0.1", "qwen-mm-plugins-core"]`
   - env（Hermes 必写，Claude 可选）: `UV_CACHE_DIR: 'D:\AI工具\uv\cache'`
3. 复制 skill：core 仓库 SKILL.md → `{SKILL_DIR}`，按 Agent 技能格式放分类目录。
4. 验收：`hermes mcp test {NAME}` 应 5s 内连上、7 工具齐全；Claude 用 `read_image` 实测。
5. 底座判断：若 Agent 底座为多模态（Claude）→ read_image 直接描述画面；若为纯文本（DeepSeek）→ read_image 只返回图像数据，必须配 OCR（rapidocr-onnxruntime，`uv pip install --python <venv-python> rapidocr-onnxruntime`）提取文字，并如实说明"OCR 兜底，只能读文字"。

## 变体
- V1.1 纯文本底座视觉链路：MCP(read_image/read_video) 仅作预处理，实际内容靠 rapidocr OCR + 像素统计（PIL）如实汇报。
- V1.2 多模态底座视觉链路：Claude Code 直读，read_image/read_video/visualize 全覆盖，无需 OCR。
- V1.3 图像量产链路：豆包 ark（doubao-vision）走调度网关 `--provider ark --mode vision`，待 Key。

## 验收标准
- `hermes mcp test` Connected ≤ 10s，7 工具发现
- skill 在 `skills list` 显示 enabled
- OCR 单图识别 ≤ 10s，输出与画面文字一致（抽查 1 张）
- 交付说明里注明底座类型与所用链路，不假装看图

## 使用记录
- 2026-08-13：Claude Code + Hermes 落地（本机），实测 read_image 5s / OCR 4.8s 通过；复盘见 growth\reviews\20260813_抖音DeepSeek视觉能力原理分析.md
- 2026-08-14：Hermes 值班带图自检实战通过（贵州宣传片 3 帧：read_image+OCR+PIL 三源交叉，OCR f01=4/f02=49/f03=34 条）；一键脚本 D:\AI员工体系\scripts\视觉值班自检_Hermes读图_20260814.ps1；复盘见 growth\reviews\复盘_视觉自检_贵州宣传片抽帧_20260814.md

- 2026-08-14 补记：豆包视觉链路已通——Doubao-Seed-2.1-turbo 为多模态模型（文本+视觉一体），无需单独 vision 接入点；网关 `--provider ark --mode vision --model ep-xxx` 直接读图（实测 28s/2628 tokens，语义级描述）。Vision 专用模型（1.5-vision-* 等）旧版本已全部下线，新系列直接复用多模态 Seed 模型。