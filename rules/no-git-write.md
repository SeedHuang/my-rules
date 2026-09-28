# 全局 Git 写操作禁令

## 核心约束
除非用户在当前对话中明确、直接地要求执行某个 Git 写操作，否则你禁止主动执行任何 Git 写操作。

## 被禁止的 Git 写操作（除非用户明确要求）
以下操作一律禁止主动执行：
- git commit
- git push
- git pull（当它会修改工作区或产生合并时）
- git merge
- git rebase
- git reset
- git revert
- git cherry-pick
- git checkout（当它用于切换分支或丢弃更改时）
- git restore
- git clean
- git stash
- git tag（创建或删除标签时）
- git branch（创建、删除或重命名分支时）
- git remote（修改远程配置时）
- git config（任何写入配置的操作）
- git add（除非用户明确要求暂存文件）
- 任何会修改 .git 目录内容的命令

## 允许的 Git 读操作（无需用户额外要求）
以下读操作可以正常执行：
- git status
- git log
- git diff
- git show
- git branch（仅查看分支列表时）
- git remote -v（仅查看远程列表时）

## 例外条件（必须同时满足）
只有当用户在当前对话中**明确说出**类似于以下表述时，才可以执行对应的写操作：
- “帮我 commit”
- “执行 git push”
- “帮我提交代码”
- “创建分支”
- 其他直接、无歧义的 Git 写操作指令

## 遇到不确定情况时的行为
如果你不确定某个 Git 操作是否属于写操作，或者不确定用户是否已明确授权，必须：
1. 暂停执行该操作；
2. 向用户说明你打算执行什么 Git 命令；
3. 等待用户明确确认后再继续。

## 绝对禁止的行为
- 禁止在用户不知情的情况下自动提交代码；
- 禁止使用 --force 或 -f 参数执行任何 Git 操作；
- 禁止修改 Git 全局配置或远程仓库配置；
- 禁止对 main/master 分支执行强制推送。
