# git
## git提交解决本地与远程冲突
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
## 介绍
本地git仓库和远程仓库冲突是指当你试图将你的本地代码推送到一个远程仓库时，发现远程仓库已经有了和你的本地代码不同的修改，导致Git无法自动合并它们。你需要手动解决冲突，然后再推送你的代码。

有几种方法可以解决本地git仓库和远程仓库冲突，比如：

+ 使用git stash命令将你的本地修改暂时保存起来，然后使用git pull命令将远程代码拉取到你的本地仓库，然后使用git stash pop命令将你的本地修改合并到更新后的代码中去，解决冲突后再提交你的代码。
+ 使用git reset --hard命令将你的本地代码重置到上一次提交的状态，然后使用git pull命令将远程代码拉取到你的本地仓库，然后用你备份过的本地修改覆盖你的本地文件，解决冲突后再提交你的代码。
+ 使用git mergetool命令启动一个合并工具，比如vim或Kaleidoscope，让你在编辑器中选择要保留的修改。

你可以根据你的喜好和需求选择合适的方法来解决本地git仓库和远程仓库冲突。

### Understanding Git Pull Conflicts(理解Git Pull冲突)

`Pull`(拉取命令) merges changes frome a `remote` repository into a `local` repository.When there are conflicting changes in the same file,a git pull conflict occurs.`Git diff` command can be used to find conflicts between two branches 

Git pull命令将远程仓库中的更改合并到本地仓库中。当同一个文件中有冲突的更改时，会发生git pull冲突。 Git diff命令可以用来查找两个分支之间的冲突。

It is important to keep pull requests up-to-date to avoid conflicts. Formatting rules should be established and followed to avoid conflicts.

保持pull request是最新的以避免冲突是很重要的。应建立并遵循格式规则以避免冲突。

> GitLens是一个能够增强VS Code中Git功能的插件，它能够解锁每个仓库中隐藏的知识。它帮助你通过Git blame注释和CodeLens快速地查看代码的作者信息，无缝地导航和浏览Git仓库，通过丰富的可视化和强大的比较命令获得有价值的洞察，以及更多其他功能。

## 为什么我的clash没有dashboard

因为没有所以没有,我找到的 clash for linux 的版本是不带控制界面的,需要访问 这个 web 控制台 才能进行控制.后来为了方便访问,我就把他的包都给抓了下来,放在了本地的 nginx 上.得有个clash-ui(但是原项目已经被删掉sad)

https://clash.razord.top/#/proxies

[https://www.joeyne.cool/http/proxy/ubuntu-安装clash并配置开机启动/#clash-for-linux](https://www.joeyne.cool/http/proxy/ubuntu-安装clash并配置开机启动/#clash-for-linux)
```shell

# 进入下载目录（默认情况是下载到 ~/Downloads 目录，如果不是请进入到对应的下载目录）
cd ~/Downloads
# 解压
gunzip clash-linux-amd64-v3-v1.15.1.gz
# 重命名
mv clash-linux-amd64-v3-v1.15.1 clash
# 添加可执行权限(解压后是一个可执行文件，如果没有执行权限，需要手动添加）
chmod +x clash
# 复制clash 到/usr/bin/文件夹(这样在终端任何位置执行 clash 即可启动)
sudo mv clash /usr/bin/
```


```shell
[Unit]
Description=clash

[Service]
Type=simple
ExecStart=/usr/bin/clash -f /home/along/desk/clash/config.yaml

[Install]
WantedBy=multi-user.target
```
## 关于git和github
ssh的密钥只能来验证ssh协议相关的,你如果使用的是https的就不能验证.所以添加remote的时候要注意,如果是ssh的话,就要用ssh的地址,如果是https的话,就要用https的地址.

### 更改或者添加git remote仓库的地址
``` bash
git remote set-url origin yours_github_repository_url
git remote add origin yours_github_repository_url
# 此处origin可以自行修改
```
### 强制推送
如果你不在乎`main`分支的内容，并且你想要将`pa1`分支的内容强制推送到`main`分支，你可以使用以下命令：

```bash
# 切换到pa1分支
git checkout pa1

# 强制推送到远端的main分支
git push origin +pa1:main
# origin是远端仓库的名称，+pa1:main表示将本地的pa1分支强制推送到远端的main分支
```

这里的`+`符号表示强制推送。这将会覆盖远端的`main`分支，使其与你本地的`pa1`分支完全相同。

请注意，强制推送是一种破坏性操作，它会永久地覆盖远端的`main`分支。在你执行这个操作之前，你应该确保你不需要`main`分支上的任何更改，或者你已经备份了这些更