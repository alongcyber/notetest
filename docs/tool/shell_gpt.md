
## shell-gpt使用说明

需求就是在命令行使用大模型,项目已经非常不错,而且也有好心人可以生成免费的api来使用chatgpt.所以用起来就非常爽(ubuntu这个b输入法真的是一炮大便啊,写下这句话的后面我就来装好的输入法,rime输入法+雾淞拼音)


``` shell
(base) along@along-Lenovo-Legion-Y9000K2020H: /home/along  
➜   sgpt --shell "查看我系统上是如何开机启动clash的"         
ls /etc/systemd/system/clash.service
sudo systemctl status clash.service
[E]xecute, [D]escribe, [A]bort: E
/etc/systemd/system/clash.service
[sudo] password for along: 
Sorry, try again.
[sudo] password for along: 
● clash.service - clash
     Loaded: loaded (/etc/systemd/system/clash.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-06-28 01:41:38 CST; 1h 26min ago
   Main PID: 1109 (clash)
      Tasks: 22 (limit: 76692)
     Memory: 36.8M
     CGroup: /system.slice/clash.service
             └─1109 /usr/bin/clash -f /home/along/desk/clash/config.yaml

Jun 28 03:07:42 along-Lenovo-Legion-Y9000K2020H clash[1109]: time="2024-06-28T03:07:42+08:00" level=info msg="[TCP] 127.0.0.1:45726 --> >
Jun 28 03:07:42 along-Lenovo-Legion-Y9000K2020H clash[1109]: time="2024-06-28T03:07:42+08:00" level=info msg="[TCP] 127.0.0.1:45742 --> >
Jun 28 03:07:50 along-Lenovo-Legion-Y9000K2020H clash[1109]: time="2024-06-28T03:07:50+08:00" level=info msg="[TCP] 127.0.0.1:34094 --> >
Jun 28 03:07:50 along-Lenovo-Legion-Y9000K2020H clash[1109]: time="2024-06-28T03:07:50+08:00" level=info msg="[TCP] 127.0.0.1:34080 --> >
Jun 28 03:07:53 along-Lenovo-Legion-Y9000K2020H clash[1109]: time="2024-06-28T03:07:53+08:00" level=info msg="[TCP] 127.0.0.1:34106 --> >
Jun 28 03:07:58 along-Lenovo-Legion-Y9000K2020H clash[1109]: time="2024-06-28T03:07:58+08:00" level=info msg="[TCP] 127.0.0.1:43650 --> >
Jun 28 03:07:58 along-Lenovo-Legion-Y9000K2020H clash[1109]: time="2024-06-28T03:07:58+08:00" level=info msg="[TCP] 127.0.0.1:43654 --> >
Jun 28 03:08:06 along-Lenovo-Legion-Y9000K2020H clash[1109]: time="2024-06-28T03:08:06+08:00" level=info msg="[TCP] 127.0.0.1:41268 --> >

```
## 今夜的奇幻旅行
首先我发现我的zsh_history出大问题,有一些不好的中文历史记录要删除
```
zsh_histroy的编码格式是一个特殊的格式,导致了zsh_history的文件内容出现了乱码,所以vim或者vscode打开时候就会出现中文乱码.
```
这个不是说常见的zsh的中文乱码而是历史记录的乱码,[解码含有Unicode的zsh历史记录](https://wszqkzqk.github.io/2024/03/31/zsh-history-unicode-decode/)

#### zsh_hist原因
zsh的历史记录并非直接使用UTF-8编码，而是使用了一种特殊的元格式编码。这种编码的格式如下：

Unicode字节前加上了元字节0x83开头
后续的内容需要与32进行异或运算才能得到原始值
因此，如果要解码zsh的历史记录，需要先找到0x83开头的字节，将其后面的内容转化为与32进行异或运算的值，才能得到结果。

#### zsh_history优化
.zsh_history 历史记录优化
.zsh_history的存储编码是一个非常特殊的编码,不是常见的utf,gbk什么的，所以里面中文用别的软件打开了就乱码了，想用python把.zsh_history的重复的都去掉，读的时候就报错了，zsh官方有人问特殊字符怎么办，倒是有一个c语言的版本，最后用:

bash
setopt HIST_IGNORE_ALL_DUPS
sort -t ";" -k 2 -u ~/.zsh_history | sort -o ~/.zsh_history
解决问题，有时候shell真的是神器

#### 我试出来的一个很唐的
在zsh的'.zshrc'的plugins中添加history就会把所有的历史记录中的乱码全部删除.这个是一个很唐的方法,但是确实有效,我也不知道为什么,但是确实有效.

## rime输入法
### 安装等一笔带过
[Ubuntu 22.04 Desktop配置雾凇拼音Rime-Ice](https://www.cnblogs.com/KLangHu/p/17699295.html)
### 需要注意的地方
Plum管理rime配置


Ctrl+` 会切换输入方案,这个是默认的快捷键,我把这个快捷键改没了,这个是我习惯的(因为他这几个热键我都要用),在rime输入法的配置文件'default.yaml(/home/along/.config/ibus/rime/default.yaml)'中修改


把ibus这颜色换成绿色,橙色太刺眼来.
### vim的tab还是设置为4格比较好

在 Python 编程中，通常推荐使用4个空格来进行缩进。这一建议来自于 Python 的官方风格指南 PEP 8（Python Enhancement Proposal 8），它被广泛接受和应用于大多数 Python 项目中。然而，2个空格的缩进在其他编程语言和一些团队中也很常见。以下是两种缩进方式的优缺点：

### 4个空格缩进
#### 优点：
1. **可读性高**：4个空格的缩进使代码块之间的层次结构更加明显，特别是在嵌套较深的情况下。
2. **广泛接受**：4个空格是 Python 社区和许多其他编程语言社区的标准，遵循这一标准可以使代码更易于共享和协作。
3. **工具支持**：大多数代码编辑器和IDE默认支持并配置为4个空格的缩进。

#### 缺点：
1. **占用空间多**：相比2个空格，4个空格会在每一层缩进时占用更多的水平空间，这在一些情况下可能导致代码行过长。

### 2个空格缩进
#### 优点：
1. **节省空间**：2个空格的缩进占用的水平空间更少，可以在有限的屏幕宽度内显示更多的代码内容。
2. **简洁**：对于简单的代码或嵌套层次较少的代码，2个空格的缩进显得更简洁。

#### 缺点：
1. **可读性较差**：在深层嵌套时，2个空格的缩进可能不如4个空格的缩进那么清晰，容易导致阅读和理解困难。
2. **不符合主流标准**：在Python 编程中，2个空格的缩进不符合 PEP 8 的建议，可能会导致团队合作中的不一致性。

### 总结
对于 Python 编程，通常推荐使用4个空格的缩进，因为它是 PEP 8 的建议并且被广泛接受和应用。如果你在一个团队中工作，遵循团队的代码风格指南是最重要的。如果没有特定的要求，4个空格是一个安全的选择。对于其他编程语言和特定项目，可以根据实际需求选择2个或4个空格的缩进。

### 配置编辑器
确保你的编辑器配置正确，以便自动使用4个空格进行缩进：

#### VS Code
```json
{
    "editor.tabSize": 4,
    "editor.insertSpaces": true
}
```

#### Vim
```vim
set tabstop=4
set shiftwidth=4
set expandtab
```



通过这些配置，可以确保你的代码缩进始终符合标准，提高代码的可读性和一致性。

## zsh vs fish
被zsh_history搞了一次后我有点想换到fish了(但是确实有点麻烦),我确实有点沉迷在工具的优化上了,当然了,输入法还是值得优化一下的.







































