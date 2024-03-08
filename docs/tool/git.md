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
本地git仓库和远程仓库冲突是指当你试图将你的本地代码推送到一个远程仓库时,发现远程仓库已经有了和你的本地代码不同的修改,导致Git无法自动合并它们.你需要手动解决冲突,然后再推送你的代码.

有几种方法可以解决本地git仓库和远程仓库冲突,比如：

+ 使用git stash命令将你的本地修改暂时保存起来,然后使用git pull命令将远程代码拉取到你的本地仓库,然后使用git stash pop命令将你的本地修改合并到更新后的代码中去,解决冲突后再提交你的代码.
+ 使用git reset --hard命令将你的本地代码重置到上一次提交的状态,然后使用git pull命令将远程代码拉取到你的本地仓库,然后用你备份过的本地修改覆盖你的本地文件,解决冲突后再提交你的代码.
+ 使用git mergetool命令启动一个合并工具,比如vim或Kaleidoscope,让你在编辑器中选择要保留的修改.

你可以根据你的喜好和需求选择合适的方法来解决本地git仓库和远程仓库冲突.

### Understanding Git Pull Conflicts(理解Git Pull冲突)

`Pull`(拉取命令) merges changes frome a `remote` repository into a `local` repository.When there are conflicting changes in the same file,a git pull conflict occurs.`Git diff` command can be used to find conflicts between two branches 

Git pull命令将远程仓库中的更改合并到本地仓库中.当同一个文件中有冲突的更改时,会发生git pull冲突. Git diff命令可以用来查找两个分支之间的冲突.

It is important to keep pull requests up-to-date to avoid conflicts. Formatting rules should be established and followed to avoid conflicts.

保持pull request是最新的以避免冲突是很重要的.应建立并遵循格式规则以避免冲突.

> GitLens是一个能够增强VS Code中Git功能的插件,它能够解锁每个仓库中隐藏的知识.它帮助你通过Git blame注释和CodeLens快速地查看代码的作者信息,无缝地导航和浏览Git仓库,通过丰富的可视化和强大的比较命令获得有价值的洞察,以及更多其他功能.

## 为什么我的clash没有dashboard

因为没有所以没有,我找到的 clash for linux 的版本是不带控制界面的,需要访问 这个 web 控制台 才能进行控制.后来为了方便访问,我就把他的包都给抓了下来,放在了本地的 nginx 上.得有个clash-ui(但是原项目已经被删掉sad)

https://clash.razord.top/#/proxies

[https://www.joeyne.cool/http/proxy/ubuntu-安装clash并配置开机启动/#clash-for-linux](https://www.joeyne.cool/http/proxy/ubuntu-安装clash并配置开机启动/#clash-for-linux)
```shell

# 进入下载目录（默认情况是下载到 ~/Downloads 目录,如果不是请进入到对应的下载目录）
cd ~/Downloads
# 解压
gunzip clash-linux-amd64-v3-v1.15.1.gz
# 重命名
mv clash-linux-amd64-v3-v1.15.1 clash
# 添加可执行权限(解压后是一个可执行文件,如果没有执行权限,需要手动添加）
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
如果你不在乎`main`分支的内容,并且你想要将`pa1`分支的内容强制推送到`main`分支,你可以使用以下命令：

```bash
# 切换到pa1分支
git checkout pa1

# 强制推送到远端的main分支
git push origin +pa1:main
# origin是远端仓库的名称,+pa1:main表示将本地的pa1分支强制推送到远端的main分支
```

这里的`+`符号表示强制推送.这将会覆盖远端的`main`分支,使其与你本地的`pa1`分支完全相同.

请注意,强制推送是一种破坏性操作,它会永久地覆盖远端的`main`分支.在你执行这个操作之前,你应该确保你不需要`main`分支上的任何更改,或者你已经备份了这些更

## 关于Github Cli的个人访问令牌
[介绍Git Personal Access Token](https://deepinout.com/git/git-questions/763_git_where_to_store_my_git_personal_access_token.html)

存储个人访问令牌的选项
以下是几种常见的个人访问令牌存储选项：

1. 存储在环境变量中
将个人访问令牌存储在环境变量中是一种常见的做法.通过设置环境变量,可以轻松地在命令行中访问令牌,而不需要在每次使用Git命令时都手动输入.例如,在Linux和Mac上,可以将个人访问令牌添加到~/.bashrc或~/.bash_profile文件中：

```shell

export GIT_TOKEN=your_token_here

```

然后,通过访问$GIT_TOKEN来使用令牌,这样可以很好的解决我的一个痛点,就是我会在一个平台有不同账户的仓库,这样一个ssh key会在其他账户用了,这样我的一些github仓库的push就会成问题,如果有了这个环境变量存储个人访问令牌的功能之后,我只需要对应不同的账户设置不同的环境变量就可以了,例如`along_git_token`,`cyberalong_token`.



2. 存储在Git配置文件中
Git配置文件（~/.gitconfig）是存储Git配置信息的文件.可以在其中添加一个新的配置项来存储个人访问令牌.在配置文件中添加以下内容：

```bash

[github]
    token = your_token_here

```
在这种情况下,可以使用git config命令访问令牌：

```bash

$ git config --global github.token your_token_here

```
3. 存储在密钥管理器中
如果使用的是密码管理工具,例如Keychain（Mac OS）或Credential Manager（Windows）,可以将个人访问令牌存储在这些工具中.这样,令牌将被安全地存储,不易泄露,在需要时可以自动提取.

## 如何在ubuntu20.04中更新vscode
[如何在ubuntu20.04中更新vscode](https://www.joeyne.cool/http/vscode/如何在ubuntu20.04中更新vscode/#如何在ubuntu20.04中更新vscode)
### 如何查看vscode的安装方式
要查看在 Ubuntu 中安装的 Visual Studio Code (VSCode) 的方式,您可以执行以下步骤：

1. **检查软件包管理器：** 如果您是通过包管理器（如apt）安装的VSCode,则可以通过以下命令查看：

    ```bash
    apt list --installed | grep code
    ```

    如果输出中显示有关`code`的条目,则说明您是通过包管理器安装的VSCode.

2. **检查Snap包：** 如果您是通过Snap包管理器安装的VSCode,则可以使用以下命令：

    ```bash
    snap list | grep code
    ```

    如果输出中显示了与VSCode相关的条目,则说明您是通过Snap包安装的.

3. **检查安装目录：** 如果您手动安装了VSCode,您可以查看其安装目录,通常情况下,它会安装在 `/usr/share/code` 或者 `~/.vscode` 目录下.您可以使用以下命令来查找：

    ```bash
    which code
    ```

    这会显示VSCode的可执行文件的位置.

通过这些步骤,您可以确定您的系统中VSCode的安装方式.