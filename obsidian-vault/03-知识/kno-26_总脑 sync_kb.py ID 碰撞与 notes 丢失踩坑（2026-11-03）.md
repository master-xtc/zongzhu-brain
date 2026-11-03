---
id: "kno-26"
cat: "知识"
tags: [总助大脑, 知识]
---

# 总脑 sync_kb.py ID 碰撞与 notes 丢失踩坑（2026-11-03）

领域：技术/总脑 ｜ 来源：总助 2026-11-03 实测

现象：
① `sync_kb.py --type prompt` 新增封面提示词时自动分配 prompt-81，但自动索引已有 prompt-81（任务派单五要素）；build_kb 合并手工条目时按 ID 去重，手工条目被跳过，knowledge.md 里看不到。
② `--type note` 写入的 note-01 在 `--rescan` 后消失，因为 build_kb.py 的手工合并键只含 memories/skills/prompts/reviews/knowledge/capabilities/requirements，没有 notes。

根因：
- `next_id()` 只在 manual.json 内取最大 ID，不看自动生成的 knowledge.json；自动索引的 prompt 编号会持续增长，手工编号必然撞车。
- notes 既不是 manual 类型，也不在合并键里，重建即丢。

修复：
① 手工 prompt 改为 prompt-901（避开自动 prompt-01..84；本次自动索引在会话中从 83 涨到 84，84 也撞了，故直接留大余量到 901）；
② build_kb.py 合并键加 notes；
③ sync_kb.py 的 note 类型改为 manual: True、extra 改 src，写入 manual.json，重建可保留；
④ 重跑 --rescan 验证 prompts 85（自动84+手工1）/ notes 1，封面提示词与 note-01 都在 knowledge.json。

后续规则：
- 新增手工 prompt 前先查自动最大 ID，手工编号留 100+ 余量（如 901）或改用独立前缀；根本修复是把 next_id 改为跨 manual+kb 取最大。
- 不要把“写入成功”当成“重建后还在”，必须 --rescan 后复查。

验收：
- knowledge.json 中 prompt-901 标题=提示词_小说封面_青铜铃系统环_解铃人_v1；
- note-01 标题=2026-11-03 解铃人建书与总脑回填会话摘要；
- P04=小说《解铃人》；--list 显示 prompts 85、notes 1。

二次踩坑（同日）：自动索引在会话中从 83 涨到 84，原修复用的 prompt-84 再次撞车。结论：手工编号不要贴着自动最大+1，直接留 100+ 余量，或改用独立前缀（如 mprompt-01）。

## 关联

- [[ROOT_总助大脑.md]]
