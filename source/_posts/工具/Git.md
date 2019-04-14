---
title: Git
date: 2017-01-01 09:00:00
tags: [工具]
---

# 暂存

`git add -i` 进入交互模式

- status：展示已暂存的（staged）、未暂存的（unstaged）和该文件的路径。
- update：命令行提示符变为 `Update>>`，通过输入数字来选择文件，数字之间用空格或逗号分隔，如：2-3 5,6；数字前加减号取消选择。最后输入一个空行表示确认，被选择的文件会被暂存。
- revert：交互模式同 update。被选择的文件会用 HEAD 覆盖暂存区内容。
- add untracked：交互模式同 update。
- patch：选择一个文件，对文件内容分块依次决定是否暂存。
  - y：暂存此块。
  - n：不暂存此块。
  - q：取消并退出。
  - a：暂存此块以及后面的块。
  - d：不暂存此块以及后面的块。
  - g：选择并跳转到某个块。
  - /：用正则表达式搜索某个块。
  - j：暂时不决定此块，跳转到下一个未决定的块。
  - J：暂时不决定此块，跳转到下一个块。
  - k：暂时不决定此块，跳转到上一个未决定的块。
  - K：暂时不决定此块，跳转到上一个块。
  - s：将当前块继续分割成小块。
  - e：编辑当前块。
  - ?：帮助
- diff：展示 HEAD 和暂存区之间的区别。
- quit
- help

`git add -p` 同交互模式中的 patch。

# 提交

- `git commit`
  打开编辑器来撰写 commit 信息
- `git commit -m "message"`
  commit 到 Head，并附带备注 "message"
- `git commit -am "message"`
  结合了 add . 和 commit，不适用于新添加的文件
- `git commit --amend`
  修改最近一次提交，使用当前暂存区做提交，并可以修改提交信息

# 删除

- `git rm [-f|--force] [-n] [-r] [--cached] [--ignore-unmatch] [--quiet] [--] {filename}...`
  删除暂存区和工作区中的指定文件。该文件在三棵树中的内容必须相同，否则要用 `-f` 强制删除暂存区和工作区，或用 `--cached` 删除暂存区（此时暂存区必须要么跟工作区相同，要么跟版本库相同）。
  - `{filename}` 可以用通配符（要做 shell 转义）
  - `-f|--force` 强制删除暂存区和工作区中的指定文件（否则要求该文件在三棵树中的内容相同）
  - `-n|--dry-run` 预览操作结果
  - `-r` 指定目录做递归删除
  - `--cached` 只从暂存区删除
  - `--ignore-unmatch` 若未匹配到文件也不显示错误
  - `-q|--quiet` 不显示执行结果
- `git rm [-r] --cached {filename}`
  删除 index 中的文件，保留工作区中的文件，`-r` 表示递归
- `git clean [-d] [-f] [-i] [-n] [-q] [-e <pattern>] [-x | -X] [--] <path>...` 清除未跟踪的文件和目录
  - `-n` 预览将被删除的文件，不会直接删除。
  - `-d` 除了未跟踪的文件外，还删除未跟踪的目录。
  - `-f` `--force` 若未将 `clean.requireForce` 选项置为 false，则不能删除文件和目录，除非带有参数 `-f`/`-n`/`-i`。对于子目录中带有 .git 文件夹的，需要给出两个 `-f` 参数。
  - `-x` 扫描未跟踪文件时无视 .gitignore 和 $GIT_DIR/info/exclude，但仍会考虑 `-e` 参数。
  - `-X` 仅删除被 git 无视的文件。通常用来在清理的同时保留用户自建的文件。
  - `-q` `--quiet` 仅报告错误。
  - `-i` 交互式。

# 撤销

- `git reset HEAD {file}...` 用版本库覆盖暂存区
- `git checkout {file}...` 用暂存区覆盖工作区
- `git clean -fd` 删除未跟踪的文件和目录
- `git reset [--soft|--mixed|--hard] {HEAD~|commit_id}` 重置头指针
  - `soft` 只回退版本库，不回退暂存区和工作区；
  - `mixed` 缺省选项，回退版本库和暂存区；
  - `hard` 回退版本库、暂存区和工作区

