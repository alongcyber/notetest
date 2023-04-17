# git
### git提交解决本地与远程冲突
[how-to-resolve-git-pull-conflicts-and-overwrite-local-changes-step-by-step-guide](https://copyprogramming.com/howto/how-to-resolve-git-pull-conflicts-and-overwrite-local-changes-step-by-step-guide)
> markdown的引用语法 
``` markdown
    [title](url)
    <http://baidu.com>
    [tutorial][1]

    [1]: https://baidu.com
```
[tutorial][1]

[1]: https://copyprogramming.com/howto/how-to-resolve-git-pull-conflicts-and-overwrite-local-changes-step-by-step-guide
### 介绍
本地git仓库和远程仓库冲突是指当你试图将你的本地代码推送到一个远程仓库时，发现远程仓库已经有了和你的本地代码不同的修改，导致Git无法自动合并它们。你需要手动解决冲突，然后再推送你的代码。

有几种方法可以解决本地git仓库和远程仓库冲突，比如：

+ 使用git stash命令将你的本地修改暂时保存起来，然后使用git pull命令将远程代码拉取到你的本地仓库，然后使用git stash pop命令将你的本地修改合并到更新后的代码中去，解决冲突后再提交你的代码。
+ 使用git reset --hard命令将你的本地代码重置到上一次提交的状态，然后使用git pull命令将远程代码拉取到你的本地仓库，然后用你备份过的本地修改覆盖你的本地文件，解决冲突后再提交你的代码。
+ 使用git mergetool命令启动一个合并工具，比如vim或Kaleidoscope，让你在编辑器中选择要保留的修改。

你可以根据你的喜好和需求选择合适的方法来解决本地git仓库和远程仓库冲突。

#### Understanding Git Pull Conflicts(理解Git Pull冲突)

`Pull`(拉取命令) merges changes frome a `remote` repository into a `local` repository.When there are conflicting changes in the same file,a git pull conflict occurs.`Git diff` command can be used to find conflicts between two branches 

Git pull命令将远程仓库中的更改合并到本地仓库中。当同一个文件中有冲突的更改时，会发生git pull冲突。 Git diff命令可以用来查找两个分支之间的冲突。

It is important to keep pull requests up-to-date to avoid conflicts. Formatting rules should be established and followed to avoid conflicts.

保持pull request是最新的以避免冲突是很重要的。应建立并遵循格式规则以避免冲突。

> GitLens是一个能够增强VS Code中Git功能的插件，它能够解锁每个仓库中隐藏的知识。它帮助你通过Git blame注释和CodeLens快速地查看代码的作者信息，无缝地导航和浏览Git仓库，通过丰富的可视化和强大的比较命令获得有价值的洞察，以及更多其他功能。

