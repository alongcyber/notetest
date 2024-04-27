# 建站的记录


## Another heading

Some more shit
### 一些沟是
哈哈哈把喇叭卡和单位,很喜欢我的键盘敲击的声音,有一种清新愉悦的快感,谁懂阿,给虚拟机新增了32GB内存,顿时流畅了许多,输入法不再有莫名的顿挫.
### 这些天
半角字母的美,谁懂,感觉很多事情,没有必要弄的不一样,懂我意思么.
### 关于这个礼拜
这个礼拜有些事情步入了正规,比如考研,比如coding,比如五级流水线CPU,比如CSAPP,比如我对一些事情的认识.
积极使用半角标点符号的好处(`ctrl`+`up`)=>
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

还是得手动修改mkdocs.yml中的目录,并不是自动检测新增文本文件来自动生成mkdocs.yml

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

请注意，强制推送是一种破坏性操作，它会永久地覆盖远端的`main`分支。在你执行这个操作之前，你应该确保你不需要`main`分支上的任何更改，或者你已经备份了这些更改。
## 一些关于markdown的记录
作者：奚水溪流西
链接：https://zhuanlan.zhihu.com/p/681840313
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


1. 什么是“民族团结主义”，什么是“团结派”
所谓“民族团结主义”是我国特殊国情发展出来的逆向民族主义，“团结主义”的本质是在民族差异对待政策下以民族文化为对立基础的民族分裂主义。它过分强调“民族的虚无性”，同时致力于“去汉族化”，否定广大汉族人民应有的汉族历史叙事。它不仅抹杀了各少数民族的文化源头，各少数民族有自身文化的人文初祖，可“团结主义”一味的扭曲少数民族文化，力图将所有的少数民族的人文初祖强行划定为炎帝黄帝，它更扭曲了汉文化的文化原教旨阶段的发展历史。“团结派”们在口号上和言论中拥护民族团结平等，实际上模糊了实际存在的民族政策的差异对待，和社会主义市场经济发展条件下的客观民族关系的内在矛盾，他们往往假马克思主义者，以统战政策为政治正确之背书，借“民族团结”之名，不提先验矛盾而只提结果终论。就如同整天把“共产主义一定要实现”的不成熟的，尚且幼稚的网络左派兴趣少年们一样，不同的是，“团结派”们并不幼稚，他们的底气是“统战价值”，他们的本质是少数民族中居心不良的分裂分子。

2. 什么是“团结史观”
“团结史观”是在世界观上是错误的唯心史观，团结派往往指鹿为马，指“团结史观”为“阶级史观”，披着阶级史观的皮，混淆历史概念和掩盖历史发展过程中的事件和事实以及基本结果，阶级史观要与“团结史观”划清界限。团结史观忽略，模糊，甚至美化不同民族历史发展之间的矛盾和具体的历史民族遗留问题，它在哲学观上否定了历史唯物主义，否定了马克思主义哲学。它把“民族共同体”的概念套用到了不符合实际情况的古代的各个历史阶段，只一昧的强调血统的融合，文化的包容，而忽视了不同民族之间的不同历史经济发展基础，不同文化习俗诞生社会及自然地理条件，只提出共性，不突出个性，只谈论整体，不讨论部分。只说明全部，不搞清区别。在现代科学，分子生物学的发展和开放民间的唾液检测的潮流下，早就应该被扫入历史垃圾桶的“汉人杂血论”，“汉人胡化论”已经被击破，实际情况证明了，汉民族血统在世界上恰恰是极为纯正的，反倒是不少原本是少数民族的人民群众，发现自己汉人血统的比重占主体。团结史观反对实事求是看待问题，是落后且错误的愚蠢史观，马克思主义者要警惕这种史观的虚假面貌。

3. “团结主义”，“团结派”的方法论和方法
团结派们往往运用马克思主义哲学及马克思主义政治经济学中的专业术语来哄骗，裹挟无知的网络民众和对马克思主义了解不深的，只存在于言论模仿的网络马克思主义身份兴趣爱好者。他们只强调我国曾经“反对大汉族主义”，却只口不提我国也反对“地方民族主义”他们重复念叨“世上有阶级而无民族”，而这不过是斯大林主义在苏共建设中所发展出错误的民族观点的论调之一罢了。马克思和列宁从未直接提出“世界上有阶级而无民族”的观点。阶级的存在基于生产资料私有制而存在。民族的存在基于以一定阶段的民族血统为基础，和一定时期长期共存交往关系中的人类共同文化。马克思主义者认为，阶级和民族在物质基础和意识形态两个方面是同时消灭的，而这种消灭需要在共产主义建立的前提下。消灭民族，不代表消灭民族肉体，也不代表民族文化，和否定民族发展的历史。而是消灭民族主义对文化交流和物质交往的思想隔阂和制度阻断。世界上有阶级，也有民族。民族主义是指一国的民族资产阶级为扩张市场经济利益对本国本民族无产阶级的压迫和动员，包括但不限于贸易保护主义，对外战争征兵等。那些鼓吹“自由主义”与“民族主义”联合起来的，都是以压榨本民族利益为目的的资产阶级论调。同样的，那些鼓吹“世上有阶级而无民族”与“团结统战”联合起来的，是逆向民族主义的利益分子，假借马克思主义名义，动员绝大多数人团结极少数人，在民族问题上观点失衡，本质是分裂主义的论调。

4. 马克思主义者的方法论和方法
首先，拥护民族平等就要反对民族差异对待政策。统战只能是暂时的需要，而“暂时”是有界限、有时间期限的。例如少数民族加分政策，是实打实的冗政冗策，它忽略了少数民族地区真正需要的教育资源调配，教育事业发展，经济水平相对落后等多方面问题，量化加分是没有科学依据的，有人说：“加分政策加的分如今很少了，大多数少民加分少则不过一两分”。这就更该取消它了！有不如无的东西应该立马就无。一拖再拖会被统战大旗绊住自己的脚。在这个基础上，反对一切少数民族优待政策，我们鼓励将真正少数民族需要的经济资源，科技发明，基础设施，教育理念予以投入和扩大进而发展，而非拥护它们.

5. 评论区

把满清当中国的人，至少马克思主义就没学好，老马都说了当时中国被这些鞑虏征服了。

“满清王朝实行这种闭关锁国政策的更主要原因是它害怕外国人会支持很多的中国人在十七世纪的大约前半个世纪里即在中国被鞑靼人征服以后所怀抱的不满情绪。由于这种原因，外国人才被禁止同中国人有任何来往。”

这些往外输出“秩序”，企图扶持摇摇欲坠的满清王朝的列强，恐怕是忘记了：仇视外国人，把他们逐出国境，这在过去仅仅是出于中国地理上、人种上的原因，只是在满洲鞑靼人征服了这个国家以后才形成一种政治制度。

节选自文章：《中国革命和欧洲革命》。
作者：卡尔·马克思。1853年5月20日，伦敦发表。