# 反转

- `git revert [--[no-]edit] [-n] [-m parent-number] [-s] [-S[{keyId}]] {commit}...`
  - `-e` `--edit` 不自动提交反转，让用户先手动编辑提交信息。这是终端的默认参数。
  - `--no-edit` 自动提交反转。
  - `-n` `--no-commit` 反转但不自动产生提交。
  - `-m parent-number` `--mainline parent-number` 用于反转合并提交。
- `git revert --continue`
- `git revert --quit`
- `git revert --abort`

# 重命名

- `git mv {file_from} {file_to}`
相当于依次执行下面两条命令
  - `git rm {file_from}`
  - `git add {file_to}`

# 储藏

- `git stash [--keep-index] [-u] [save '{message}']`
  储藏当前所有未提交的改动，可附加备注；`--keep-index` 表示不储藏暂存区；`-u` 表示储藏未追踪的文件
- `git stash list`
  列出储藏栈中内容
- `git stash apply [stash@{n}] [--index]`
  恢复指定某次的储藏内容中的工作区文件，缺省为最近一次的；`--index` 表示恢复工作区和暂存区文件
- `git stash drop [stash@{n}] `
  删除储藏
- `git stash clear`
  清除所有储藏
- `git stash pop [stash@{n}] [--index]`
  应用并删除储藏
- `git stash branch {branch_name} [stash@{n}]`
  新建一个分支并应用储藏内容，如果没有冲突则会自动删除该储藏
- `git stash show [{stash}]`
  查看 stash 改动内容。可以接受参数来修改展示方式（同 `git diff`）

# 日志

- `git log -p -{n}`
显示最近 n 次的提交日志，并展开每次提交的内容差异
- `git log --stat`
仅显示简要的增改行数统计
- `git log --shortstat`
仅显示 `--stat` 中最后的行数修改添加移除的统计
- `git log --name-only`
仅在提交信息后显示已修改的文件清单
- `git log --name-status`
显示新增、修改、删除的文件清单
- `git log --abbrev-commit`
仅显示 SHA-1 的前几个字符
- `git log --relative-date`
使用较短的相对时间显示（如：“2 weeks ago”）
- `git log --graph`
显示 ASCII 图形表示的分支合并历史
- `git log --pretty`
使用其他格式显示历史提交信息。可用选项包括 `oneline`、`short`、`full`、`fuller`、`format`（后跟指定格式）
- `git log [--left-right] {b1}...{b2}`
只存在于 {b1} 和 {b2} 中的一个，而不同时存在于两者的提交；`--left-right` 会显示每个提交具体在哪个分支中

`git log [{options}] [{revision_range}] [[\\--] {path}...]`

- {options}
    - `--follow` 继续显示文件被移动或重命名前的历史，只针对单个文件有效。
    - `--no-decorate` `--decorate[=short|full|auto|no]`
    - `--source`
    - `--use-mailmap`
    - `--full-diff`
    - `--log-size`
    - `-L {start},{end}:{file}`
    - `-L:{funcname}:{file}`
- {revision_range}
- [\\--] {path}
- 提交范围限制
    - `-{number}`
    - `-n {number}`
    - `--max-count={number}`
    - `--skip={number}`
    - `--since={date}`
    - `--after={date}`
    - `--until={date}`
    - `--before={date}`
    - `--author={pattern}`
    - `--committer={pattern}`
    - `--greg-reflog={pattern}`
    - `--greg={pattern}`
    - `--all-match`
    - `--invert-greg`
    - `-i` `--regexp-ignore-case`
    - `--basic-regexp`
    - `-E` `--extended-regexp`
    - `-F` `--fixed-strings`
    - `--perl-regexp`
    - `--remove-empty`
    - `--merges`
    - `--nomerges`
    - `--min-parents={number}` `--max-parents={number}` `--no-min-parents` `--no-max-parents`
    - `--first-parent`
    - `--not`
    - `--all`
    - `--branches[={pattern}]`
    - `--tags[={pattern}]`
    - `--remotes[={pattern}]`
    - `--glob={glob-pattern}`
    - `--exclude={glob-pattern}`
    - `--reflog`
    - `--ignore-missing`
    - `--bisect`
    - `--stdin`
    - `--cherry-mark`
    - `--cherry-pick`
    - `--left-only`
    - `--right-only`
    - `--cherry`
    - `-g` `--walk-reflogs`
    - `--merge`
    - `--boundary`

