# 提示词_抖音视频内容分析_链接转写抽帧OCR_v1

## 名称
抖音视频内容深度分析（链接 → 转写 → 抽帧 OCR → 分析报告）

## 用途
输入抖音分享链接，一键产出结构化视频分析报告（元信息 / 口播全文 / 画面核查 / 原理拆解 / 可借鉴点）

## 适用场景
- 习总发抖音链接要求"分析视频内容"（教程、信息差、AI 玩法、文案拆解）
- 需要图文一致性核查、画面文字提取（skill 配置、终端命令、界面截图）
- 竞品/素材拆解：收藏率高的干货视频，提取可复用的表达结构与操作流程

## 变量位
- {链接}：抖音分享文本（含 v.douyin.com 短链）
- {分析侧重点}：默认全量；可指定"只讲原理/只提取文案/只做画面核查/评论区要点"
- {输出目录}：默认当前窗口主目录（D:\抖音提取文案和执行）

## 正文模板
1. 解析链接：用 playwright 打开分享页，从页面 JSON（如 share_router.json / _ROUTER_DATA）取视频 ID、页面地址、作者、发布时间、点赞/收藏/评论/分享数
2. 下载视频与音频（yt-dlp 或既有 dl 脚本），输出 {视频ID}.mp4 / .wav
3. 转写：faster-whisper 生成 transcript_{视频ID}.txt/.json，人工校正术语误识别
4. 抽帧：ffmpeg 每 2 秒一帧到 frames_{视频ID}/，文件名 frame_001.png 起
5. OCR：rapidocr_onnxruntime 批量提取帧文字，输出 ocr_frames_{视频ID}.json
6. 评论区要点：从页面数据提取代表性评论（模型方案/效果对比/原理复述等）
7. 整合为分析报告（结构见验收标准），落盘窗口主目录
8. 沉淀：任务复盘 → growth\reviews\；提示词使用记录更新

## 变体
- 变体 A（纯文案提取）：只做步骤 3，输出口播全文 + 文案结构拆解
- 变体 B（画面核查）：只做步骤 4-5，输出逐帧文字表 + 图文一致性结论
- 变体 C（竞品拆解）：侧重评论区 + 收藏率分析 + 可借鉴点

## 验收标准
- 报告含 8 节：元信息表 / 口播全文 / 画面核查（帧表）/ 原理拆解 / 评论区要点 / 可借鉴点 / 结论 / 遗留问题
- 图文一一对应：每个关键画面有 OCR 证据，不做脑补；总助不假装"看到"画面
- 术语校正：whisper 误识别（Scout/Cloud Code/DeepSig 类）须按上下文与 OCR 双向校正
- 文件命名规范：{类别}_{对象}_{用途}_{时间}.md，产物全部落窗口主目录
- 任务完成回填复盘与使用记录

## 使用记录
- 2026-08-13：分析《我给DeepSeek添加视觉能力的原理》视频（video_7645960331184868644），产出 分析_DeepSeek视觉能力原理_20260813.md，首次使用并入库
- 2026-08-13（追加）：分析《Qwen-MM-Plugins 给 DeepSeek 装眼睛》视频（video_7672400663624295589），16s 图文型无口播，33 帧 OCR 完成，产出 分析_QwenMMPlugins给DeepSeek装眼睛_20260813.md；同时定位官方仓库 QwenLM/Qwen-MM-Plugins，支撑 Hermes 视觉接入方案
- 2026-08-13（追加）：分析《宣传视频skill分享》视频（video_7673184898613982694，短链 lHBY_vyM_Yk），34s 口播 4 段 + 17 帧 OCR；识别项目 video-shotcraft（AI video skill for Claude Code/Codex，Remotion 152 分镜 + 209 动效）；用 GitHub API 核实「4万星」实为 4.8k；产出 分析_宣传视频skill分享_video-shotcraft_20260813.md

- 2026-08-14（追加）：分析《两毛钱做出一个完整应用，面试直接乱杀》视频（video_7673457418671607092，短链 dYS199zUGyc），20s 口播 10 段 + 10 帧 OCR；识别开源项目 Prism-Shadow/penguin-harness（1,268 star，TypeScript/pnpm monorepo，文件系统当唯一真相源架构）；用 GitHub API 核实真实性；产出 分析_PenguinHarness两毛钱Agent应用_20260814.md；本次因 yt-dlp 需 cookies，改用浏览器 init script 拦截媒体 URL 直接下载，已验证可行

