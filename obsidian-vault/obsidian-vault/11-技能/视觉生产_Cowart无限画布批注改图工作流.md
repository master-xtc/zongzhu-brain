# 技能 视觉生产_Cowart无限画布批注改图工作流

- 技能名：Cowart 无限画布批注改图
- 触发场景：需要「图生图/改图 + 可视化对比迭代」；或人工批注改图需求；或宣传片配图/海报需要改图后对比验收。
- 适用模型：豆包 doubao-seedream-5-0-pro（网关默认）、Lite/4.5/4.0 可换；视觉复核用 Doubao-1.5-vision-pro（ep-20260814015916-q97nj）。

## 步骤
1. **出图/改图（网关）**：
   `python "D:\AI员工体系\scripts\AI员工调度网关_DeepSeek与豆包API.py" --provider ark --mode image --ref-image <参考图路径> --size 2K --prompt "<图生图提示词>" --out <输出路径> --task <任务名>`
   - 提示词模板：`保持图一主体，将X改为Y、A改为B，16:9，其他保持不变，无文字`。
   - 会话需先 `$env:ARK_API_KEY="ark-<你的ARK_API_KEY>"`。
2. **插入画布**：`node <cowart-canvas-skill>/insert-image.mjs --anchor first-image --file <改图路径>`（服务 127.0.0.1:43217，tldraw 架构）。
3. **画布截图**：headless chrome 用 Python subprocess 调起（PowerShell 拦 `--screenshot`），先 PUT `/api/view-state` 把相机对准 draw bounds 中心（`中心世界坐标 = 视口中心 + camera.x`），`--window-size=2000,1100`，去掉 `--virtual-time-budget`。
4. **视觉复核（豆包 vision）**：`--provider ark --mode vision --image <路径> --prompt "<检查项>" --task <任务名>`，逐项确认属性符合。
5. **沉淀**：完成截图与结论存窗口主目录；复盘写 growth\reviews\，本技能同步更新。

## 验收标准
- 改图属性与提示词逐项一致（颜色/衣物/配饰），主体保持一致。
- 画布中双图并排可截图核对，无文字水印（prompt 含"无文字"）。
- 视觉复核逐项通过；截图留档。

## 关键坑（务必先看）
- tldraw 快照：`store.serialize()` + `store.allRecords()` 手写，须含 page/document/instance/instance_page_state/camera/pointer 六类；`getSnapshot` 报 not ready 不可用。
- 无选中 shape 用 `--anchor first-image`。
- esbuild：`npm approve-scripts esbuild` 后必须 `node node_modules/esbuild/install.js`。
- 中文路径删除用 Python shutil.rmtree（UTF-8 脚本文件执行），PowerShell 递归删除与管道传中文会被拦。
