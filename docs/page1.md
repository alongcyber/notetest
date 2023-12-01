# 建站的记录
## Another heading

Some more shit
### 一些沟是
哈哈哈把喇叭卡和单位,很喜欢我的键盘敲击的声音,有一种清新愉悦的快感,谁懂阿,给虚拟机新增了32GB内存，顿时流畅了许多,输入法不再有莫名的顿挫.
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
|-------|---------|
|get    |shit     |
|`get`  |:material-check: Fetch resource|
|`PUT`  |:material-check-all: Update res|

->

{{ read_csv('./data.csv') }}

->

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