# 衍合

- `git rebase -i {ref}`
其中 {ref} 为希望重排的提交的父提交。一般只修改提交信息或合并若干提交。若重排提交顺序或修改历史提交中的提交内容，容易发生冲突而需要大量手动解决。
- `git rebase [-i] {branch_to}`
将当前分支衍合进目标分支。完成后原本分开的两条提交线会合并成一条。

# 跟远程仓库交互

## remote

- `git remote [-v]`
  查看所有远程仓库，`-v` 额外查看远程仓库的地址
- `git remote show {remote_name}`
  查看远程仓库的详细信息
- `git remote add {short_name} {url}`
  添加一个远程仓库，并为其指定一个简称
- `git remote rename {old_name} {new_name}`
  重命名远程仓库的简称
- `git remote rm {remote_name}`
  移除远程仓库
- `git remote prune {remote}`
  清除本地无效的跟踪分支。

## clone

`git clone [-b {branch}] {url} [custom_repo_name]`
从服务器上克隆项目，包括所有历史数据。可自定义本地项目的名称，可指明克隆的分支。

## checkout

`git checkout [-b {local_branch_name}] --track {remote_repo}/{remote_branch}`
从远程分支检出一个跟踪分支，即可以直接运行 `git push` 和 `git pull` 的分支，本地分支的名字默认为 {remote_branch}

`git checkout -b {branch_name} {remote_name}/{remote_branch}`
从远程分支检出一个跟踪分支，并自定义本地分支名

## fetch & pull

`git fetch {remote_name}`
将远程仓库中的数据抓取到本地，更新的是跟踪分支的代码。

`git pull`
获取（fetch）并合并（merge）远程仓库的改动至本地

## push

`git push [-f] {remote_name} {branch_name}`
上传到远程仓库 的 {branch_name} 分支，`-f` 表示不做检查强制重写

`git push {remote_name} :{branch_name}` `git push origin --delete {branch_name}`
删除远程分支

`git push {remote_name} [tag_name]`
上传标签

`git push {remote_name} --tags`
上传所有标签

## branch

`git branch -r`
查看远程分支

# 分支

- `git branch`
查看本地分支
- `git branch -a`
查看所有分支
- `git branch -v`
查看各分支最后一次提交信息
- `git branch -vv`
查看各分支最后一次提交信息以及对应的 upstream 分支（若有的话）
- `git branch --merge|--no-merged`
查看已/未与当前分支合并的分支
- `git branch {new_branch_name}`
创建指定名称的新分支
- `git checkout {new_branch_name}`
切换到指定名称的分支
- `git checkout -b|--orphan {new_branch_name} [{start_point}]`
创建并切换到指定名称的新分支或空白的新分支。可以指定创建点，通常是 commit-id。
- `git branch [-d|-D] [-r|--remotes] [{remote}/]{branch_name}`
  - `-d`：分支已经合并到主干后删除分支；
  - `-D`：强制删除。
  - `-r|--remotes`：删除跟踪分支。
- `git branch -u {remote}/{remote_branch} [{local_branch}]`
  `git branch --set-upstream-to={remote}/remote_branch} [{local_branch}]`
对已有分支绑定跟踪分支

# 差异比较

`git diff {source_branch} {target_branch}`
查看两个分支之间的差异
`git diff {source_branch}...{target_branch}`
查看 target 分支与 source 和 target 共同祖先之间的差异
`git diff`
暂存区到工作区的变化
`git diff --cached|--staged`
版本库到暂存区的变化
`git diff HEAD`
版本库到工作区的变化

