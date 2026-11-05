---
description: coding
mode: primary
temperature: 0
---

# 项目概览

pi coding agent 插件 monorepo，两个插件作为独立 npm 包发布，包名与原独立仓库一致。仓库级基础设施（tsconfig、eslint、oxlint、oxfmt、CI、git hooks、依赖安装）只在根声明一次；`packages/<name>/` 下是各包自己的 manifest（deps、peerDeps、pi.extensions）、README 与源码。两插件互相独立、无共享运行时代码。host 提供的包（`@earendil-works/pi-coding-agent`、`typebox`）只在根作为 devDependencies 声明用于本地类型检查，各包仅以 `peerDependencies` 声明，遵循 pi 的包规范。

- `packages/pi-replace-function/`：提供 `replace_function` 工具，AI 给出函数名和完整的新声明，工具按语法树定位旧函数后原位替换。动机：避免 hashline edit 逐行 hash 污染上下文与缓存，避免整段替换重新输出未变更代码；函数是编程中最普遍的修改单元，按函数粒度替换能覆盖大多数修改场景。
- `packages/pi-reuse-model/`：让每个新会话从本 workspace 上次使用的模型和 thinking level 开始。

# replace-function：已定决策

- 解析器统一用 `@ast-grep/napi`，只做 kind 级节点查询；不引入 ast-grep 规则 DSL，也不直接操作 tree-sitter API。
- 定位顺序：`name` → `params_num`（防重载）→ `line`（1-based 精确匹配）。候选为零或不唯一必须中止并列出现状，不允许盲目替换。
- content 校验要求恰好一个最外层声明节点。tree-sitter 对缺失括号能容错恢复且无 error node，但残缺片段不再构成声明——这条规则靠语法结构而非错误节点截获截断内容，不要改弱。
- 方法体语法要求外层类型的语言（TS/JS/Java/C#）解析 content 时外包一层 `class Wrap { … }`，使方法可以顶格传入。
- 旧声明上方紧跟的注释默认保留；新 content 自带注释块则整体替换之。注释语法跟随语言（`//`、`#`、`--`）。
- 重命名天然支持：新 content 声明 `bar` 即把 `foo` 改名为 `bar`，回复中说明。

# replace-function：关键实现事实

- 语言表在 `packages/pi-replace-function/src/langs.ts`：TS/TSX/JS 由 ast-grep 内置，其余语言加载时从 tree-sitter npm 包动态注册（均为预编译产物，安装无需编译工具链）。表中 kind/field 在加载时由 ast-grep 按语法校验，未知 kind 直接 throw；新增语言 = 加表项 + 加对应依赖。
- 替换流程（`packages/pi-replace-function/src/replace.ts`）：拼接范围取自语法树；新 content 重新缩进到旧声明的缩进；BOM 与 CRLF/LF 原样保留；写前整体重新 parse，出现新语法错误即拒绝写入；写入走 pi 的 per-file mutation queue 串行化。

# reuse-model

- 存储文件 `~/.pi/agent/pi-reuse-model.json`，按 workspace 目录为键。文件名常量在 `packages/pi-reuse-model/src/index.ts`，改名会丢失所有用户已存的模型选择。

# 开发

根目录 `pnpm verify` = fmt:check + lint + tsc + test，覆盖全部包，提交前必须通过。发版走 Release workflow（workflow_dispatch），对两个包分别 `npm publish`；发布前需手动 bump 对应包的 version。
