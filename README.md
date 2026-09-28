# my-rules

我的**全局规则**的源仓库。运行时目录只读——这里是唯一的编辑点。

## 单向数据流（体系公理）

```
改这里（唯一编辑点）                     同步        ~\.trae-cn\user_rules\（每次对话全量注入）
D:\Seed\my-rules\rules\      ────────────────────►   只读，不要手改
```

**为什么不能直接改运行时**：`user_rules\` 里的改动会生效，但它不在任何 git 里，而且等同步一到就被冲掉——等于白改 + 丢改动。

体系全貌见 `retro-skills/docs/architecture.md`。

## 目录

```
my-rules\
└── rules\     规则的源文件（有意义命名，由同步脚本落到运行时）
```

> **为什么是中立名 `rules\`（而不是 `.trae\rules\`）**：**换编辑器是很基础的需求**。
> 曾经用过 `.trae\rules\`（`ais` 生态的"每个工具一个源目录"约定），但那个布局的两条理由都没兑现：① `ais` 在本机不可用（它不认识 Trae CN 的 `.trae-cn` 路径），装完就卸了；② "多工具扩展位"不是白捡的——加第二个工具要么建跨目录链接、要么格式得兼容。
> 而我们的内容本来就是**纯 markdown、不含工具专有 frontmatter**，天生中立 → 所以把**目录名也中立化**：换编辑器时**只改同步项目的配置（`D:\Seed\agent-assets-sync\sync.config.json` 里那条映射的 `dstDir` 与 `dstName`），源文件一个字都不用动**。
> 技能那侧早就是这个形态（`retro-skills\skills\<名>\` 用的是跨工具的 Agent Skills 约定），现在两侧一致了。

## 收编账目（bootstrap 于 2026-09-28）

| 本仓库文件 | 来源 | 来源文件 |
|---|---|---|
| `rules/import-guard.md` | 运行时（活着） | `user_rules\rule-1788769641982.md` |
| `rules/no-git-write.md` | 运行时（活着） | `user_rules\rule-1790090235459.md` |
| `rules/poll-deferred-at-start.md` | 运行时（活着） | `user_rules\rule-1790523252016.md` |
| `rules/numbers-must-be-measured.md` | 运行时（活着） | `user_rules\rule-1790562609979.md` |
| `rules/plain-language-to-user.md` | 运行时（活着） | `user_rules\rule-1790564060830.md` |
| `rules/code-style.md` | **死文件复活** | `~\.agents\rules\code-style.md` |
| `rules/testing-pitfalls.md` | **死文件复活** | `~\.agents\rules\testing-pitfalls.md` |
| `rules/powershell-file-encoding.md` | **新写**（非收编） | 2026-09-28 由复盘体系落地（BOM 教训第二次踩中） |
| `rules/landing-sweep.md` | **新写**（非收编） | 2026-09-28 评审第 4 轮落地（"落地后回头扫"的检查点） |

**「死文件复活」是什么意思**：这两条规则一直躺在磁盘上，但 Trae **从不读取** `~\.agents\rules\` 这个目录，所以它们**从未生效过**。收编并经同步进入 `user_rules\` 后，才第一次真正生效。

规则正文**逐字保留原文**（收编的 7 条未作任何改写；后加的 2 条为新建）。

## 现状与待办

**已完成**：仓库建好（git + GitHub 远程）｜全部规则已同步进运行时（符号链接；**条数取 `aas` 体检输出里的"已就位"数**，不在此重复）｜同步/体检脚本已抽成**独立项目** `agent-assets-sync`（配置驱动，命令 `aas`）｜文件命名机制已验证

**换编辑器怎么办**（这是选中立布局的收益）：源不动，只改 `D:\Seed\agent-assets-sync\sync.config.json` 里那条映射的 `dstDir` 与 `dstName` —— 例如换成 Claude Code 时，目标路径与命名规则按那时查到的官方约定改即可（规则是纯 markdown，一般可直接用）

**待办**：

| # | 事项 | 说明 |
|---|---|---|
| — | `poll-deferred-at-start.md` **待改版** | 已定三处改动但尚未落改：① 触发点从「每次开工」改为「阶段收口」② 未命中**静默不出声** ③ 脚本不可用就跳过，别手工模拟 |
| — | 给运行时加"并发写保护" | 同 KB 的 H4（两个会话同时写会撞） |

## 安装与同步

**同步**（不要在运行时目录里手改）—— 脚本住在**独立项目** `agent-assets-sync` 里，命令叫 **`aas`**（注册一次：在项目里 `npm link` 或 `npm run localg`）：

```
aas                       只体检（默认，绝不改动任何东西）
aas sync                  建缺失的链接
aas sync --replace        允许把真实副本换成链接
aas sync --rm-old         允许清理残留
aas add / update / remove / import   安装形态（从远端装、刷新、摘除、收编手建条目）
```

**换机器 / 别人来装**（路径全在 `agent-assets-sync\sync.config.json` 里，**不用改代码**）：

1. clone 本仓库 + `retro-skills` 仓库 + `agent-assets-sync` 仓库
2. Node ≥ 20 + git + npm；**Windows 必须开「开发者模式」**（规则那侧用符号链接，靠它）
3. 装 aas：进 `agent-assets-sync` 跑 `npm install` 后 `npm link`（或 `npm run localg`）
4. 需要时设环境变量覆盖路径（变量名与路径的绑定写在配置的 `env` 字段里；下表的"默认值"就是配置里的值）：

| 环境变量 | 作用 | 默认值 |
|---|---|---|
| `MY_RULES_SRC` | 规则源目录 | `D:\Seed\my-rules\rules` |
| `RETRO_SKILLS_DIR` | 技能源目录 | `D:\Seed\retro-skills\skills` |
| `TRAE_RULES_DST` | Trae CN 全局规则目录 | `~\.trae-cn\user_rules` |
| `TRAE_SKILLS_DST` | Trae CN 技能目录 | `~\.trae-cn\skills` |

5. 跑 `aas sync`（或 `aas` 先体检看看）

**只想要规则内容、不想装脚本**：在 Trae 里「设置 → 规则 → 创建 → 全局」粘贴正文即可（官方文档写明的唯一方式）

**两条硬约束**（直接往目录放文件时必须遵守，否则**静默不生效**）：

- 运行时文件名**必须**是 `rule-<名字>.md`（少了 `rule-` 前缀 → Trae 不加载）
- 全局规则**没有 frontmatter**，是纯 markdown 正文

## 禁止

- 直接改 `~\.trae-cn\user_rules\`（会被同步冲掉）
- 直接改 `~\.trae-cn\skills\` 里的技能副本（同上，且会静默失效）
- 改 `~\.agents\rules\`（Trae 不读该目录，改了等于白改）
