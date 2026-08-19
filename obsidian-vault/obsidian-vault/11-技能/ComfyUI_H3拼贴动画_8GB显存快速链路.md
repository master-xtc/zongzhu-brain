# 技能_ComfyUI_H3拼贴动画_8GB显存快速链路

## 技能名
ComfyUI MiniMax H3 拼贴动画 8GB 显存快速生成

## 触发场景
- 需要生成抖音竖屏拼贴/纸艺/手作动画。
- 使用 MiniMax H3 fl2va + Turbo 4-step LoRA。
- 显卡为 8GB 显存笔记本（RTX 5060 Laptop 实测）。

## 步骤
1. 检查模型：minimax_h3_fl2va_pruned_int8_convrot.safetensors，官方 SHA256 为 e889202c41dafb67b10d67b97f0d8541508036a6090af23425a5c2615d03c47a。
2. 启动 ComfyUI：D:\AI工具\ComfyUI\venv\Scripts\python.exe -u main.py --listen 127.0.0.1 --port 8188。
3. 工作流参数：MiniMaxH3ImageToVideo width=480 height=864 length=124；KSampler steps=4 sampler_name=res_multistep scheduler=simple；LoraLoaderModelOnly 加载 MiniMax H3\Turbo 4-step.safetensors strength_model=0.8。
4. 6 段串行：S01 先跑，S02-S06 使用 MiniMaxH3MotionContextLoadLatent/MiniMaxH3MotionContext 串接上一段。
5. 每段用 H3_正式生成_6段_4步_480p.py 串行提交并轮询，约 2.5 分钟/段。
6. 配音合成：用 H3_合成成片_v4.py 按配音句长拉长视频段落，concat 后只叠人声，输出 1080x1920。

## 验收标准
- 6 段均 ComfyUI status=success。
- 成片 1080x1920、24fps、AAC 48kHz 立体声、无 BGM、无字幕。
- 音量 loudnorm 后 max 约 -3dB，mean 约 -16dB 到 -20dB。

## 适用模型
- ComfyUI 0.33.0 + PyTorch 2.11.0+cu130 + MiniMax H3 fl2va。
- 配音：edge-tts 已生成 mp3。

## 踩坑记录
- 704x1248/175帧 在 8GB 上单步 1242 秒，不可用。
- 模型下载不要用 Python urllib 并发；用 curl.exe，避免内存耗尽重启。
- 合成时人声句长可能超过单段视频，必须按句拉长段落，否则语音会被截断或错位。
