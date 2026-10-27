---
id: "cap-01"
cat: "能力"
tags: [总助大脑, 能力]
---

# 静态站密码层与权限隔离

领域：安全 ｜ 状态：本阶段新增

对静态站采用 AES-GCM 客户端加密与 PBKDF2 密钥派生，仓库只发布密文与必要静态资源；完整记忆、原始 vault、内部路径与密钥不进入公开仓库。

工具：Node crypto、Web Crypto、GitHub权限、Cloudflare Access

## 关联

- [[ROOT_总助大脑.md]]
- [[TOOL-CloudflareAccess_Cloudflare Access.md]]
- [[TOOL-GitHub权限_GitHub权限.md]]
- [[TOOL-Nodecrypto_Node crypto.md]]
- [[TOOL-WebCrypto_Web Crypto.md]]
