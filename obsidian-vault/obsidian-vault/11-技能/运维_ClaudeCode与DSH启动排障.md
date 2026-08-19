# 技能_Claude与DSH启动排障

## 技能名
运维_ClaudeCode与DSH启动排障

## 触发场景
- Claude Code / DSH 执行岗打不开、双击无反应、闪退、报"不是有效的 Win32 应用程序"、报 `claude.exe is not recognized`。
- 用户反馈"还是没用"时，作为第一轮排障结论失效后的深挖清单。

## 步骤
1. 查库：团队共享记忆 + 岗位手册（profile\）确认安装基线与调用方式。
2. 命令层：`claude --version`、`dsh --version` 确认程序本体是否正常。注意：某条路径走通不代表用户双击的路径可用，必须沿"快捷方式 → shim → 目标 exe"全链路核对。
3. 后端层：Claude 用 `claude -p "最小提示"` 验证 DeepSeek 后端连通；DSH 用 `dsh web` + 访问 127.0.0.1:3080 验证。
4. 入口层：读桌面 lnk 目标（WScript.Shell）→ 读 shim 内容（claude.cmd/claude.ps1 指向的 exe）→ Test-Path 目标 exe。
5. 关键检查（Claude Code）：`nodejs\node_modules\@anthropic-ai\claude-code\bin\` 下必须存在 `claude.exe`；若只有 `claude.exe.old.*`，说明自动更新中断，将 `.old` 重命名为 `claude.exe` 即可恢复；后续建议正规重装 `npm install -g @anthropic-ai/claude-code`。
6. 修复：lnk 指向可用的 .cmd；DSH 必须用 dsh.cmd/dsh.ps1，禁用无扩展名 shim 做入口。
7. 验证（以用户实际入口形态为准）：双击快捷方式 / cmd 与 PowerShell 双入口 --version / 最小真实调用。
8. 落盘报告 + 回填复盘。

## 验收标准
- 用户实际使用的入口（双击/小窗）实测通过；命令层与后端层均实测通过；报告落盘、复盘回填。

## 注意事项与踩坑
- npm 无扩展名 shim 是 bash 脚本，Windows 直接调用报"不是有效的 Win32 应用程序"。
- 快捷方式失效常见两类：旧 shim 指向 WorkBuddy 内嵌路径（随版本变动消失）；Claude 自动更新残留 `.old.*` 导致 exe 缺失。
- `--version` 走通≠双击可用：不同 shim 可能解析到不同安装，验证必须以用户实际路径为准。
- DEEPSEEK_API_KEY 在用户级环境变量，需新开终端生效。

## 使用记录
- 2026-08-19：Claude 执行岗 + DSH 恢复可用（习总报障，两轮排障，v2 找到自动更新残留根因）。