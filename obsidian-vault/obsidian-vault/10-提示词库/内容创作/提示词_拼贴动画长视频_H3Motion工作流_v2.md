# 提示词_拼贴动画长视频_H3Motion工作流_v2

## 名称
H3 拼贴动画长视频 T2VA 提示词模板（官方结构 + 链式工作流实测版）

## 用途
给调度层（豆包任务/Codex skill/总助）一份可直接落地的 H3 视频生成提示词模板：输入一段口播文案 → 自动拆分为 N 段 T2VA 结构提示词（integrated_multimodal_description + overall_soundscape + non_diegetic_music）→ 走 ComfyUI H3-Motion-Context 链式生成 → edge-tts 人声合成，产出 30-60s 竖屏拼贴动画长视频。

## 适用场景
- 抖音知识/情感/故事类口播动画：纸艺定格、复古拼贴、剪报素材风
- "口播文案 → 完整成片"全自动链路，零真人出镜、零素材版权风险
- 批量选题量产（科普叙事、情感叙事、节气/节点选题）

## 变量位
- {口播文案}：200-400 字解说稿，按意群切 6-7s/段
- {风格}：默认"papercraft stop-motion collage"；可换"复古剪报拼贴 / 暗黑纸艺 / 童书绘本"
- {目标时长}：默认 36-42s（6 段）；30-60s 范围
- {画幅}：默认 9:16 竖屏（704x1248 或 480x864 低显存）
- {切点时间戳}：每段 3 Shot，首切 2.5-2.9s、二切 5.0-5.4s（严格递增且小于段长）

## 正文模板（T2VA 三字段，英文撰写，每段 ≥600 字符）

### 字段一：integrated_multimodal_description（画面主文）
```
Papercraft stop-motion collage animation, {风格补充词}, {色调氛围}, {纸张质感描述}.

[Shot 1] {初始构图：景别 + 主体 + 环境 + 关键道具 + 材质细节 + 光线}，{氛围副词}.

[Shot 2] At 00:0X.XXX, the shot cuts to {新景别/新角度}, the camera {镜头运动类型} {幅度} {速度}, revealing {新细节}.

[Shot 3] At 00:0X.XXX, the camera continues {镜头运动}, {主体微变化}, preserving {情绪基调}. No text, no subtitles{如需无字加 , no people speaking}.
```
### 字段二：overall_soundscape（全片声音，铁律版）
```
No dialogue, no voice-over, no people speaking; complete silence, no ambient sound, no sound effects.
```
### 字段三：non_diegetic_music（BGM，铁律版）
```
No background music.
```

### Shot 写法规范（官方指南要点）
- [Shot 1] 不写时间戳，开头先定风格与初始构图；后续 Shot 时间戳严格递增且落在段长内
- 镜头运动三要素：类型（Zoom In/Out、Pan、Dolly、Orbit）+ 幅度（small/wide）+ 速度（slow/gentle）
- 每段 2-3 个 Shot；段内信息推进靠切景别或镜头运动，不重复描述
- 全部英文撰写；画面中的文字/对白保持原语言（本模板要求无文字）

## 完整示例（《月球在告别》S01，实测通过）
```
integrated_multimodal_description: Papercraft stop-motion collage animation, vintage handmade aesthetic, deep blue paper night sky with visible paper grain and layered textures.

[Shot 1] A wide establishing shot frames a large paper-cut moon floating at the center of the composition, its surface built from torn textured paper with subtle craters; small paper-cut stars twinkle around it, and tiny human silhouette figures stand at the bottom edge looking up. Paper layers cast soft shadows, a warm rim light traces the moon's edge, the scene stays calm and slow.

[Shot 2] At 00:02.500, the shot cuts to a closer view, the camera slowly pushes forward toward the moon, revealing coarse paper fibers and glued collage layers; the tiny silhouettes remain visible at the lower frame, still gazing upward.

[Shot 3] At 00:05.200, the camera continues a gentle slow zoom in, the moon drifting imperceptibly, stars drifting with it, preserving the quiet handmade mood. No text, no subtitles, no people speaking.

overall_soundscape: No dialogue, no voice-over, no people speaking; complete silence, no ambient sound, no sound effects.

non_diegetic_music: No background music.
```

## 生成参数（实测写死值）
- 分辨率 704x1248（9:16 竖屏，32 倍数）；低显存 480x864
- 帧数 = 段长秒 x 24fps（7.3s → 175 帧；5.2s → 124 帧）
- KSampler：res_multistep / simple / 8 步 / CFG 1.0 / denoise 1.0
- MiniMaxH3SigmaShift：shift_video 12.0 / shift_audio 3.0
- Turbo 4-step LoRA：strength 1.0
- 链式：S01 无 context；S02+ LoadLatent(前段) + context_length=22 + audio_context_length=24 + SaveLatent(clip_index=段号)
- 配音：edge-tts Xiaoxiao 女声 -8%（6.2-7.5s/句，与段对齐）；合成时 H3 原声压 0，仅人声

## 反例规避
- 禁止跳过三字段头：模型依赖字段名与顺序（官方指南硬性要求）
- 禁止每段各自为政：必须 LoadLatent 前段 latent，否则段间画面跳变
- 禁止单段超 15s：H3 质量衰减，稳定区间 6-7s
- 禁止加 BGM/音效：习总规则"视频仅人声"；soundscape 与 music 字段写静音
- 禁止口播与画面脱节：每段画面由对应文案反推
- 禁止低质提示词：每段 ≥600 字符、3 Shot 结构、镜头运动三要素齐备

## 验收标准
- 成片时长=目标时长±2s；段间衔接肉眼不可辨
- 口播逐字完整（whisper 转写比对）；画面无文字乱码（豆包 vision 抽帧核对）
- 画面风格统一（纸艺质感/色调延续）、无 BGM、仅人声
- 全流程可一键触发：只输入{口播文案}即出成片

## 使用记录
- v1（2026-08-15）：来源抖音《豆包加上minimax，一句话搞定拼贴动画长视频》（ID 7673461473679461450）拆解，调度层任务书版
- v2（2026-08-15）：《月球在告别》实测升级——官方 T2VA 三字段模板、Shot 写法规范、生成参数写死值、静音铁律；6 段 JSON 已生成（H3_月球在告别_v2_S01~S06.json），待下载完成后实测登记画面效果
