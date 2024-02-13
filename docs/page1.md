# 建站的记录
## Another heading

Some more shit
### 一些沟是
哈哈哈把喇叭卡和单位,很喜欢我的键盘敲击的声音,有一种清新愉悦的快感,谁懂阿,给虚拟机新增了32GB内存,顿时流畅了许多,输入法不再有莫名的顿挫.
### 这些天
半角字母的美,谁懂,感觉很多事情,没有必要弄的不一样,懂我意思么.
### 关于这个礼拜
这个礼拜有些事情步入了正规,比如考研,比如coding,比如五级流水线CPU,比如CSAPP,比如我对一些事情的认识.
积极使用半角标点符号的好处(`ctrl`+`up`)=>可以以,.作为分割的依据.
## 开始实现代码高粱
文档看的不够仔细.
开始嗯造
=== "Customization"
  
    - nav
    - js
    - css
    目前了解的情况,大概就是nav是navigator导航,js应该跟katex有点关系,css大概或许已经解决?(haha)
    接下来想做的事情,~~大概就是好好的写note了.~~,大概就是梳理清除nav的用法,然后改一下下icon,然后开始嗯造note

=== "很开心的事情"
  
    目前总算把git最基本的一些东西搞的比较清楚了,还没弄清楚的是merge这些多线开发,团队协作的东西,但是我单人开发,单项开法.

    好像markdown也有点这个就近原则的意思,听难绷的.

    这个星期接下来的时间就会是恩造,积分高数balabla.

    Changing the logo and icons
    When installing Material for Mkdocs,you immediately get access to over 8,000 icons ready to be used for customization  of specific parts of the theme and/or then writing your documentation in Markdown.Not enough

## 关于table
| shit  | shit    |
|-------|------r lobortis orci

=== "Ordered list"
   |
|`get`  |:material-check: Fetch resource|
|`PUT`  |:material-check-all: Update res|

->r lobortis orci

=== "Ordered list"


=== "C"

    ``` c
    #include <stdio.h>

    int main(void) {
      printf("Hello world!\n");
      return 0;
    }
    ```

=== "C++"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```
``` markdown title="Conten tabs with code blocks"
在markdoown中输入
=== "C"

    ``` c
    #include <stdio.h>

r lobortis orci

=== "Ordered list"


=== "C++"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ``` 
```

=== "Unordered list"

    * Sed sagittis eleifend rutrum
    * Donec vitae suscipit est
    * Nulla tempor lobortis orci

=== "Ordered list"

    1. Sed sagittis eleifend rutrum
    2. Donec vitae suscipit est
    3. Nulla tempor lobortis orci

一定要空格阿,并且有缩进(嵌套制表符)
需要两次缩进

## nav(导航)


## neovim
+  四种mode`command`,`insert`,`normal`,`view`.
+  `Esc`  => `normal` => `i`,`I`,`o`,`O`,`a`,`A` => `insert`=>`Esc` => `normal`=>`ctrl+v`=>`view`=>`:`=>`command`

+ `0`

## 使用多个密钥来在多个github账号上切换
How can I configure multiple SSH keys for different GitHub accounts?
To configure multiple SSH keys for different GitHub accounts, you can follow these steps:

1. **Generate SSH keys**: For each GitHub account, generate a new SSH key (if you haven't already) with a unique email address.

```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

When prompted, give each key a unique name, such as `id_rsa_account1`, `id_rsa_account2`, etc.

2. **Add SSH keys to ssh-agent**: Start the ssh-agent in the background and add your new SSH private key to the ssh-agent.

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa_account1
ssh-add ~/.ssh/id_rsa_account2
```

3. **Add SSH keys to GitHub accounts**: For each GitHub account, go to Settings -> SSH and GPG keys -> New SSH key, then paste the contents of the corresponding public key file (e.g., `id_rsa_account1.pub`, `id_rsa_account2.pub`, etc.).

4. **Create SSH config file**: Create a new configuration file for SSH in your user directory, if it doesn't already exist.

```bash
touch ~/.ssh/config
```

5. **Edit SSH config file**: Open the SSH config file in a text editor, and add an entry for each GitHub account.

```bash
# Account 1
Host github.com-account1
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_account1

