# 技能_开发建站_小程序数据层抽取管线_运行时id补齐与幂等校验

**技能名**：网页静态数据 → 小程序数据模块的抽取管线（运行时 id 补齐 + 幂等校验）
**触发场景**：把已有网页/网站的题库、词库、素材库等数据搬到小程序端，需要「一份数据两端共用、永不增叉」。
**与其他技能的关系**：本技能是「开发建站_网站搬运到微信小程序_个人主体合规与差距矩阵」的**执行篇**；
那份管「能不能搬、怎么规划」，这份管「数据层怎么搬、怎么验收」。
**适用模型**：任何具备文件读写与命令执行能力的执行体（总助直连即可）。
**来源**：2026-09-14 CET4 备考工具包小程序 P1 实战（验收 49 项全通过）。

## 一、核心结论（先记住这五条，能省一整轮返工）

1. **数据只维护一份**，网页端是唯一真相；小程序端的数据文件是**生成物**，标「自动生成，禁止手改」。
2. **id 不要在生成物里预写**。若网页端用 `appendWithIds()` 运行时补齐 id，生成物就必须保持"无 id/占位 id"，
   id 由小程序端在 `app.js` 初始化阶段按同样的合并顺序补齐——这样两端语义才一致。
3. **合并顺序 = id 分配顺序**。顺序错一位，全站 id 全错。必须在代码注释里把「基础 N + 扩充 M = 总量」写死。
4. **幂等要能被证明**：做成「重跑脚本 + 比对 sha256」的自动断言，而不是口头保证。
5. **验收必须跑真源码对照**：拿网页端原版合并函数真跑一遍，与小程序端结果逐项比对（含首尾元素）。

## 二、标准步骤

### 第 1 步：定位数据源与数据结构

- 网页端若只有一个 `script.js` 放全部数据，**先把顶层数组清单抓出来**：
  ```powershell
  Select-String -Path web\script.js -Pattern '^(const|let|var)\s' | ForEach-Object { "$($_.LineNumber): $($_.Line.Trim())" }
  ```
- 只要「行首 `const/let/var NAME = [`」这一种形态，就是可搬运的纯数据数组；
  带 `document.` / `localStorage` 的行都是逻辑，跳过。
- 记录每个数组的**条目数**与**是否含 id**（有的数组是纯字符串，如名言库，不含对象）。

### 第 2 步：写抽取器（括号匹配，不要用正则切分）

- 用「行首声明正则 + 括号深度匹配」抽数组字面量，**匹配时跳过字符串内容与行注释**
  （否则字符串里的 `[`、`]`、`"` 会让深度算错）。
- 字面量边界＝从 `=` 后的 `[` 到与之配对的 `]`；**不要用 `split('=')` 切**（值里出现 `=` 就崩）。
- Python 参考实现要点：
  ```python
  BLOCK_RE = re.compile(r'^(?:const|let|var)\s+([A-Za-z_$][\w$]*)\s*=\s*(?=\[)')
  # 逐字符扫：in_str 状态机跳过字符串；遇到 "//" 提前 break 本行；depth==0 时收尾
  ```

### 第 3 步：交给 Node 求值成 JSON（不要自己写 JS 解析器）

- 把抽出来的字面量写进一个临时 `.js`，用 `node 临时文件` 执行，`process.stdout.write(JSON.stringify(...))`。
- **不要用 `node -e "<很长的代码>"`**：命令行长度上限会直接报 `WinError 206 文件名或扩展名太长`。
- 求值成功 = JS 语法正确（等于免费拿到一层语法校验）；失败就把 stderr 原样抛给使用者。
- 用临时目录，结束后清理。

### 第 4 步：渲染 `module.exports` 数据模块

- 头部注释必须写清四件事：**自动生成禁止手改 / 唯一真相路径 / 重新生成命令 / id 策略**。
- 导出形式 `module.exports = { NAME, NAME, ... }`，名字与网页端数组名保持一致（便于对照）。
- **紧凑 JSON**（`separators=(',',':')`、`ensure_ascii=False`）：
  814 词的词库若 pretty-print 会膨胀到 500 KB+，紧凑输出约 130 KB，小程序主包才吃得消。
- 行尾统一 LF，否则「重跑字节级一致」的幂等断言会因 CRLF 漂移而失败。

### 第 5 步：小程序端合并逻辑（与网页端逐行等价）

```js
function appendWithIds(target, extras) {
  if (!Array.isArray(target)) return target;
  if (typeof extras === 'undefined' || extras === null || !Array.isArray(extras)) return target;
  const base = target.length;
  extras.forEach(function (item, i) {
    if (item && typeof item === 'object') item.id = base + i + 1;  // 纯字符串元素不处理
    target.push(item);
  });
  return target;
}
```

- 把合并纯函数放 `utils/merge.js`（不放 `app.js`）：`app.js` 依赖 `wx` 环境不便单测，
  拆出来后 Node 可直接加载校验，后续分册/按需加载也只改一处。
- `mergeExtraData()` 内部先 `.slice()` 浅拷贝再合并：小程序 `require` 有模块缓存，
  原地改会污染缓存对象，多次初始化时出问题。
- 合并完成后跑一次**启动自检**（条目数 + id 连续性），结果挂 `globalData`，开发期一眼可见。

