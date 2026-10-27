# 技能_运维_软件下载安装到D盘与CDN风控绕过

- 技能名：Windows 软件下载安装到 D 盘 + CDN 风控绕过
- 领域：运维 / 系统落地
- 适用模型：总助 Codex（任何具备 shell 能力的执行体）
- 沉淀日期：2026-09-17

## 触发场景

- 习总说"下载 XX 软件放到桌面 / 装到 D 盘 / 建独立文件夹"。
- 需要从官网下载安装包，但页面动态渲染、直链藏在 JSON 里。
- 命令行下载拿到 200 却只有几十 KB（CDN 返回验证页而非真实文件）。
- 需要把装在 C 盘的软件及其数据迁到 D 盘。

## 步骤

### 一、勘察（先查库再动手）
1. 查桌面/目标盘是否已装：`Get-ChildItem 'C:\Users\XTC\Desktop' -Filter '*关键词*'`；注册表 Uninstall 项。
2. 查目标盘空间：`Get-PSDrive D`。
3. 查权限：`(New-Object Security.Principal.WindowsPrincipal([Security.Principal.WindowsIdentity]::GetCurrent())).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)`——非管理员要预判哪些步骤做不了。
4. 查团队库：`D:\AI员工体系\growth\skills`、`reviews`、`提示词库` 是否已有同类沉淀。

### 二、拿官方直链
1. 拉官网下载页：`Invoke-WebRequest -UseBasicParsing`，用正则搜 `dl\.<域名>\.com[^"' <>)]*`。
2. 若是动态页面：搜 URL 转义形态（`\u002F` = `/`），还原后再筛，通常能捞到 Windows/mac/linux 全平台直链。
3. 直链探测：`curl.exe -sS -I` 看 `Content-Length`，记下期望体积作为校验基准。

### 三、下载（关键：绕过 CDN 风控）
```powershell
curl.exe -sS -L -o 'D:\目标目录\安装包.exe' `
  -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36' `
  -e 'https://官网下载页' `
  --max-time 600 '直链'
```
- **必须带浏览器 UA**，否则 EdgeOne 等 CDN 返回 JS 挑战页（HTTP 200，体积约 29 KB 的 `<!DOCTYPE html>`）。
- 下载后校验：文件头两字节应为 `4D 5A`（MZ）；`Get-Item` 的 `VersionInfo` 应有 `FileVersion / CompanyName`——版本信息为空 = 下载的是 HTML 不是 exe。

### 四、安装到 D 盘
1. 建独立文件夹：`D:\<软件名>\{安装包, 程序, 数据}`。
2. 判安装器类型：读二进制找 `Nullsoft`/`NSIS`（→ `/S /D=路径`）、`Inno Setup`（→ `/VERYSILENT /DIR="路径"`）。
3. 静默安装：
```powershell
Start-Process -FilePath '安装包.exe' -ArgumentList '/S','/D=D:\软件名\程序' -PassThru
```
- NSIS 铁律：`/D=` 必须是最后一个参数、路径不要加引号（含空格路径除外，NSIS 会截断）。
4. 验收：`Get-ChildItem` 目标目录应出现主 exe；查注册表 Uninstall 项的 `InstallLocation`。

### 五、桌面入口
- 安装器一般会在 `C:\Users\Public\Desktop` 建快捷方式（用户桌面可见），核验：`(New-Object -ComObject WScript.Shell).CreateShortcut($lnk).TargetPath` 应指向 D 盘主程序。
- 若安装器没建，手动建到 `C:\Users\XTC\Desktop`，**不要两个桌面都建**（会出现重复图标）。

### 六、数据也搬到 D 盘（可选，需管理员）
1. 先查残留：`C:\Users\XTC\AppData\Local\<软件>`、`C:\Users\XTC\AppData\Roaming\<软件>`、`C:\ProgramData\<软件>`。
2. **动手前先确认占锁进程能否停**——非管理员不要贸然 Move-Item，会"部分成功"留下半迁移脏状态。
3. 完整流程（管理员）：停服务 → Move 到 `D:\软件名\数据\` → 源位置 `cmd /c mklink /J "源" "目标"` → 重启服务 → 写日志。
4. 目录联接（junction）不需要管理员，删除/移动目标目录才需要；建 junction 前源目录必须不存在。

## 验收标准

- 安装包体积与官网 `Content-Length` 一致，`VersionInfo` 有版本号与公司名。
- 主程序进程路径（`Get-Process | Select Path`）在 D 盘。
- 桌面快捷方式存在且目标指向 D 盘 exe。
- C 盘残留有清单、有体量数据、有处置结论（迁移 / 脚本待批 / 说明理由）。
- 交付说明 `.md` 落盘，含目录表、数据落点、卸载方式、回滚方式。

## 反例（禁止）

- 只发 `curl 直链` 不看返回内容，把 29 KB 的 HTML 当成 148 MB 的安装包交付。
- 用 HTTP 200 判断下载成功（CDN 风控页也是 200）。
- 非管理员直接 Move 被服务占用的目录，留下半迁移状态还继续往下走。
- 在用户桌面和公共桌面各建一个快捷方式，桌面上出现两个重复图标。
- 只把程序装到 D 盘，对 C 盘 AppData/ProgramData 残留不闻不问、不汇报。

## 使用记录

- 2026-09-17 ToDesk 5.1.0.0：官网动态页解析直链 + Chrome UA 绕过 EdgeOne 风控 + NSIS `/S /D=` 装到 `D:\ToDesk\程序`，一次成功。
