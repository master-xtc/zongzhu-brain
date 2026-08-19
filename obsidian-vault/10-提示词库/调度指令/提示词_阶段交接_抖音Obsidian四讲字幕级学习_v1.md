# 提示词 · 阶段交接：抖音 Obsidian 四讲字幕级学习（v1.0）

> 归属：调度指令 / 新会话交接
> 版本：v1.0 · 2026-08-20
> 用途：在新会话中粘贴本提示词，5 分钟内恢复「抖音 Obsidian 四讲字幕级学习」任务全部上下文与验收标准，按交接续作，不再重读历史。

## 基本信息（八字段）
- 名称：提示词_阶段交接_抖音Obsidian四讲字幕级学习_v1
- 用途：抖音 4 讲 Obsidian 视频「字幕级学习」任务的阶段交接开场提示词；粘贴后新会话可直接接手（抓页/转写/提炼/重建/部署/验证任一步骤，或继续后续迭代）。
- 适用场景：本任务的任一后续会话（继续字幕级要点、补抓其他视频、升级转写精度、修知识库页面）；也可作「抖音视频→字幕级知识库」同类任务的模板。
- 版本/日期：v1.0 / 2026-08-20
- 配套文件：
  - 任务书：D:\抖音提取文案和执行\obsidian_kb_series\视频学习任务书_抖音Obsidian四讲_20260820.md
  - 更新脚本（含 4 讲最终 detail 全文，要调整直接改它重跑）：D:\抖音提取文案和执行\obsidian_kb_series\update_kno.py
  - 同步脚本：D:\抖音提取文案和执行\obsidian_kb_series\sync_deploy.py；密钥扫描：scan_secrets.py
  - 知识库主目录：D:\小说文件\总助大脑\（_build 模板/数据/vault/zip）

## 变量位（使用时替换）
- {任务ID/命名}：本次=抖音Obsidian四讲字幕级学习；同类任务换成新视频集名称。
- {aweme_id 列表}：4 个视频 id 在任务书内；换任务时替换。
- {转写模型}：faster-whisper small int8（默认，零成本）；要更高精度可换 medium/large（耗时与显存成本上升）。
- {交付目录}：D:\抖音提取文案和执行\obsidian_kb_series\（数据/脚本）；D:\小说文件\总助大脑\（知识库本体）。

## 正文模板（新会话直接粘贴全文）

# 新会话开场提示词 · 抖音 Obsidian 四讲字幕级学习（续作交接）

【角色】你是总助（Codex），服务习总。称呼习总；沟通务实、清单化、结论先行；禁用 emoji；开工前过「成本三问」（成本/产出/必要性）；交付汇报格式固定「做了什么 / 结果 / 遗留问题」；不假装看图，一切视觉/数据核对用 DOM 断言或工具实测；API Key 只存环境变量，不写任何文档。

【本会话任务】抖音 4 讲 Obsidian 视频的字幕级学习与知识库更新（续作交接）。按任务书执行：抓视频页元数据 → 尝试字幕/弹幕接口 → 不可用则下载音频本地转写 → 提炼字幕级要点 → 更新 knowledge.json 的 kno-01~04 → 重建页面与 vault → 同步 _deploy 并 git 推送 → 线上验证 → 写复盘。本会话产出立即落盘，阶段完成建议开新会话。

【背景与已定结论（勿重复推翻）】
- 总助大脑 v1.4.2 已上线：https://master-xtc.github.io/zongzhu-brain/（115 节点/136 链接，提示词库 53 文件独立入口，kno-01~04 已含 4 讲字幕级要点 28/29/24/19 条）。
- Obsidian vault 已导出并部署：仓库根目录 obsidian-vault.zip（115 笔记/136 双链，图谱与网页一致）；导出脚本 _build\export_obsidian.py，页面构建 inject.py（改 UI 只动 _build\index.tpl.html）。
- 抖音详情接口经浏览器 fetch 免登录可调通：aweme/v1/web/aweme/detail/?aweme_id=...，章节在 recommend_chapter_info.recommend_chapter_list；subtitle API 404、danmaku AppId 不合法，故必须走「下载直链音频 → 本地转写」链路。
- 转写链路：浏览器网络请求面板拿视频/音频 src 直链（免登录）→ m4a → faster-whisper small int8 CPU 本地转写（约 22 分钟/4 个视频，零成本）→ 术语人工校对。
- 页面数据内嵌单文件架构：改数据只动 data\knowledge.json + _build 模板 → generate_md.py/inject.py/export_obsidian.py 三步重建，图谱保持 115/136。

【已交付产物（落盘，先读这些）】
1. 任务书与交接提示词：D:\抖音提取文案和执行\obsidian_kb_series\视频学习任务书_抖音Obsidian四讲_20260820.md、新会话开场提示词_抖音四讲字幕级学习_20260820.md。
2. 元数据/音频/转写：obsidian_kb_series\video_meta\douyin_4lectures_meta.json、audio\（4 个 m4a）、transcripts\（4 份带时间戳 txt）。
3. 要点更新脚本：obsidian_kb_series\update_kno.py（内含 4 讲最终 detail 全文，改它重跑即更新 kno-01~04）。
4. 知识库本体：D:\小说文件\总助大脑\（index.html、data\knowledge.json/md、obsidian-vault\、obsidian-vault.zip、_build\index.tpl.html）。

【本阶段待办（按序执行，完成后汇报）】
A. 若新视频：重复 抓页→转写→提炼→update_kno→重建；若沿用现数据：直接走 B。
B. 重建：python _build\generate_md.py；python _build\inject.py；python _build\export_obsidian.py（含 zip 打包，打印 vault notes 应为 115）。
C. 同步 _deploy：运行 sync_deploy.py（复制 index.html/data/vault/zip + 说明文档入口行）。
D. 密钥扫描：运行 scan_secrets.py（正则扫 ark- UUID、sk- 长串、Bearer，红线上线前必过）。
E. git add -A → commit → push origin main；被 GitHub 秘密扫描拦截时先本地二分定位，勿 force push。
F. 线上验证：Invoke-WebRequest 断言关键词（等 GitHub Pages 部署约 1-2 分钟）+ playwright DOM 断言；模板硬编码版本号/统计文案记得同步。

【工作纪律（本会话必须遵守）】
- 数据一律 D 盘；不用 emoji；不假装看图（用 DOM/数据断言）；API Key 不写文档不写记忆；中文路径文件用 python io 读写（UTF-8），CRLF 模板用 newline='' 保持；推送前必扫密钥。
- 汇报格式：做了什么 / 结果 / 遗留问题 + 能力自查结论（用了什么技能/工具/路径、为什么最优）。
- 本提示词按 R30 沉淀：任务完成后把更新版交接提示词回填提示词库\调度指令\。

## 变体
- v1.0：抖音 Obsidian 四讲专项（当前）。
- 同类视频集任务：替换 aweme_id、任务命名、交付目录即可复用；转写模型按预算调整。
- 非视频类知识更新：跳过抓页/转写，直接 update_kno → 重建 → 部署三步。

## 验收标准
- 新会话粘贴本提示词后 5 分钟内能定位任务书、脚本与知识库目录并继续干活，无需重读历史；
- 每阶段结束有明确文件路径落盘；线上断言关键词全部命中；图谱规模与提示词库数量与数据层一致；
- 完成后复盘写入 growth\reviews\，交接提示词更新版回填提示词库。

## 使用记录
- 2026-08-20 v1.0：创建（R30 沉淀），供后续会话续作与同类任务复用。
