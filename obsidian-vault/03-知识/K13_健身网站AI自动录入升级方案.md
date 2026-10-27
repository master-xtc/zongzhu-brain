---
id: "K13"
cat: "知识"
tags: [总助大脑, 知识]
---

# 健身网站AI自动录入升级方案

领域：开发建站 ｜ 来源：总助 20260821 对接文件

健身指挥台(fitness-command.netlify.app)AI化四块方案：①拍照算热量=新增Netlify Function food-analyze，前端canvas压缩图片后调ark视觉API，结果写localStorage复用p0.js calorieItems；②拍照自动体态自评=posture-analyze函数返回score+建议；③拍照展板卡通化=纯前端SVG贴纸+圆角卡片，保留v6炸裂特效；④每晚23:00自动打分=Netlify Scheduled Function(daily-score)，前端仅同步文本快照到Blobs daily-snapshots(不含照片)，评分写回daily-scores次日展示。完整交接文档:D:\健身教练\对接文件_健身网站AI自动录入升级_v1_20260821.md。红线:照片本机处理不留存服务器。

## 关联

- [[ROOT_总助大脑.md]]
