
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


把ibus这颜色换成绿色,橙色太刺眼来.OK搞好了







