# Account 2
Host github.com-account2
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_account2
```

6. **Use SSH config in Git**: When cloning a repository, use the Host alias you set in the SSH config file.

```bash
git clone git@github.com-account1:username/repo.git
```

Replace `username` with your GitHub username and `repo` with your repository name. Use the correct Host alias for the account you want to use.

This way, you can use multiple SSH keys for different GitHub accounts.

### 关于tweak
这个ubuntu上的改变全局的东西有点神奇,可以改字体,系统语言设置的更改还得`reboot`,不过现在结果还行.(half punctuations)



#### 关于clash 
要在Ubuntu 20.04上实现开机自动运行Clash代理,你可以按照以下步骤进行操作：

1. **创建启动脚本**：
   - 首先,创建一个启动脚本,用于启动Clash代理.你可以在 `/etc/init.d/` 目录下创建一个新的脚本文件,比如 `clash-start.sh`.

2. **编辑启动脚本**：
   - 使用文本编辑器（比如 `nano` 或 `vim`）编辑这个脚本文件.在文件中添加如下内容：
     ```bash
     #!/bin/bash
     /path/to/your/clash-binary -d /path/to/your/config/directory
     ```
     请替换 `/path/to/your/clash-binary` 为你实际的 Clash 可执行文件路径,`/path/to/your/config/directory` 为你的配置文件目录.

3. **添加可执行权限**：
   - 添加可执行权限给你的启动脚本：
     ```bash
     sudo chmod +x /etc/init.d/clash-start.sh
     ```

4. **设置为开机自启动**：
   - 使用 `update-rc.d` 命令将脚本添加到启动项中：
     ```bash
     sudo update-rc.d clash-start.sh defaults
     ```

5. **测试**：
   - 现在你可以测试这个启动脚本是否能够正常运行.你可以重启你的计算机,然后检查 Clash 是否已经自动启动了.

这样,每次系统启动时,Clash 代理都会自动运行.
#### clash透明代理
TUN 模式
新版的 Clash Premium 内核支持 TUN 模式,且目前已支持 Linux 系统下的 auto-route 和 auto-detect-interface,无需手动设置转发表,可以方便快捷的实现 透明网关（旁路由） 的功能.

首先需要下载 Clash Premium 版本,替换上面的 clash 文件.接着需要设置 Linux 系统,开启转发功能.编辑文件 /etc/sysctl.conf,添加以下内容：

``` shell
net.ipv4.ip_forward=1

```
保存退出后,执行以下命令使修改生效：

``` shell
sudo sysctl -p
```

然后接着需要关闭系统的 DNS 服务,使用以下命令：

``` shell
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```
关于代理环境下 DNS 解析行为的深入探讨,可以参见浅谈在代理环境中的 DNS 解析行为以及我有特别的 DNS 配置和使用技巧.

接着需要设置 Clash 的配置文件,添加以下内容：

```json
dns:
  enable: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip
  nameserver:
    - 114.114.114.114
  fallback:
    - 8.8.8.8
tun:
  enable: true
  stack: system # or gvisor
  dns-hijack:
    - 8.8.8.8:53
    - tcp://8.8.8.8:53
    - any:53
    - tcp://any:53
  auto-route: true # auto set global route
  auto-detect-interface: true # conflict with interface-name
  ```

最后重启 Clash 服务即可,这样流量就会通过 TUN 接口转发,同时利用强大的分流规则,实现按需代理.也可以设置局域网内的网关地址和 DNS 服务器地址,实现透明网关.

##### 为什么我的clash没有dashboard
因为没有所以没有,我找到的 clash for linux 的版本是不带控制界面的,需要访问 这个 web 控制台 才能进行控制.后来为了方便访问,我就把他的包都给抓了下来,放在了本地的 nginx 上.

