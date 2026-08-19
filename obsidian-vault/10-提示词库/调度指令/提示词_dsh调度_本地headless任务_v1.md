# 提示词 · dsh 调度（本地 headless 任务）

## 用途
向 DeepSeek Harness（DSH）员工下发本地任务的标准指令模板；覆盖单任务执行、批量流水线、Web 值守三种调用形态。

## 适用场景
- 本地代码/文件分析、改码、执行命令（workspace 为当前目录）
- 批量无头任务（一条指令一任务、结果即答即出）
- Web UI 值守（127.0.0.1:3080，模型页管理凭据与会话）

## 变量位
- {任务指令}：headless 模式的单条任务描述，越具体越好（含目标/范围/约束/输出）
- {工作目录}：dsh 运行目录，默认作为 workspace 根目录（sandbox 权限边界）
- {profile名}：默认 web / headless；自定义 profile 用 dsh plugin 创建

## 正文模板
1. 安装与基线（2026-08-15 实测）：
   - 程序：D:\AI工具\npm-global\dsh.cmd（用户 PATH 已配，直接 dsh 可调）
   - 数据：DSH_HOME=D:\AI工具\dsh-home（profiles/sessions 全 D 盘）
   - 凭据：用户级 DEEPSEEK_API_KEY（自 Hermes .env 转存，不写文档）
2. 调用形态：
   - 单任务：dsh --profile headless 任务指令（持久化会话，打印最终答案退出）
   - Web 值守：dsh web（默认 127.0.0.1:3080）
   - 插件管理：dsh plugin --profile profile名 add 包名
3. 任务书要素（下发任务时必带）：目标 / 背景 / 约束（审美规范、红线）/ 交付物形式 / 验收标准 / 截止要求

## 示例
- dsh --profile headless 分析 D:\问题解决 下所有 .md 文件，列出命名不符合 类别_对象_用途 规范的文件并给出改名建议
- dsh --profile headless 读取 x.csv，汇总每列空值率，输出 Markdown 报告

## 反例规避
- 不用 headless 跑需要交互确认的长任务（先 web 值守）
- 不在指令中携带 API Key、习总隐私、真实姓名
- 不在 C 盘目录运行（sandbox 边界随 cwd，工作数据一律 D 盘）

## 验收标准
- headless 返回最终答案且非零错误；结果落到当前工作区/指定 D 盘路径
- 任务后回填复盘 growth\reviews\ 与调度指令使用记录

## 使用记录
- 2026-08-15：入编首测（headless 自我介绍，5.6s 返回）

## 注意事项与踩坑记录
- **任务范围必须收敛**（2026-08-15 实战教训）：headless 递归扫描全目录（D:\问题解决 14.3 万行清单）会陷入超长执行，务必限定目录层级/排除大目录/只查首层，或先出清单再本地脚本分析
- npm 新版本默认 allow-scripts 拦截原生包脚本：需补跑 --allow-scripts=@deepseek-ai/dsh-subprocess-local,koffi,node-pty,@google/genai,protobufjs，否则 node-pty/koffi 不可用
- 首次跑 headless 若报 MISSING_CREDENTIAL：把 DEEPSEEK_API_KEY 设为用户级环境变量即可（web Models 页也可写入）
- npm prefix 已迁 D:\AI工具\npm-global，后续全局包默认装 D 盘