- 2026-08-14（追加）：分析《让你也能快速复刻大厂UI的开源项目它来了》视频（video_7636688012906073390，短链 XmwZBQrWQzQ），33s 口播 14 段 + 17 帧 OCR；定位真实仓库 VoltAgent/awesome-design-md（GitHub 实时 108K star，口播 71.7K 为拍摄时数据）；定位方法升级：code search 搜独特文件名 preview-dark.html 一次命中；产出 分析_awesome-design-md复刻大厂UI开源项目_20260814.md

- 2026-08-14（追加）：分析《独立开发最大的坑：逻辑写得飞起，UI丑得想哭？》视频（video_7599999592515834212，短链 e73TDz1w3q8），67.7s 口播 33 段 + 34 帧 OCR 597 项；首次遇到抖音音画分离（video-hvc1 无音轨，需单独下载 audio-und-mp4a 流）；识别工具链 Pixso AI + Pixso MCP（whisper 误识 Pixel 经 OCR 校正），豆包 ark vision 语义核对 4 帧与 OCR 互证；产出 分析_PixsoMCP设计转代码UI工作流_20260814.md；同步沉淀技能 开发建站_PixsoMCP设计转代码链路.md

- 2026-08-14（追加）：分析《VibeCoding大赏｜一套UI工作流，让AI审美直接拉满》视频（video_7668903548205157682，短链 OEM4BXp1JZE），5 分 30 秒干货长视频（首次）：音画分离独立抓音频流转 wav；whisper 214 段 + 33 帧 OCR 558 项逐条双向校正；GitHub API 核实 ui-ux-pro-max-skill（116K star）与 taste-skill（76K star）真实，interface-design/impeccable 未命中如实标注；产出 分析_UI工作流Skill_前置路由分发_20260814.md；方法论（前置路由分发/判读单/覆盖矩阵）与总助提示词库机制同构，后续建站接单可直接复用判读单六字段
- 2026-08-14（追加）：分析《DeepSeek、GLM 的「眼角膜」来了》视频（video_7673332216189348325，短链 4KIhd_qen0k），93s 口播 16 段 + 24 帧 OCR；同主题第二支（先查库比对 8-13 分析，转向增量对比）；复用 performance API 抓音画分离媒体 URL 直连下载；产出 分析_眼角膜野菜人教程_20260814.md

- 2026-08-14（追加）：分析《DeterminFlow 笔枢开源AI小说工作流推出安装版》视频（video_7670954145335069988，短链 sq7KbGptyHA），60.8s 口播 11 段 + 30 帧 OCR；yt-dlp 无 cookies 失败，改从 Chrome 页面上下文 fetch detail API 取 play_addr 直链（首次下载遇截断 1.2MB，换新签名 1080 档 URL 得完整 9MB，经验：直链含时效签名需同会话内新取）；数字密集段（36 agent/32节点/6×34/4×25/7条流水线）用 OCR 逐项校正；GitHub API 核实 alikon-art/DeterminFlow（387★，AGPL-3.0，bishu-novel 插件 7 工作流/84 节点）；产出 分析_DeterminFlow笔枢AI小说安装版_20260814.md；沉淀经验：detail API 直链有签名时效，旧 token 可能返回截断文件

- 2026-08-14（追加）：分析《AI还能做动画？ Codex+Remotion产出动画保姆级全流程！》视频（video_7661104649675623714，短链 LQ7Pd4gxWuM），7分31秒长视频（音画分离：4K HEVC 视频流 + AAC 音频流）；whisper 270 段 + 45 帧 OCR + 豆包 vision 三帧语义核对（开场条形竞速动画/Remotion Studio 预览/高德控制台 Key 列表）；GitHub API 核实 Codex++（BigPizzaV3/CodexPlusPlus 28.9K 星，口播与实测一致）；产出 分析_Codex与Remotion动画全流程_20260814.md；同步沉淀技能 视频生产_Remotion数据可视化动画.md