git维护代码分为三部分：“工作区 working directory”、“暂存区 index”、“版本库 head”（分别标注为1，2，3）。
`git add` 完成1->2
`git commit` 完成2->3
`git commit -a` 两者结合
`git diff` 2到1的变化
`git diff --cached|--staged` 3到2的变化
`git diff HEAD` 3到1的变化

# 合并

`git merge [-n] [--stat] [--no-commit] [--squash] [--[no-]edit] [-s <strategy>] [-X <strategy-option>] [-S[<keyid>]] [--[no-]allow-unrelated-histories] [--[no-]rerere-autoupdate] [-m <msg>] [<commit>…]`

- `--stat` `--no-stat` `-n` 显示/不显示合并的差异数据。
- `--commit` `--no-commit` 合并后是否自动提交。
- `--edit`/`-e` `--no-edit` 合并后是否打开编辑器（来编辑提交信息）。
- `--ff` `--no-ff` 是否使用“fast-forward”合并（若可行）。
- `--ff-only` 若不能“fast-forward”则取消合并。
- `-S[<keyid>]` `--gpg-sign[=<keyid>]` 对提交做签名。
- `--log[=<n>]` `--no-log` 将每个被合并的提交转换为单行描述，再将这些描述添加到此次合并的提交信息中。
- `--squash` `--no-squash` 将待合并的所有提交压缩在一起合并到当前工作区，然后需要手动提交这些改动。跟 `--no-ff` 的区别在于 `--squash` 只是合并内容，不合并提交线。
- `-Xours|-Xtheirs` 若出现冲突则直接使用当前分支/对方分支的内容。
- `-m <msg>` 设置此次合并的提交信息（若合并产生一个新提交的话）。

`git merge --abort`

当合并发生冲突且等待用户解决时即处于合并过程中间状态，此时可以 `git merge --abort` 取消合并，冲突文件会回滚至合并操作之前的状态。

对于合并开始时未提交的文件，若合并开始后发生了修改，那么合并取消后这些文件可能无法正常回滚。

`git merge --continue`

合并发生冲突后继续合并流程（通常先解决冲突）。

# 其他

- `ssh-keygen -C myemail -t rsa`
  创建 ssh 密钥
- `git gc`
  压缩松散对象
- `git cherry-pick {SHA-1}`
  将指定提交的代码引入当前分支
- `{ref}^[n]`
  引用 {ref} 的父提交，或第 n 父提交（只在合并提交时有用）
- `{ref}~[n]`
  引用 {ref} 的第一父提交，或第一父提交的第一父提交……
- `{b1}..{b2}`
  在 {branch2} 而不在 {branch1} 中的提交
- `{b1} {b2} ^{b3}` `{b1} {b2} --not {b3}`
  在 {b1}、{b2} 而不在 {b3} 中的提交，可比较多个分支的提交差异
- `git blame [-C] [-L m,n] {file}`
  查看文件内容的最近修改者。`-C` 可查看文本行的来源文件（如果该行是复制过来的话）；`-L m,n` 将查看范围限制在第 m ~ n 行
- `gitk --all` 包括全部分支

# 文件状态生命周期

```seq
untracked->unmodified:add the file
unmodified->modified:edit the file
modified->staged:stage the file
staged->unmodified:commit
unmodified->untracked:remove the file
```

# git config

## user

`user.name "John"`
`user.email John@example.com`
配置用户信息。不加 `--global` 则仅对当前项目有效（.git/config），加上 `--global` 则全局有效（C:/Users/$USER/.gitconfig）。提交的时候会用到用户信息。

`user.signingkey {gpg-key-id}`
GPG 签署密钥，用来创建经签署的含附注的标签。

## core

`core.editor emacs`
配置文本编辑器。其中 emacs 可以换成其他编辑器的名字。

`core.pager {pager}`
运行诸如 log、diff 等所使用的分页器，可设置为 more 或任何其他分页器（缺省为 less），设置为空字符串''则会在一页显示所有内容。

`core.excludesfile {files}`
无需纳入 Git 管理的文件，类似 .gitignore 文件。

`core.autocrlf true|false|input`
换行符转换。