### 第 6 步：必须静态 require（BIM 小程序 P3 教训）

```js
const base = require('./data/data-base.js');
```

- 必须**字面量静态路径**，不要变量拼接、不要条件分支、不要动态路径；
  否则开发者工具 `ignoreDevUnusedFiles` 会把文件当成未使用从代码包里剔除，真机上「数据为空」。

### 第 7 步：验收脚本（本技能的核心资产）

至少覆盖 6 类校验：
| 类别 | 内容 |
|---|---|
| 结构性 | 工程文件齐备、`app.json` 声明的 pages 真实存在、静态 require 到位 |
| 条目数 | 每个数据集与基线逐一相等 |
| id 连续性 | 带 id 的数据集必须 1..N 连续、零重复零跳号（纯字符串数组除外） |
| 无空洞 | 对象字段不得出现 `undefined`（JSON 序列化不该产生空洞） |
| 网页端对照 | **真跑网页端原版合并函数**，两边条目数 + 首尾元素逐项比对 |
| 幂等 | 重跑转换脚本，比对三个生成物 sha256 字节级不变 |

用法约定：`node tools/校验_xxx.js`，**退出码即结论**（0 通过 / 1 失败），可挂 CI。

## 三、踩坑记录（重要）

1. **「多个 `<script>` 共享全局词法环境」在 Node 里不成立**。
   网页端 `script.js` 里若写了 `(function mergeExtraData(){ ...typeof EXTRA_X !== 'undefined'... })()`，
   它只在 `data-extra.js` 等文件都执行完之后才有效。
   Node 里把三段源码拼成一段执行时**顺序被改变**，IIFE 会在扩充数据就位前跑完，
   条件全部跳过 → **静默产出「数量全对、但一条扩充数据都没进去」的结果**。
   **对策：把合并 IIFE 从拼接串里摘出来，放到所有数据都就位之后执行。**
2. **`typeof X !== 'undefined'` 守卫会伪装错误**。它是真实前端的容错写法，
   但在搬迁场景会把「作用域不可见」伪装成「没有这份数据」，把错误吞掉。
   搬迁时必须**显式校验合并结果**，不能信守卫。
3. **三份源码里同名 `const` 会冲突**。拼接执行前把顶层 `const/let` 改写成 `var`
   （`text.replace(/^(const|let)(\s+[A-Za-z_$][\w$]*\s*=)/gm, 'var$2')`）即可，
   文件内部的 `const` 不受影响。
4. **`vm.createContext(sandbox)` 返回新对象，不是 sandbox 本体**：脚本里 `globalThis.x = ...`
   之后要从 **sandbox** 读，不是从 `ctx` 读。
5. **顶层 `const` 是脚本作用域的词法绑定**，不会出现在 sandbox 对象属性上；
   要在上下文内再跑一段 `globalThis.__raw = { NAME, ... }` 才能把它们取出来。
6. **要在 Node 里跑网页端数据层，必须给足浏览器桩**：`localStorage`（含 `getItem/setItem/removeItem/clear`）、
   `document`（`body/documentElement/head/getElementById/createElement/querySelector(All)/addEventListener`）
   都要给，否则脚本在到达合并逻辑之前就因 `localStorage is not defined` 中断——
   表现同样是「数量全对、扩充数据没进去」。
7. **PowerShell here-string 写文件会引入双重转义**（`\n` 变 `\\n`），
   生成脚本后务必跑一次 `node -c` 或 `ast.parse` 做语法自检。
8. **`Out-File -Encoding utf8` 在 Windows PowerShell 5.1 会写 BOM 与 CRLF**；
   要 LF 无 BOM 就用 `[System.IO.File]::WriteAllText($p, ($s -replace "\r\n","\n"), (New-Object System.Text.UTF8Encoding($false)))`。

## 四、验收标准（逐条打勾）

- [ ] 转换脚本可重复执行，输出字节级一致（sha256 断言通过）；
- [ ] 生成物头部含「自动生成，禁止手改 / 唯一真相 / 重新生成命令 / id 策略」；
- [ ] 小程序端每个数据集条目数与基线逐一相等；
- [ ] 带 id 的数据集 id 为 1..N 连续、零重复零跳号；
- [ ] 与网页端**真跑**的合并结果逐项一致（含首尾元素）；
- [ ] 生成物无 `undefined` 空洞；
- [ ] `app.js` 对数据文件为静态字面量 require；
- [ ] 项目内 `AGENTS.md` 写明「生成物禁止手改」「必须静态 require」；
- [ ] 验收脚本退出码可用（0/1），可挂 CI；
- [ ] 生成物体积在主包上限内（数据 + 逻辑 + 资源 < 2048 KB，否则规划分包）。

## 五、可复用产物（下次直接抄）

- 抽取器骨架：括号匹配 + 字符串/注释状态机（见第 2 步）；
- 临时 Node 求值器（第 3 步）；
- `appendWithIds()` / `mergeExtraData()` / `auditData()` 三件套（第 5 步）；
- 验收脚本 6 类校验框架（第 7 步）；
- 参考实现：`D:\接单客户\CET4-Toolkit\tools\脚本_数据转小程序模块.py`、
  `tools\校验_P1数据层与工程骨架.js`、`miniprogram\utils\merge.js`。
