# 《Pro Git》读书笔记和cheatsheet

[阅读地址](https://git-scm.com/book/zh/v2)

## Git概念

1. 暂存区、工作区
2. 已追踪的文件和未追踪的文件
3. 检出
4. `.gitignore`
5. `commit`
6. 轻量标签与附注标签
7. Git内部的数据对象，`blob`
8. 分支
9. 快进合并（fastforward merge）、三分合并
10. 合并冲突
11. 远程引用
12. 追踪分支
13. 变基
14. 不能使用变基的情况（如果提交存在于你的仓库之外，而别人可能基于这些提交进行开发，那么不要执行变基。）
15. 拣选（cherry-pick）
16. Rerere
17. 三棵树（HEAD、索引工作目录

## `.gitignore` 格式规范

文件 `.gitignore` 的格式规范如下：

* 所有空行或者以 # 开头的行都会被 Git 忽略。
* 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
* 匹配模式可以以（/）开头防止递归。
* 匹配模式可以以（/）结尾指定目录。
* 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。

## Git基本命令（文件增删改查与提交）

### 查看暂存区和工作区状态

```sh
git status
```

### 以简略格式查看暂存区和工作区状态

```sh
git status -s
```

示例输出：

```sh
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```

其中，`??`表示为追踪，`M`表示已修改，`A`表示新建文件。输出中有两栏，左栏指明了暂存区的状态，右栏指明了工作区的状态。

### 追踪新文件

```sh
git add <file>
```

### 查看暂存区和工作区之间的文件差异（diff）

```sh
git diff
```

### 查看暂存区和上次提交之间的文件差异（diff）

```sh
git diff --staged
```

### 跳过暂存区直接提交

```sh
git commit -a
```

### 从工作区中删除文件

```sh
git rm <file>
```

注意，当删除操作不可撤回时（例如已经修改过或者已经提交到暂存区），则git将会拒绝删除。
处理办法是使用`-f`选项强制要求删除。

### 在工作区中移动（重命名）文件

```sh
git mv <oldfile> <newname>
```

### 查看提交历史

```sh
git log
```

可以搭配使用的扩展选项：

* 查看各个提交之间的diff：

```sh
git log -p
```

* 限制输出的commit数：

```sh
git log -2
```

* 输出总结性信息：

```sh
git log --stat
```

* 以单行格式输出：

```sh
git log --pretty=oneline
```

* 定制输出格式

```sh
git log --pretty=format:"%h - %an, %ar : %s"
```

* 以图形的形式输出

```sh
git log --graph
```

* 查看特定时间的历史
利用`--since`和`--after`加上时间可以查看特定时间的历史。这个时间可以是具体的时间（例如`2023-8-15`）或者模糊的相对时间（例如`2.weeks`）。

```sh
 git log --since=2.weeks
```

* 查找那些添加或删除了该字符串的提交

```sh
git log -S str
```

或者改用`-G`选项指定正则表达式。

* 查找某一行或者某一个函数的所有提交历史

```sh
git log -L :>git log -L :<function name>:<source file>
```

* 查看来某分支减去另一个分支的提交（排除）

```sh
git log <branch-A> --not <branch-B>
```

此命令可以用来确定将要合并什么内容。

### 查看B相对与A和B分支公共祖先的更改（diff）

```sh
git diff <A-branch>...<B-branch>
```

### 查看提交简报

```sh
git shortlog --no-merges master
```

会将各个提交按作者汇总。

### 修正上一个提交

```sh
git commit --amend
```

### 撤销对文件的修改（回到上一次提交的内容）

```sh
git checkout -- <file>
```

### 从暂存区移除文件

```sh
git reset HEAD <file>
```

## 重写提交

```sh
git rebase -i HEAD~<n>
```

`<n>`是你要修改的提交数量，从`HEAD`开始。当然也可以使用别的指定提交范围的办法。这将会启用一个交互式的命令行，git将会引导你如何操作。

通过正确的编辑git生成的交互式文件，可以做到重写提交、重排提交、删除提交等等。在此文件中提交的排列顺序是旧提交在上，新提交在下。

例如，假设git生成了如下的交互式文件：

```sh
edit f7f3f6d changed my name a bit
pick 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
```

将其修改成：

```sh
edit f7f3f6d changed my name a bit
pick 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
```

此时git将会在第一个提交上停下来，此时可以使用`git commit --amend`修改提交。
如果改为：

```sh
pick 310154e updated README formatting and added blame
pick f7f3f6d changed my name a bit
```

则会跳过提交`310154e`（但是最终的快照结果不变）。
如果改为：

```sh
pick f7f3f6d changed my name a bit
squash 310154e updated README formatting and added blame
squash a5f4a0d added cat-file
```

则会将三个提交压缩为一个提交。
如果改为：

```sh
pick f7f3f6d changed my name a bit
edit 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
```

则会在第二个提交上停下来。此时可以用`git reset HEAD^`取回文件并重做提交。

另一个方法是使用`git filter-branch`，这可以对整个仓库所有提交进行改写。例如，从提交中移除一个文件：

```sh
git filter-branch --tree-filter 'rm -f <file>' HEAD
```

使一个子目录成为新的根目录：

```sh
git filter-branch --subdirectory-filter <dir> HEAD
```

全局修改邮箱地址：

```sh
git filter-branch --commit-filter '
        if [ "$GIT_AUTHOR_EMAIL" = "<old-email>" ];
        then
                GIT_AUTHOR_EMAIL="<new-email>";
                git commit-tree "$@";
        else
                git commit-tree "$@";
        fi' HEAD
```

## Git远程仓库操作

### 列出所有远程仓库

```sh
git remote
```

或使用简略形式：

```sh
git remote -v
```

### 添加远程仓库

```sh
git remote add <shortname> <url>
```

### 重命名远程仓库

```sh
git remote rename <oldname> <newname>
```

### 删除远程仓库

```sh
git remote remove <name>
```

### 查看远程仓库

```sh
git remote show <name>
```

或者列出所有远程仓库：

```sh
git remote show
```

### 抓取数据

```sh
git fetch <remote>
```

该操作不会合并任何分支。

## Git标签操作

### 列出标签

```sh
git tag
```

或者使用模式进行查找，例：

```sh
git tag -l "v1.8.5*"
```

### 创建附注标签

```sh
git tag -a <tag> -m "<message>" <commit hash>
```

当省略提交校验和时，将会在上一个提交上打标签。

### 创建轻量级标签

```sh
git tag <name>
```

### 删除标签

```sh
git tag -d <tag>
```

如果要从远程服务器上删除标签，使用下面两种命令之一：

```sh
git push origin :refs/tags/<tag>
git push origin --delete <tag>
```

### 共享标签

默认情况下，`git push` 命令并不会传送标签到远程仓库服务器上。 在创建完标签后你必须显式地推送标签到共享服务器上。
推送单个标签：

```sh
git push <remote> <tag>
```

推送所有标签：

```sh
git push <remote> --tags
```

### 检出标签

```sh
git checkout <tag>
```

或检出到新的分支：

```sh
git checkout -b <branch> <tag>
```

### 验证标签

```sh
git tag -v <tag-name>
```

## 分支

### 查看所有分支

```sh
git branch
```

### 查看各个分支当前所指的对象

```sh
git log --oneline --decorate
```

### 创建分支

```sh
git branch <name>
```

这并不会检出新创建的分支。要创建分支后立刻检出分支，可以使用：

```sh
git log --oneline --decorate --graph --all
```

### 删除分支

```sh
git branch -d <name>
```

### 检出分支

```sh
git checkout <name>
```

### 查看项目分叉历史

```sh
git log --oneline --decorate --graph --all
```

### 合并分支

```sh
git merge <branch>
```

### 查看已合并分支和未合并分支

```sh
git branch --merged # 查看已合并分支
git branch --no-merged # 查看未合并分支
```

在这个已合并分支列表中分支名字前没有 * 号的分支通常可以使用 git branch -d 删除掉；你已经将它们的工作整合到了另一个分支，所以并不会失去任何东西。

### 推送分支

```sh
git push <remote> <branch>
```

推送到远程的不同名分支：

```sh
git push <remote> <local-branch>:<remote-branch>
```

### 拉取分支

对于有追踪分支的本地分支，运行：

```sh
git pull
```

会从远程分支下载数据并合并远程分支到本地分支。

### 从远程分支新建分支

```sh
git checkout -b <local-branch> <remote-branch>
```

或者：

```sh
git checkout --track <rmemote branch>
```

如果你尝试检出的分支 (a) 不存在且 (b) 刚好只有一个名字与之匹配的远程分支，那么如下操作会从远程分支创建一个新分支：

```sh
git checkout <branch> 
```

以上操作均会自动设置追踪分支。

### 移除跟踪分支

```sh
git branch --unset-upstream
```

### 删除远程分支

```sh
git push <remote> --delete <branch>
```

### 找出两个分支的公共祖先

```sh
git merge-base <branch-A> <branch-B>
```

### 变基

```sh
git rebase
```

对A分支减去B分支（求差集）的结果变基：

```sh
git rebase --onto <branch> <B-branch> <A-branch>
```

## Stash

### 将当前工作区加入Stash

```sh
git stash push
```

默认情况下，会将已经加入暂存区的内容也一并加入stash，并情况暂存区。可以使用如下选项：
`--keep-index`这样会保持暂存区不变。可以使用`-u`选项包括未跟踪的文件，`-a`包括被忽略的文件。

### 列出贮藏

```sh
git stash list
```

### 应用贮藏

```sh
git stash apply <number>
```

注意这并不会删除贮藏。

### 删除贮藏

```sh
git stash drop <number>
```

### 应用并删除贮藏

```sh
git stash pop <number>
```

### 从贮藏创建新分支

```sh
git stash branch <new branchname>
```

## 子模块

### 添加子模块

```sh
git submodule add <url for submodule>
```

### 克隆含有子模块的仓库

在克隆一个新仓库后，默认子模组还没有被初始化，这时可以用如下命令完成初始化：

```sh
git submodule update --init --recursive
```

### 更新子模块

子模块和一般的git仓库一样，都可以手动拉取并合并远程分支。但是使用如下命令可以自动更新所以子模块：

```sh
git submodule update --remote
```

这会拉取并检出所以模块的子分支。注意这不会合并或提交拉取的更改。可以使用`--merge`选项来自动合并子模块更改。

可以使用如下方法设置要跟踪的分支

```sh
git config -f .gitmodules submodule.<submodule>.branch <branch> 
```

当在主项目拉取（`git pull`）时，默认只会拉取子模组的更改但不会应用。应当使用选项`--recurse-submodules`使得子模组有正确的状态。

### 推送子模组更新

如果对子模组做出了更改，那么在push主项目时要先把所有子模组先push到远端。可以手动操作，也可以使用如下命令自动完成：

```sh
git push --recurse-submodules=on-demand
```

### 遍历子模组

可以使用如下命令来对每一个子模组都进行相同操作：

```sh
git submodule foreach '<command>'
# for example: git submodule foreach 'git checkout -b featureA'
```

## 搜索

### 搜索工作目录、提交历史等

```sh
git grep <pattern>
```

使用`--line-number`(`-n`)输出行号。使用`--count`(`-n`)输出统计信息。使用`--show-function`(`-p`)显示匹配所在的方法或函数。

使用`--break`启用额外的断行，`--heading`将文件名显示在文件中的匹配项之上，而不是在每一行的开头显示。这些能提升可读性。

使用`-e`指定匹配模式，`-n`表示取反，`--and`表示与，`--or`表示或，利用括号分组，可以构建更加强大的搜索。

## Misc

### 启用重用已记录的冲突解决方案（reuse recorded resolution）

```sh
git config --global rerere.enabled true
```

### 手工创建Git对象

```sh
git hash-object -w --stdin
```

将会从标准输入读取数据并存储到一个`blob`。

例如，将GPG密钥导入到Git：

```sh
gpg -a --export <hash> | git hash-object -w --stdin
```

### 自动生成构建号

```sh
git describe <tagname>
```

### 清理工作目录

```sh
git clean
```

这会清理所有**未跟踪的**且**未被忽略的**的文件。可以用`-d`选项清理空目录，`-n`进行dry run，`-x`同时清理所有被忽略的文件，`-i`交互式运行。

### 打包源代码

```sh
git archive master --prefix='<prefix>/' --format=zip > a.zip
```

### 设置GPG密钥

```sh
git config user.signingkey <gpg-key-hash>
```

### 签署提交

```sh
git commmit -S
```

如果决定在正常的工作流程中使用它，你必须确保团队中的每一个人都理解如何这样做。 如果没有，你将会花费大量时间帮助其他人找出并用签名的版本重写提交。

### reset的三步走

```sh
git reset <commit-hash>
```

`reset` 命令会以特定的顺序重写这三棵树，在你指定以下选项时停止：

1. 移动 HEAD 分支的指向 _（若指定了 `--soft`，则到此停止）_
2. 使索引看起来像 HEAD _（若未指定 `--hard`，则到此停止）_
3. 使工作目录看起来像索引

当指定了路径时，将会跳过第一步，并执行第二、三步，不过只限于给定的路径。这个命令可以用来取消某些暂存文件（通过重置索引区为上一个提交时的状态），还可以撤销某些文件的修改。

### 标注文件

使用`git blame`标注文件，这会输出一个标注文件，每一个行的开头都显示了这行上一次修改的提交。

```sh
git blame -L <start lineno>,<end lineno> <file>
```

使用`-C`将会计算出代码可能的原始位置。这些代码可能从别的文件复制过来的（重构），Git能够帮你分析出来。

### 二分查找

启动二分查找的流程：

```sh
git bisect start # 开始二分查找
git bisect bad   # 标记当前commit为坏
git bisect good v1.0 # 标记上一个好的commit
# 这样我们就指定了二分查找的范围
```

现在，`git`已经检出了中点提交。在此运行一些测试，以确定代码是好是坏，然后标记这个commit：

```sh
git bisect good # 如果代码是好
git bisect bad  # 如果代码是坏
```

现在git会修改搜索区间并检出下一个中点提交，重复此过程直到找到搜索区间为空。
然后，回退到之前的`HEAD`：

```sh
git bisect reset
```

### 别名

使用如下命令创建别名：

```sh
git config alias.<alias-name> "command"
```

例如：

```sh
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
```

## 最佳实践

### 如何写commit信息

这里是一份 [最初由 Tim Pope 写的模板](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)

> 首字母大写的摘要（不多于 50 个字符）
>
> 如果必要的话，加入更详细的解释文字。在大概 72 个字符的时候换行。
> 在某些情形下，第一行被当作一封电子邮件的标题，剩下的文本作为正文。
> 分隔摘要与正文的空行是必须的（除非你完全省略正文），
> 如果你将两者混在一起，那么类似变基等工具无法正常工作。
>
> 使用指令式的语气来编写提交信息：使用“Fix bug”而非“Fixed bug”或“Fixes bug”。
> 此约定与 git merge 和 git revert 命令生成提交说明相同。
>
> 空行接着更进一步的段落。
>
> \- 标号也是可以的。
>
> \- 项目符号可以使用典型的连字符或星号，后跟一个空格，行之间用空行隔开，
> 但是可以依据不同的惯例有所不同。
>
> \- 使用悬挂式缩进

## 手册表格

### 客户端钩子

| 钩子                 | 说明                                                                                                                                             |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `pre-commit`         | 在键入提交信息前运行。 它用于检查即将提交的快照，例如，检查是否有所遗漏，确保测试运行，以及核查代码。 如果该钩子以非零值退出，Git 将放弃此次提交 |
| `prepare-commit-msg` | 在启动提交信息编辑器之前，默认信息被创建之后运行。 它允许你编辑提交者所看到的默认信息。                                                          |
| `commit-msg`         | 接收一个参数，此参数即存有当前提交信息的临时文件的路径。 如果该钩子脚本以非零值退出，Git 将放弃提交。                                            |
| `post-commit`        | 在整个提交过程完成后运行。不接受任何参数。                                                                                                       |
| `pre-rebase`         | 钩子运行于变基之前，以非零值退出可以中止变基的过程。                                                                                             |
| `post-rewrite`       | 钩子被那些会替换提交记录的命令调用，比如 `git commit --amend`` 和`git rebase`                                                                    |
| `post-checkout`      | 在`git checkout`成功运行后被调用                                                                                                                 |
| `post-merge`         | 在`git merge`成功运行后被调用                                                                                                                    |
| `pre-push`           | 会在 `git push` 运行期间， 更新了远程引用但尚未传送对象时被调用。                                                                                |

### `git log --pretty=format`常用的选项

| 选项 | 说明                                          |
| ---- | --------------------------------------------- |
| %H   | 提交的完整哈希值                              |
| %h   | 提交的简写哈希值                              |
| %T   | 树的完整哈希值                                |
| %t   | 树的简写哈希值                                |
| %P   | 父提交的完整哈希值                            |
| %p   | 父提交的简写哈希值                            |
| %an  | 作者名字                                      |
| %ae  | 作者的电子邮件地址                            |
| %ad  | 作者修订日期（可以用 --date=选项 来定制格式） |
| %ar  | 作者修订日期，按多久以前的方式显示            |
| %cn  | 提交者的名字                                  |
| %ce  | 提交者的电子邮件地址                          |
| %cd  | 提交日期                                      |
| %cr  | 提交日期（距今多长时间）                      |
| %s   | 提交说明                                      |

### `git log` 的常用选项

| 选项            | 说明                                                                                                        |
| --------------- | ----------------------------------------------------------------------------------------------------------- |
| -p              | 按补丁格式显示每个提交引入的差异。                                                                          |
| --stat          | 显示每次提交的文件修改统计信息。                                                                            |
| --shortstat     | 只显示 --stat 中最后的行数修改添加移除统计。                                                                |
| --name-only     | 仅在提交信息后显示已修改的文件清单。                                                                        |
| --name-status   | 显示新增、修改、删除的文件清单。                                                                            |
| --abbrev-commit | 仅显示 SHA-1 校验和所有 40 个字符中的前几个字符。                                                           |
| --relative-date | 使用较短的相对时间而不是完整格式显示日期（比如“2 weeks ago”）。                                             |
| --graph         | 在日志旁以 ASCII 图形显示分支与合并历史。                                                                   |
| --pretty        | 使用其他格式显示历史提交信息。可用的选项包括 oneline、short、full、fuller 和 format（用来定义自己的格式）。 |
| --oneline       | --pretty=oneline --abbrev-commit 合用的简写。                                                               |

### 限制 `git log` 输出的选项

| 选项                  | 说明                                       |
| --------------------- | ------------------------------------------ |
| `-<n>`                | 仅显示最近的 n 条提交。                    |
| `--since`, `--after`  | 仅显示指定时间之后的提交。                 |
| `--until`, `--before` | 仅显示指定时间之前的提交。                 |
| `--author`            | 仅显示作者匹配指定字符串的提交。           |
| `--committer`         | 仅显示提交者匹配指定字符串的提交。         |
| `--grep`              | 仅显示提交说明中包含指定字符串的提交。     |
| `-S`                  | 仅显示添加或删除内容匹配指定字符串的提交。 |

### reset和checkout的行为

|                             | HEAD | Index | Workdir | WD Safe? |
| --------------------------- | ---- | ----- | ------- | -------- |
| Commit Level                |      |       |         |          |
| reset --soft [commit]       | REF  | NO    | NO      | YES      |
| reset [commit]              | REF  | YES   | NO      | YES      |
| reset --hard [commit]       | REF  | YES   | YES     | NO       |
| checkout \<commit\>         | HEAD | YES   | YES     | YES      |
| File Level                  |      |       |         |          |
| reset [commit] \<paths\>    | NO   | YES   | NO      | YES      |
| checkout [commit] \<paths\> | NO   | YES   | YES     | NO       |

“HEAD” 一列中的 “REF” 表示该命令移动了 HEAD 指向的分支引用，而 “HEAD” 则表示只移动了 HEAD 自身。 特别注意 'WD Safe?' 一列——如果它标记为 **NO**，那么运行该命令之前请考虑一下。

## Git常用配置

| 项目                | 描述                                              |
| ------------------- | ------------------------------------------------- |
| `core.editor`       | 文本编辑器                                        |
| `core.pager`        | 分页器                                            |
| `merge.tool`        | 设置合并工具                                      |
| `user.signingkey`   | GPG签署密钥                                       |
| `core.excludesfile` | 忽略文件列表（额外的`.gitignore`）                |
| `help.autocorrect`  | 在设定值乘以0.1秒之后自动修正错误命令（较为危险） |
| `color.ui`          | 自动着色功能（默认值为`auto`）                    |
| `core.autocrlf`     | 自动转换换行符（默认为`true`）                    |