- true：提交时 CRLF -> LF；签出时 LF -> CRLF
- false：不做改变
- input：提交时 CRLF -> LF；签出时不做改变

`core.safecrlf true|false|warn`
提交包含混合换行符的文件时的行为

- true：拒绝提交包含混合换行符的文件
- false：允许提交包含混合换行符的文件
- warn：提交包含混合换行符的文件时给出警告

`core.whitespace {var1,var2,...,varn}`
探测和修正空白问题的选项。在某个参数前加上 `-` 表示禁用。

- `blank-at-eol`：将行尾的空格作错误来处理（缺省开启）。
- `space-before-tab`：将位于行首缩进部分的 tab 符号前的空格作错误来处理（缺省开启）。
- `indent-with-non-tab`：将用空格来缩进而非用 tab 符号的行作为错误来处理（缺省关闭）。
- `tab-in-indent`：将行首缩进部分存在 tab 符号的行作为错误来处理（缺省关闭）。
- `blank-at-eof`：将文件末尾的空白行作为错误来处理（缺省开启）。
- `trailing-space`：包含了 `blank-at-eol` 和 `blank-at-eof`。
- `cr-at-eol`：将行尾的回车符 cr 当作行结尾的一部分，也就是说，行尾回车符前若存在空白符不会触发 `trailing-space`。
- `tabwidth=<n>`：设定 1 个 tab 符号占几个空格；此项与 `indent-with-non-tab` 和 `tab-in-indent` 相关。缺省为 8。允许范围为 1 到 63。

`core.quotepath`
是否对文件路径中的非拉丁字符进行转义。

## help

`help.autocorrect {1|0}`
若设成 1，则在只有一个命令被模糊匹配到的情况下，Git 会自动运行该命令。

## merge

`merge.tool vimdiff`
配置差异分析工具。可选的还有 kdiff3、tkdiff、meld、xxdiff、emerge、vimdiff、gvimdiff、ecmerge、opendiff 等。

## commit

`commit.template {file}`
提交时使用的模板文件。

## push

`push.default`
push 时的默认行为。Git2.0 之前默认使用 `matching`，之后默认使用 `simple`。

```
git branch --set-upstream-to={remote}/{branch} [{local_branch}]
// 或者 git branch -u {remote}/{branch} [{local_branch}]
// 或者 git push --set-upstream {remote} [{local_branch}:]{remote_branch}
```

