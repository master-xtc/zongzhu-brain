# 视频生产_ffmpeg中文图文轮播渲染_v1

## 技能名
ffmpeg 中文图文轮播宣传片渲染（图片序列 + 配音 + BGM + 中文字幕）

## 触发场景
- 用多张 AI 出图 + edge-tts/豆包配音做轮播式宣传片/短视频
- ffmpeg 合成中文视频报 drawtext 错误、0 帧成片、concat SAR 报错

## 步骤（Windows，均实测通过 2026-08-14）
1. 素材齐备：img01..img08（2K 横图）+ vo01..vo08（配音）+ BGM。
2. 字幕中文不写 filter 的 text=：写入 UTF-8 文本文件到纯 ASCII 目录（如 D:\XTC\_subs\s01.txt），drawtext 用 textfile 引用。
3. filter 路径写法：`drawtext=fontfile='C\:/Windows/Fonts/simhei.ttf':textfile='D\:/XTC/_subs/s01.txt':fontsize=64:fontcolor=white:borderw=3:bordercolor=black@0.8:x=(w-text_w)/2:y=h-180`（路径单引号包裹 + 冒号转义）。
4. 图片输入必须循环补帧：`-loop 1 -framerate 30 -t 4 -i img.png`（否则单帧输入成片只有 0.1s）。
5. 每段视频链尾加 `format=yuv420p,setsar=1`（scale/crop 后 SAR 不一致会导致 concat 报 Invalid argument）。
6. 配音延迟拼接：`[音]:adelay=起始毫秒|起始毫秒,volume=1.6[a0]`，多路 amix；BGM 用 `-stream_loop -1 -i bgm` + atrim。
7. 整条命令用 Python subprocess（args 列表）执行，勿经 PowerShell 传中文路径（会损坏编码导致 0 帧/静默失败）；脚本先落盘 .py 再跑。
8. 抽帧复核用输出型 seek：`-i 成片 -ss 时间 -frames:v 1 帧.png`（快速 seek 在部分编码下 0 帧）。

## 验收标准
- 成片 ffprobe 帧数 = 秒数×帧率（如 32s×30fps=960 帧），含音轨。
- 每段抽帧交豆包视觉复核：画面与字幕一一对应、无乱码/方块/花屏。

## 适用模型/工具
- ffmpeg 9.0 full build（Gyan）、Python subprocess、豆包 turbo 视觉复核。
