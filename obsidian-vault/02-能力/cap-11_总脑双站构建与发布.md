---
id: "cap-11"
cat: "能力"
tags: [总助大脑, 能力]
---

# 总脑双站构建与发布

领域：知识库运维 ｜ 状态：已实测

把总助大脑的本地数据构建成两套发布物并完成线上验收：旧完整站走 Python 构建链（collect/build_kb/generate_md/inject/export_obsidian）产出 knowledge.json、knowledge.md、index.html、Obsidian 库与 zip，复制进 _deploy 后 git 提交推送 GitHub Pages；新共享站走脱敏生成器加 AES 加密器，产出只含加密载荷的 loader 页面，密码单独保管，明文不进仓库。验收动作固定为：查 Pages 最新构建状态、拉取线上页面核对版本号与最新条目标题、核对知识图谱节点数、核对 meta.counts 合计、共享站用密码解密后抽查条目。

工具：Python、PowerShell、Git、GitHub Pages、AES 加密脚本

## 关联

- [[ROOT_总助大脑.md]]
- [[TOOL-AES加密脚本_AES 加密脚本.md]]
- [[TOOL-Git_Git.md]]
- [[TOOL-GitHubPages_GitHub Pages.md]]
- [[TOOL-PowerShell_PowerShell.md]]
- [[TOOL-Python_Python.md]]