- `nothing`：push 操作无效，除非显式指定远程分支：`git push origin develop`。
- `current`：push 当前分支到远程同名分支，如果远程同名分支不存在则自动创建同名分支。
- `upstream`：push 当前分支到它的 [upstream](#git中的upstream和downstream) 分支上（这一项其实用于经常从本地分支 push/pull 到同一远程仓库的情景，这种模式叫做 central workflow）。
- `simple`：simple 和  upstream 是相似的，只有一点不同，simple 必须保证本地分支和它的远程 upstream 分支同名，否则会拒绝 push 操作。
- `matching`：push 所有本地和远程两端都存在的同名分支。

## branch

`branch.{branch_name}.merge`
merge 时的默认操作。在 pull 的时候也会用到。

```
[branch "develop"]
    remote = origin
    merge = refs/heads/develop
```

或者通过 bash 设置：

```
git config branch.develop.merge refs/heads/develop
```

为什么不是refs/remotes/develop？

> 因为这里 merge 指代的是我们想要 merge 的远程分支，是站在远程仓库的角度看到的引用路径。
 在本地仓库来看，远程仓库的分支引用路径为 `refs/remotes/{branch}`；站在远程仓库的角度来看，这个分支引用路径自然就是 `refs/heads/{branch}`。
 和我们在本地直接执行 `git merge` 是不同的（本地执行 `git merge origin/develop` 则是直接 merge refs/remotes/develop）。

## other
`git config --list`
查看配置信息。其中重复的变量来自不同的配置文件，Git 会采用最后一个。

`git config user.name`
查看特定变量。

# git help

`git help {verb}`
`git {verb} --help`
`man git-{verb}`
查看特定命令 `{verb}` 的帮助信息。

# git tag

- **通用参数**
    - `{commit}` 某次提交的校验 ID。缺省为 HEAD。
    - `{object}` 某个对象的校验 ID。缺省为 HEAD。

- **查看标签**
> git tag [-n[{num}]] -l [--contains {commit}] [--points-at {object}] [--column[={options}] | --no-column] [--create-reflog] [--sort={key}] [--format={format}] [--[no-]merged [{commit}]] [{pattern}…]

    - `无参数` 查看当前版本库的标签列表。
    - `-n{num}` 查看所有标签及对应的注释。通过 {num} 来指定显示多少行注释；缺省只显示注释的第一行。若某标签无注释，则显示其对应的提交信息。
    - `-l {pattern}` 利用通配符对标签名进行过滤。若未给出通配符，则显示所有标签；若给出多个通配符，则标签只要匹配其中之一就会显示。
    - `--contains {commit}` 只列出指定提交上的标签，缺省为 HEAD。
    - `--points-at {object}` 只列出指定对象上的标签。
    - `--column[={options}]` `--no-column` 分别表示“总是”、“总不”以列形式展示标签。只在显示不带注释的标签时有效。其中 {options} 包括：
        - 以下选项控制开关
            - `always` 总是以列形式显示。
            - `never` 总不以列形式显示。
            - `auto` 若输出到终端，则以列形式显示。
        - 以下选项控制布局，缺省为`column`。若出现以下任一选项，而上述3个选项未出现，则默认为`always`。
            - `column` 先填充列，后填充行。
            - `row` 先填充行，后填充列。
            - `plain` 显示在一列。
        - 以下选项可与上述布局选项混合使用，缺省为`nodense`。
            - `dense` 列宽不相等。
            - `nodense` 列宽相等。
    - `--create-reflog` 为标签创建操作记录。
    - `--sort={key}` 按照 {key} 对显示结果进行升序排列。通过添加`-`前缀来进行降序排列。若指定多个 {key}，则最后一个 {key} 将作为主键。缺省为`tag.sort`的值，否则就按字典顺序。
    - `--format={format}` A string that interpolates %(fieldname) from the object pointed at by a ref being shown. The format is the same as that of git-for-each-ref(1). When unspecified, defaults to %(refname:strip=2).
    - `--[no-]merged [{commit}]` Only list tags whose tips are reachable, or not reachable if --no-merged is used, from the specified commit (HEAD if not specified).

- **创建标签**
> git tag [-a | -s | -u {keyid}] [-f] [-m {msg} | -F {file}] {tagname} [{commit} | {object}]

    - `无参数` 创建轻量级标签。该标签指向一个提交，无标签创建过程记录。
    - `-a` 创建一个无签名的、带注释的标签。带注释的标签指向一个标签对象（而不是一个提交），该对象中包含了创建标签时的说明、对应的提交 ID 等信息。
    - `-s` 创建一个 GnuPG 签名的、带注释的标签，使用默认 e-mail 地址作为公钥/私钥对。
    - `-u {keyid}` 创建一个 GnuPG 签名的、带注释的标签，使用指定的 {keyid} 作为公钥/私钥对。
    - `-m {msg}` 若创建的是带注释的标签，则必须使用 `-m {msg}` 参数来创建该标签的注释。若使用了 `-m {msg}` 而未使用 `-a`、`-s` 或 `-u {keyid}`，则会隐含使用 `-a`。
    - `-F {file}` 读取指定文件的内容作为标签注释。`-F -` 表示从标准输入中读取注释。若使用了 `-F {file}` 而未使用 `-a`、`-s` 或 `-u {keyid}`，则会隐含使用 `-a`。

- **删除标签**
> git tag -d {tagname}...

- **校验标签**
> git tag -v {tagname}...

    - 校验指定标签的 GnuPG 签名。

- **其他**
    - `git show {tagname}` 查看指定标签的详细信息。
    - `git log --decorate` 在查看日志时显示提交对应的标签及其他引用。
    - `git describe [{commit}]` 将提交显示为一个易记的名字。该名字来自于建立于该提交之上的标签。
        - 若该提交存在标签，则显示该标签的名字。
        - 若该提交没有标签，则显示其最近一个存在标签的历史版本的标签名，并以 `{tag}-{num}-g{commit}` 的格式显示。其中 {tag} 是该历史版本的标签名，{num} 是该历史提交与所查提交之间的距离，{commit} 是所查提交的精简 ID。
        - `--dirty` 加上该参数后，若工作区的文件被修改过，则会在标签名后面显示“-dirty”。

# .gitignore 文件

在 `.git` 同级目录下，手动创建 `.gitignore` 文件

```
 # 井号表示注释
 # 文件名/目录名中可用正则表达式

 # 指定忽略某个文件
abc.txt

 # 指定忽略某个目录
abc/

 # 指定不忽略某个文件/目录
!def.txt
!def/

 # 指定忽略某一类文件
*.doc
 # 使用标准的 glob 模式匹配
*.[ch]
```

# 乱码

**乱码情景1**

在 cygwin 中，使用 `git add` 添加要提交的文件的时候，如果文件名是中文，会显示形如 `\274\232\350\256\256\346\200\273\347\273\223.png` 的内部码点形式。

解决方案：

在 bash 提示符下输入：
```
git config --global core.quotepath false
```
core.quotepath 设为 false 的话，就不会对 0x80 以上的字符进行 quote。中文显示正常。

**乱码情景2**

在 MsysGit 中，使用 `git log` 显示提交的中文 log 乱码。

解决方案：

- 设置 git gui 的界面编码
```
config --global gui.encoding utf-8
```
- 设置 commit log 提交时使用 utf-8 编码，可避免服务器上乱码，同时与 linux上 的提交保持一致
```
git config –global i18n.commitencoding utf-8
```
- 使得在 `git log` 时将 utf-8 编码转换成 gbk 编码，解决Msys bash中 `git log` 乱码。
```
git config –global i18n.logoutputencoding gbk
```
- 使得 `git log` 可以正常显示中文（配合 `i18n.logoutputencoding = gbk`)，在 `/etc/profile` 中添加：
```
export LESSCHARSET=utf-8
```

