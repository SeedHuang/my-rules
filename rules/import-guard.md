# 编辑代码时禁止丢失 import

> **🚨 一切编辑的前置自检：同一文件只发一次 SearchReplace，import 和代码合并进去，改完立刻运行 `npx tsc --noEmit` 验证（不能用 GetDiagnostics，它返回的是 TS Server 缓存结果，编辑后立即调用不可靠）。**

## 硬规则 0：对同一个文件禁止并行 SearchReplace（P0）

**绝对禁止**在同一轮 tool calls 中向同一个文件发出两次及以上 `SearchReplace`。

原因：并行 SearchReplace 都基于编辑前的同一份文件快照。后者成功后会覆盖前者的改动（silent-fail），导致前面改的 import、代码全部回退。

```tsx
// ❌ 致命错误：并行两次 SearchReplace 改同一个文件
// 调用 1: SearchReplace(file="a.tsx", old_str="import {X}...", new_str="import {X,Y}...")
// 调用 2: SearchReplace(file="a.tsx", old_str="<div>...", new_str="<Y>...")
// → 调用 2 返回成功，调用 1 的 import 改动被覆盖，Y 找不到

// ✅ 正确：合并为一次 SearchReplace
// 用一次 SearchReplace，old_str 从 import 行开始，覆盖到代码改动行
```

## 硬规则 1：import 变更和代码变更必须在同一次 SearchReplace 中完成

绝不分两步：先加 import，再改代码。两步会导致后一次覆盖前一次。

```tsx
// ❌ 致命错误：分两步
// Step 1: 加 import
// Step 2: 用新组件替换代码（old_str 基于编辑前 → silent-fail → 文件回退，import 丢失）

// ✅ 正确：合并为一次
```

## 硬规则 2：每使用一个新变量/对象/函数，先自查

使用 `useMemo`、新 styled-component、新工具函数等任何新变量前，先确认：

1. 这个变量是 import 来的还是本地声明的？
2. 如果是 import 来的，import 行是否已包含它？
3. 如果没有，必须把新增 import 和使用它的代码写进**同一次 SearchReplace** 的 old_str / new_str

## 硬规则 3：分阶段编译检查（P0）

每次编辑后做**两级检查**，兼顾速度与完整性。禁止使用 `GetDiagnostics`，因为 TS Server 在编辑后存在缓存延迟，可能返回过时的零错误结果。

### 第一级：每次编辑后（单文件检查，快）

**每次 SearchReplace 或 Write 编辑一个 .ts/.tsx/.js/.jsx 文件后，立即运行 `npx tsc --noEmit --pretty 2>&1 | grep "src/pages/编辑文件所在目录"` 检查该文件是否有编译错误。**

- `grep` 范围控制在该文件所在目录（如 `src/pages/CustomerServiceCustomer`），输出仅限该目录的文件，速度快
- 只检查本文件引入的错误（import 遗漏、模块路径错误、类型不匹配）

```tsx
// ✅ 正确：编辑后 grep 目录检查
SearchReplace(file="src/pages/X/index.tsx", ...)  // 编辑
→ RunCommand("npx tsc --noEmit --pretty 2>&1 | grep \"src/pages/X\"")
→ 修复直到零错误
→ 下一个工具
```

### 第二级：所有编辑完成后（全量检查，确保跨文件连锁错误）

**所有文件的编辑全部完成后，运行一次不带 grep 的全量 `npx tsc --noEmit --pretty 2>&1`。**

- 捕获跨文件的连锁类型错误（改 A 文件导致 B 文件报错）
- 忽略已知已有错误清单（见下方）

```tsx
// ✅ 最终验证：全量 tsc
→ RunCommand("npx tsc --noEmit --pretty 2>&1", blocking=true)
→ 检查输出中是否有非忽略清单内的新错误
```

### 已知的已有编译错误（不属于本次修改引入）

以下文件在编辑前就存在编译错误，`tsc` 输出中如果只包含这些错误可以安全忽略：

- `src/app.tsx` — `location` 不在 `UmiHistory` 类型上
- `src/pages/404/index.tsx` — `back` 不在 `UmiHistory` 类型上
- `src/setup/theme.tsx` — 自定义 token 不在 antd 类型定义中
