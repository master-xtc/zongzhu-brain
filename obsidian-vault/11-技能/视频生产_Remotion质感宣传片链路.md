# 技能：视频生产_Remotion质感宣传片链路

## 触发场景
- 需要"动态运镜 + 字幕 + 转场"的高质感宣传片/片头/展示视频（静态轮播显 low 时）。
- 已有 AI 出图（seedream 等）作为素材，需要 2.5D 视差动效。

## 步骤
1. **查库**：reviews/skills/提示词库查同类任务沉淀；素材（图/VO/BGM）尽量复用。
2. **素材准备**：出图指定 16:9（如 size=2K 且提示词含宽幅构图）；VO 用 edge-tts/speech skill；BGM 选氛围音乐。
3. **工程搭建**：自 video-shotcraft template（Remotion + TS）拷贝，`npm i` 就位 node_modules。
4. **镜头设计**：每镜头 120 帧（4s）@30fps，`Scene` 组件：背景图 + scale 1.0→1.12 慢推 + translate 微移（2.5D）；金色字幕 `#C9A96A` 入场（scale 0.9→1 + opacity + slight letter-spacing）。
5. **转场**：相邻镜头各 6 帧（共 12 帧）交叉溶解；开场 impact、镜头切 whoosh、结尾 riser。
6. **字幕**：中文标题用 NotoSerifSC-VF（衬线，金色），副题/序号用 NotoSansSC-VF；绝对定位避免 `AbsoluteFill` 子层 inset 冲突（用显式 left/top/width/height）。
7. **合成音频**：BGM vol≈0.13、VO vol≈1.6（按镜头时间轴 i*120+14 起播）、SFX vol≈0.15。
8. **渲染**：`npx remotion render src/index.ts out/xxx.mp4`，h264 高码率（16Mbps+）保证清晰度。
9. **QA 复核**：写 UTF-8 抽帧脚本（ffmpeg `-ss` 输出型 seek 每镜头中心），豆包 vision 一次批量上传全部帧，逐帧查：主体/字幕逐字/乱码/黑边/比例/花屏。

## 验收标准
- 1920×1080 @30fps，无黑边/花屏/跑版；字幕逐字正确；运镜平滑无跳变；成片无 AI 角标。

## 适用模型
- 视觉复核：Doubao-1.5-vision-pro（ep-20260814015916-q97nj）
- 出图：doubao-seedream-5-0-pro
- 配音：edge-tts / speech skill