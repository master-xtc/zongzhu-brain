---
id: "kno-09"
cat: "知识"
tags: [总助大脑, 知识]
---

# 真实桌面路径与交付校验

领域：运维 ｜ 来源：

Windows实际桌面由注册表HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders的Desktop值决定；本机为D:\桌面，不是C:\Users\XTC\Desktop。交付到桌面时必须先读取该值并核对目标文件存在；本次征文文件曾误放C盘旧桌面导致习总不可见，已复制到D:\桌面并完成哈希与文件存在校验。今后桌面交付默认D:\桌面。

## 关联

- [[ROOT_总助大脑.md]]