**乱码情景3**

在 MsysGit 自带的 bash 中，使用 `ls` 命令查看中文文件名乱码。cygwin 没有这个问题。

解决方案：

使用 `ls --show-control-chars` 命令来强制使用控制台字符编码显示文件名，即可查看中文文件名。
为了方便使用，可以编辑 `/etc/git-completion.bash`，新增一行 `alias ls="ls --show-control-chars"`

**乱码情景4**

使用 `$GIT_HOME/bin/bash.exe` 和 `$GIT_HOME/git-cmd.exe` 执行 `git log` 中文呈乱码，而在 `git-bash.exe` 中不会。

解决方案：

1. 添加环境变量 `LESSCHARSET=utf-8`
2. 若是在 IDEA 的控制台功能使用上述两个终端软件，则需要在 `设置-Tools-Terminal-Environment Variables` 中添加上述变量。

**终极解决方案**

终极的解决方案是通过修改 git 和 TortoiseGit 源码实现，有网友这么做了：[让Windows下Git和TortoiseGit支持中文文件名/UTF-8][3] ，也可以直接访问这个开源的 Google 项目：[utf8-git-on-windows][4] 。
如果不抗拒命令行的话，直接用 Cygwin 来提交Git库。因为 Cygwin 其实是一个在 Windows 平台上的模拟器，它完全模拟 GNU/Linux 的方式运行，所以 Cygwin 中的 Git 是采用 UTF-8 编码来保存中文的。

# git 中的 upstream 和 downstream

git 中存在 upstream 和 downstream，简言之，当我们把仓库 A 中某分支 x 的代码 push 到仓库 B 分支 y，此时仓库 B 的这个分支 y 就叫做 A 中 x 分支的 upstream，而 x 则被称作 y 的 downstream，这是一个相对关系，每一个本地分支都相对地可以有一个远程的 upstream 分支（注意这个 upstream 分支可以不同名，但通常我们都会使用同名分支作为 upstream）。
