整个实验逐步引导完成一个计算机系统的构建,包括底层的 NEMU 模拟器,运行时环境 AbstractMachine（AM）,在其上的简易操作系统 NanOS-lite,以及操作系统上的应用程序库 Navy-apps.

一共分为 5 个部分,PA0 配置环境,PA1 完善 NEMU 的调试器功能,PA2 模拟 NEMU 指令运行以及补充 AM,PA3 完善 NEMU 的中断 / 异常处理、实现操作系统的系统调用以及简易文件系统功能,PA4 在操作系统中实现多道程序的运行、虚拟内存管理以及外部中断处理.

细分每个实验的内容以及涉及到的源码部分,我整理了如下一个表格供参考：

PS：其中 AM 由五个部分组成,TRM（图灵机模拟）简单而且已经实现好了,IOE 为输入输出扩展,CTE 为上下文管理扩展,VME 为虚拟内存管理扩展,还有一个 MPE（多处理器扩展）在 PA 中不使用.klib 为简单的运行时库,提供给 AM 和操作系统使用.

总而言之有了计算机系统课以及 RISCV 手册阅读的基础,大部分实验还是比较轻松的,没有什么难以理解的地方.较为痛苦的就是 debug 时一些奇怪的问题以及最后 PA4 中之前没有接触过的虚存管理的内容了.后面记录一些还能回忆起来的问题（其实是做的过程中懒得记录）.

## macro magic
炫酷的跨平台能力跨架构能力却是靠着看上去笨拙其实炫酷的宏技术实现的,所以我们不妨叫他魔法宏.这里有一些宏技术的例子,可以帮助你理解宏的强大之处.
```c
#define ARRLEN(arr) (int)(sizeof(arr) / sizeof(arr[0]))
```

See [stackoverflow的帖子关于macro](https://stackoverflow.com/questions/26099745/test-if-preprocessor-symbol-is-defined-inside-macro)

Compare the stringified macro (name) to the stringified (expanded) value of the macro:
将字符串化宏（名称）与宏的字符串化（扩展）值进行比较：
```c

#include <iostream>
#include <cstring>

#define TRACE_STRINGIFY(item) "" #item
#define TRACE(macro, message)                          \
    do {                                               \
        if (strcmp("" #macro, TRACE_STRINGIFY(macro))) \
            std::cout << message << "\n";              \
    } while (0)
```
这段代码是C++中的宏定义,用于在编译时期生成代码.让我们逐行解析：

1. `#include <iostream>` 和 `#include <cstring>` 是包含标准库的头文件,分别提供了输入/输出功能和字符串处理功能.

2. `#define TRACE_STRINGIFY(item) "" #item` 是一个宏定义,用于将输入的参数转换为字符串.`#`操作符在宏定义中被称为"字符串化"操作符,它会将后面的参数转换为字符串.

那么为什么#item前面要加引号呢“”？

在C语言的宏定义中,`#`操作符被称为"字符串化"操作符,它会将后面的宏参数转换为字符串.但是,如果直接使用`#item`,并且`item`是一个宏,那么预处理器会先将`item`转换为它所代表的值,然后再将结果转换为字符串.

例如,假设我们有以下的宏定义：

```c
#define VALUE 10
#define TO_STRING(item) #item
```

如果我们使用`TO_STRING(VALUE)`,预处理器会先将`VALUE`替换为`10`,然后将`10`转换为字符串,得到`"10"`.

但是,如果我们想得到字符串`"VALUE"`而不是`"10"`,我们就需要阻止预处理器对`VALUE`的宏扩展.这就是为什么我们在`#item`前面加上`""`的原因.`"" #item`会先将`item`转换为字符串,然后与前面的空字符串连接,得到的结果就是我们想要的`"VALUE"`.

所以,`#define TRACE_STRINGIFY(item) "" #item`这个宏定义的作用是将输入的参数转换为字符串,同时阻止预处理器对参数进行宏扩展.

3. `#define TRACE(macro, message)` 定义了一个名为`TRACE`的宏,它接受两个参数：`macro`和`message`.

4. `do { ... } while (0)` 是一个常见的宏技巧,用于确保宏的使用方式和函数调用一样,即使在需要使用分号结束的地方也不会出错.`do { ... } while (0)` 是一个常见的宏定义技巧,主要是为了解决宏扩展中可能出现的问题.

在C/C++中,宏是在预处理阶段进行文本替换的,它并不像函数那样有自己的作用域.如果一个宏定义包含多条语句,而在使用这个宏的地方没有正确地用括号包围它,就可能导致预期之外的行为.

例如,考虑以下宏定义和使用：

```c++
#define MACRO(x) foo(x); bar(x)

if (condition)
    MACRO(y);
else
    baz();
```

这段代码会被预处理器扩展为：

```c++
if (condition)
    foo(y); bar(y);
else
    baz();
```

这并不是我们想要的结果,因为`bar(y);`总是会被执行,而不管`condition`的值是什么.

如果我们使用`do { ... } while (0)`,就可以避免这个问题：

```c++
#define MACRO(x) do { foo(x); bar(x); } while (0)

if (condition)
    MACRO(y);
else
    baz();
```

这段代码会被预处理器扩展为：

```c++
if (condition)
    do { foo(y); bar(y); } while (0);
else
    baz();
```

这样就可以得到我们期望的行为：只有当`condition`为真时,`foo(y)`和`bar(y)`才会被执行.


5. `if (strcmp("" #macro, TRACE_STRINGIFY(macro)))` 这行代码比较复杂.首先,`"" #macro`将`macro`参数字符串化.然后,`TRACE_STRINGIFY(macro)`也将`macro`参数字符串化.`strcmp`函数比较这两个字符串,如果它们相等,返回0,否则返回非0值.因此,这个`if`语句的条件是"如果`macro`参数字符串化后的结果和自身不相等".

6. `std::cout << message << "\n";` 如果上述`if`条件为真,那么就输出`message`参数,并在其后添加一个换行符.

这个宏的目的是检查`macro`参数是否已经被定义为一个宏.如果`macro`已经被定义为一个宏,那么`TRACE_STRINGIFY(macro)`将返回该宏的值,而不是`macro`本身,因此`strcmp`的结果将不为0,从而触发消息的输出.
The "" # macro expands to the macro name as a string, whereas TRACE_STRINGIFY(macro) first expands the macro, then stringifies the result. If the two differ, macro has to be a preprocessor macro.
"" # macro 将宏名扩展为字符串,而 TRACE_STRINGIFY(macro) 首先扩展宏,然后将结果字符串化.如果两者不同,则 macro 必须是预处理器宏.

This approach does fail for macros that are defined to themselves, i.e. #define FOO FOO. Such macros are not detected as preprocessor macros.
这种方法对于定义为自身的宏（即 #define FOO FOO ）确实失败.这样的宏不会被检测为预处理器宏.

Most compilers should be able to completely optimize away the comparison of two string literals. GNU GCC (g++) 4.8.2 definitely does even with -O0 (as does gcc for C -- the same approach obviously works in C, too).
大多数编译器应该能够完全优化两个字符串的比较.GNU GCC（g++）4.8.2甚至对 -O0 也是如此（gcc对C也是如此--同样的方法显然也适用于C）.

This approach does work for function-like macros, but only if you retain the parentheses (and proper number of commas, if the macro takes multiple parameters) and it is not defined to itself (e.g. #define BAR(x) BAR(x)).
这种方法确实适用于类似函数的宏,但前提是您保留括号（以及适当数量的逗号,如果宏接受多个参数）并且它没有定义为自身（例如 #define BAR(x) BAR(x) ）.

For example: 举例来说：

```c
#define TEST1 TEST1

#define TEST3
#define TEST4 0
#define TEST5 1
#define TEST6 "string"
#define TEST7 ""
#define TEST8 NULL
#define TEST9 TEST3
#define TEST10 TEST2
#define TEST11(x)

#define TEST13(x,y,z) (x, y, z)


int main(void)
{
    TRACE(TEST1, "TEST1 is defined");
    TRACE(TEST2, "TEST2 is defined");
    TRACE(TEST3, "TEST3 is defined");
    TRACE(TEST4, "TEST4 is defined");
    TRACE(TEST5, "TEST5 is defined");
    TRACE(TEST6, "TEST6 is defined");
    TRACE(TEST7, "TEST7 is defined");
    TRACE(TEST8, "TEST8 is defined");
    TRACE(TEST9, "TEST9 is defined");
    TRACE(TEST10, "TEST10 is defined");
    TRACE(TEST11, "TEST11 is defined");
    TRACE(TEST12, "TEST12 is defined");
    TRACE(TEST13, "TEST13 is defined");
    TRACE(TEST14, "TEST14 is defined");

    TRACE(TEST1(), "TEST1() is defined");
    TRACE(TEST2(), "TEST2() is defined");
    TRACE(TEST3(), "TEST3() is defined");
    TRACE(TEST4(), "TEST4() is defined");
    TRACE(TEST5(), "TEST5() is defined");
    TRACE(TEST6(), "TEST6() is defined");
    TRACE(TEST7(), "TEST7() is defined");
    TRACE(TEST8(), "TEST8() is defined");
    TRACE(TEST9(), "TEST9() is defined");
    TRACE(TEST10(), "TEST10() is defined");
    TRACE(TEST11(), "TEST11() is defined");
    TRACE(TEST12(), "TEST12() is defined");
    TRACE(TEST13(,,), "TEST13(,,) is defined");
    TRACE(TEST14(,,), "TEST14(,,) is defined");

    return 0;
}
which outputs 其输出

TEST3 is defined
TEST4 is defined
TEST5 is defined
TEST6 is defined
TEST7 is defined
TEST8 is defined
TEST9 is defined
TEST10 is defined
TEST3() is defined
TEST4() is defined
TEST5() is defined
TEST6() is defined
TEST7() is defined
TEST8() is defined
TEST9() is defined
TEST10() is defined
TEST11() is defined
TEST13(,,) is defined
```
In other words, the TEST1 symbol is not recognized as defined (because it is defined to itself), nor are TEST11 or TEST13 without parentheses. These are the limitations of this approach.
换句话说, TEST1 符号不会被识别为已定义（因为它是对自身定义的）,没有括号的 TEST11 或 TEST13 也不会被识别为已定义.这些是这种方法的局限性.

Using the parenthesized form works for all parameterless macros (except TEST1, ie. those defined to themselves), and all single-parameter macros. If a macro expects multiple parameters, you need to use the correct number of commas, as otherwise (say, if you tried TRACE(TEST13(), "...")) you get a compile-time error: "macro TEST13 requires 3 arguments, only 1 given" or similar.
使用括号形式适用于所有无参数宏（除了 TEST1 ,即.它们是自己定义的）,以及所有单参数宏.如果一个宏需要多个参数,你需要使用正确数量的逗号,否则（比如说,如果你尝试 TRACE(TEST13(), "...") ）你会得到一个编译时错误：“宏 TEST13 需要3个参数,只有1个给定”或类似的.

### printf的含金量
`printf`函数是C语言中用于格式化输出的函数,它有很多特殊的用法.以下是一些例子：

#### ysyx的例子

```c
void isa_reg_display() {
  int reg_num = ARRLEN(regs);
  for (int i = 0; i < reg_num; i++) {
    printf("%-4s: 0x%08x\n", regs[i], cpu.gpr[i]);
  }
}

```
这段代码是用于在C语言中打印CPU寄存器的值.它使用了`printf`函数,这是一个标准的C库函数,用于格式化输出到标准输出设备,通常是屏幕.

`printf`函数的第一个参数是一个格式字符串,它定义了输出的格式.在这个例子中,格式字符串是`"%-4s: 0x%08x\n"`.

`%-4s`表示一个字符串,`-`表示左对齐,`4`表示这个字符串的最小宽度是4.如果`regs[i]`的长度小于4,那么输出的字符串将被空格填充到4个字符.

`: 0x%08x`表示一个以十六进制形式输出的整数.`0x`是一个前缀,表示这个数是十六进制的.`%08x`表示这个整数至少有8位,如果不足8位,那么输出的字符串将被0填充到8位.

`\n`是一个换行符,表示在输出这一行后换行.

`printf`函数的后续参数是要插入到格式字符串中的值.在这个例子中,[`regs[i]`]是要插入到`%-4s`的位置的字符串,`cpu.gpr[i]`是要插入到`%08x`的位置的整数.

总的来说,这段代码将会输出类似于`eax : 0x0000001f`这样的字符串,其中`eax`是寄存器的名字,`0x0000001f`是寄存器的值.

1. **字段宽度和精度**：你可以在格式字符串中指定字段的宽度和精度.例如,`%5d`表示一个至少有5个字符宽的整数,`%.2f`表示一个小数点后有2位的浮点数.

```c
printf("%5d\n", 123);  // 输出："  123"
printf("%.2f\n", 3.14159);  // 输出："3.14"
```

2. **左对齐和右对齐**：默认情况下,`printf`函数会右对齐输出.你可以使用`-`标志来改变这个行为.例如,`%-5d`表示一个左对齐的至少有5个字符宽的整数.

```c
printf("%-5d\n", 123);  // 输出："123  "
```

3. **填充字符**：默认情况下,`printf`函数会使用空格来填充字段.你可以使用`0`标志来改变这个行为.例如,`%05d`表示一个用0填充的至少有5个字符宽的整数.

```c
printf("%05d\n", 123);  // 输出："00123"
```

4. **十六进制和八进制**：你可以使用`%x`和`%o`来输出十六进制和八进制的整数.你还可以使用`#`标志来添加前缀.

```c
printf("%x\n", 255);  // 输出："ff"
printf("%#x\n", 255);  // 输出："0xff"
printf("%o\n", 255);  // 输出："377"
printf("%#o\n", 255);  // 输出："0377"
```

5. **指针**：你可以使用`%p`来输出指针的值.

```c
int x = 0;
printf("%p\n", &x);  // 输出类似于："0x7fffc72c4b3c"
```

6. **字符串**：你可以使用`%s`来输出字符串.

```chttps://alongcyber.github.io/notetest/PA/PA/ 输出："Hello, world!"
```

以上只是`printf`函数的一部分特殊用法.`printf`函数还有很多其他的特性和选项,你可以查阅C语言的文档来了解更多.

### 优雅的退出

```c
int is_exit_status_bad() {
  int good = (nemu_state.state == NEMU_END && nemu_state.halt_ret == 0) ||
    (nemu_state.state == NEMU_QUIT);
  return !good;
}
```
直接在cmd_q中修改成退出前把nemu_state.state改成NEMU_QUIT

#### 代码中值得注意的地方
最后我们聊聊代码中一些值得注意的地方.

+ 三个对调试有用的宏(在nemu/include/debug.h中定义)
    + `Log()`是`printf()`的升级版, 专门用来输出调试信息, 同时还会输出使用`Log()`所在的源文件, 行号和函数. 当输出的调试信息过多的时候, 可以很方便地定位到代码中的相关位置
    + `Assert()`是`assert()`的升级版, 当测试条件为假时, 在`assertion fail`之前可以输出一些信息
    + `panic()`用于输出信息并结束程序, 相当于无条件的`assertion fail`

代码中已经给出了使用这三个宏的例子, 如果你不知道如何使用它们, RTFSC.

+ 内存通过在nemu/src/memory/paddr.c中定义的大数组pmem来模拟. 在客户程序运行的过程中, 总是使用vaddr_read()和vaddr_write() (在nemu/src/memory/vaddr.c中定义)来访问模拟的内存. vaddr, paddr分别代表虚拟地址和物理地址. 这些概念在将来才会用到, 目前不必深究, 但从现在开始保持接口的一致性可以在将来避免一些不必要的麻烦.

### 就是这么简单
事实上, TRM(Technical Reference Model)的实现已经都蕴含在上述的介绍中了.

+ 存储器是个在`nemu/src/memory/paddr.c`中定义的大数组
+ PC和通用寄存器都在`nemu/src/isa/$ISA/include/isa-def.h`中的结构体中定义
+ 加法器在... 嗯, 这部分框架代码有点复杂, 不过它并不影响我们对TRM的理解, 我们还是在PA2里面再介绍它吧
+ TRM的工作方式通过`cpu_exec()`和`exec_once()`体现
在NEMU中, 我们只需要一些很基础的C语言知识就可以理解最简单的计算机的工作方式, 真应该感谢先驱啊.

### 解析命令

为了让简易调试器易于使用, NEMU通过readline库与用户交互, 使用readline()函数从键盘上读入命令. 与gets()相比, readline()提供了"行编辑"的功能, 最常用的功能就是通过上, 下方向键翻阅历史记录. 事实上, shell程序就是通过readline()读入命令的. 关于readline()的功能和返回值等信息, 请查阅
```shell
man strtok
```
另外, cmd_help()函数中也给出了使用strtok()的例子. 事实上, 字符串处理函数有很多, 键入以下内容:

``` shell
man 3 str<TAB><TAB>
```
其中<TAB>代表键盘上的TAB键. 你会看到很多以str开头的函数, 其中有你应该很熟悉的strlen(), strcpy()等函数. 你最好都先看看这些字符串处理函数的manual page, 了解一下它们的功能, 因为你很可能会用到其中的某些函数来帮助你解析命令. 当然你也可以编写你自己的字符串处理函数来解析命令.

另外一个值得推荐的字符串处理函数是sscanf(), 它的功能和scanf()很类似, 不同的是sscanf()可以从字符串中读入格式化的内容, 使用它有时候可以很方便地实现字符串的解析. 如果你从来没有使用过它们, RTFM, 或者STFW.

GNU的`readline`库提供了一种方便的方式来处理用户的输入,包括行编辑,历史记录和自动补全.以下是一些基本的使用方法：

1. **初始化readline**：在使用readline之前,你需要先初始化它.这通常在程序开始时完成：

    ```c
    #include <readline/readline.h>
    #include <readline/history.h>
    ```

2. **读取输入**：你可以使用`readline`函数来读取用户的输入.这个函数接受一个提示字符串作为参数,并返回用户输入的字符串.例如：

    ```c
    char* input = readline("Enter your command: ");
    ```

    用户可以使用readline的行编辑功能来修改他们的输入.当他们按下Enter键时,`readline`函数将返回他们输入的字符串.

3. **处理历史记录**：你可以使用`add_history`函数来添加一个命令到历史记录中：

    ```c
    add_history(input);
    ```

    用户可以使用上下箭头键来浏览历史记录.

4. **自动补全**：你可以使用`rl_completion_entry_function`和`rl_attempted_completion_function`来定制自动补全的行为.例如,你可以设置一个函数来提供可能的补全,或者改变补全的规则.

5. **清理**：当你完成了readline的使用,你应该释放它返回的字符串：

    ```c
    free(input);
    ```

以上就是GNU readline的基本用法.请注意,readline库的功能非常强大,这里只是介绍了一些基本的用法.如果你想了解更多,你可以查阅readline的官方文档.

#### 举例说明readline
以下是一个使用GNU readline库的简单C程序示例.这个程序会提示用户输入命令,然后将命令添加到历史记录中,并打印出来.用户可以使用readline的行编辑和历史功能.

```c
#include <stdio.h>
#include <stdlib.h>
#include <readline/readline.h>
#include <readline/history.h>

int main() {
    char* input;

    while(1) {
        input = readline("Enter command: ");

        if (!input) {
            // EOF, exit loop
            break;
        }

        printf("You entered: %s\n", input);

        if (*input) {
            add_history(input);
        }

        free(input);
    }

    return 0;
}
```

在这个程序中,我们使用`readline`函数读取用户的输入.我们传递一个提示字符串"Enter command: "给`readline`函数,然后它会返回用户输入的字符串.(而且readline会打印提示字符串)

我们检查返回的字符串是否为NULL,如果是NULL,那么我们就退出循环.否则,我们就打印出用户输入的字符串,并使用`add_history`函数将它添加到历史记录中.

最后,我们使用`free`函数释放`readline`函数返回的字符串.

要编译这个程序,你需要链接readline库.在gcc中,你可以使用`-lreadline`选项来做到这一点：

```bash
gcc -o my_program my_program.c -lreadline
```
nemu中的example

```c
/* We use the `readline' library to provide more flexibility to read from stdin. */
static char* rl_gets() {
  static char *line_read = NULL;

  if (line_read) {
    free(line_read);
    line_read = NULL;
  }

  line_read = readline("(nemu) ");

  if (line_read && *line_read) {
    add_history(line_read);
  }

  return line_read;
}

```
#### 关于strtok
`strtok`是一个C语言库函数,用于在字符串中查找并分割出一系列的标记（tokens）.它的原型如下：

```c
char *strtok(char *str, const char *delim);
```

这个函数接受两个参数：一个是要分割的字符串`str`,另一个是包含分隔符的字符串`delim`.函数会在`str`中查找`delim`中的任何字符,并将其作为标记的结束.然后,它会在`str`中插入一个null字符来结束这个标记,并返回一个指向这个标记的指针.

在连续的调用中,你可以传递NULL作为第一个参数,`strtok`会继续在上次的字符串上进行操作,查找下一个标记.

例如,以下的代码会分割一个由空格分隔的字符串：

```c
char str[] = "Hello, how are you?";
char *token = strtok(str, " ");
while (token != NULL) {
    printf("%s\n", token);
    token = strtok(NULL, " ");
}
```

这段代码会输出：

```
Hello,
how
are
you?
```

请注意,`strtok`函数会修改它的输入字符串,所以如果你需要保留原始字符串,你应该在调用`strtok`之前创建一个副本.另外,`strtok`函数不是线程安全的,如果你在多线程环境中需要分割字符串,你应该使用`strtok_r`函数．

#### vscode不能正确的提交我的仓库

这让我很困惑,出现该报错Git: gpg failed to sign the data,但是我仅仅配置过ssh-key,而且我在vscode输入法会把,半角逗号自动改为全角真的恼火.明白了现在是因为vscode有个哈批设置叫做提交也得commit signed真的是哈批下的一比,[how-and-why-to-sign-git-commits](https://withblue.ink/2020/05/17/how-and-why-to-sign-git-commits.html),wcnmd太抽象了,这byd也能倒逼的,用vscode功能倒逼用户用这个commit signed,byd每次提交都得验证密钥.有点像byd每一下都得验证自愿了

#### 单步执行
从`cmd_c`就可以看出来`cpu_exec()`的含金量
=== "cmd_s"
    ``` c
    static int cmd_s(char*args){
    char *arg = strtok(NULL, " ");
    if(arg == NULL){
        cpu_exec(1);
    }
    else{
        int n = atoi(arg);
        if(n <= 0){
        printf("Invalid argument\n");
        return 0;
        }
        cpu_exec(n);
    }
    return 0;
    }

    ```
=== "cpu_exec()"
    ``` c
    /* Simulate how the CPU works. */
    void cpu_exec(uint64_t n) {
    g_print_step = (n < MAX_INST_TO_PRINT);
    switch (nemu_state.state) {
        case NEMU_END: case NEMU_ABORT:
        printf("Program execution has ended. To restart the program, exit NEMU and run again.\n");
        return;
        default: nemu_state.state = NEMU_RUNNING;
    }

    uint64_t timer_start = get_time();

    execute(n);

    uint64_t timer_end = get_time();
    g_timer += timer_end - timer_start;

    switch (nemu_state.state) {
        case NEMU_RUNNING: nemu_state.state = NEMU_STOP; break;

        case NEMU_END: case NEMU_ABORT:
        Log("nemu: %s at pc = " FMT_WORD,
            (nemu_state.state == NEMU_ABORT ? ANSI_FMT("ABORT", ANSI_FG_RED) :
            (nemu_state.halt_ret == 0 ? ANSI_FMT("HIT GOOD TRAP", ANSI_FG_GREEN) :
                ANSI_FMT("HIT BAD TRAP", ANSI_FG_RED))),
            nemu_state.halt_pc);
        // fall through
        case NEMU_QUIT: statistic();
    }
    }

    ```
#### 关于换行
在C语言中,你可以使用反斜杠（`\`）在源代码中将一行字符串字面量分成两行.这被称为行连接.例如：

```c
printf("这是一行非常非常非常非常非常非常非常非常非常非常\
非常非常非常非常非常非常非常非常非常非常长的文本.\n");
```

在这个例子中,反斜杠告诉C编译器这行代码还没有结束,下一行是这行的继续.这样,你可以在源代码中将一行非常长的字符串字面量分成多行,而在程序运行时,这些行会被连接成一个单一的字符串.

请注意,**反斜杠后面不能有任何字符,包括空格或者注释.反斜杠后面的第一个字符必须是换行符.**

### isa_reg_display(void)
这段代码是一个C语言头文件,定义了一些与RISC-V处理器的寄存器相关的函数和宏.RISC-V是一个开源的指令集架构,被广泛用于教学和研究.


然后,代码定义了一个预处理器宏`__RISCV_REG_H__`,用于防止这个头文件被多次包含.如果这个宏已经被定义,那么预处理器会跳过这个文件的剩余部分.否则,它会定义这个宏,并继续处理文件.

接下来,代码包含了一个名为`common.h`的头文件.这个头文件可能包含了一些常用的类型定义和函数声明.

然后,代码定义了一个名为`check_reg_idx`的内联函数,用于检查寄存器索引是否有效.这个函数接受一个整数参数`idx`,并检查它是否在0和`MUXDEF(CONFIG_RVE, 16, 32)`之间.如果`CONFIG_RT_CHECK`被定义,那么这个检查会被执行,否则它会被忽略.函数返回传入的索引.

接着,代码定义了一个名为`gpr`的宏,用于访问CPU的通用寄存器.这个宏接受一个索引参数,通过`check_reg_idx`函数检查这个索引是否有效,然后返回对应的寄存器.

最后,代码定义了一个名为`reg_name`的内联函数,用于获取寄存器的名称.这个函数接受一个整数参数`idx`,通过`check_reg_idx`函数检查这个索引是否有效,然后返回一个指向寄存器名称的指针.

文件的末尾是预处理器宏`__RISCV_REG_H__`的结束部分,标记这个头文件的结束.

#### 关于check_reg_idx(int idx)
这段代码定义了一个名为`check_reg_idx`的内联函数,这个函数用于检查寄存器索引是否有效.

函数接受一个整数参数`idx`,这个参数代表要检查的寄存器索引.

在函数体中,首先使用了一个名为`IFDEF`的宏.这个宏的行为取决于`CONFIG_RT_CHECK`是否被定义.如果`CONFIG_RT_CHECK`被定义,那么`assert`语句会被执行.这个`assert`语句会检查`idx`是否在0和`MUXDEF(CONFIG_RVE, 16, 32)`之间.如果`idx`不在这个范围内,那么`assert`会失败,程序会打印一个错误消息并终止.如果`CONFIG_RT_CHECK`没有被定义,那么`assert`语句会被忽略.

`MUXDEF`是另一个宏,它根据`CONFIG_RVE`的值选择两个选项之一.如果`CONFIG_RVE`被定义,那么`MUXDEF`返回16,否则返回32.这意味着如果RISC-V处理器启用了RV32E扩展（这是一个只有16个整数寄存器的扩展）,那么寄存器索引的上限是16,否则上限是32.

最后,函数返回传入的索引.这样,你可以在一个表达式中使用这个函数来检查寄存器索引,并在同一时间获取这个索引.例如,`gpr[check_reg_idx(idx)]`会检查`idx`是否有效,如果有效,就返回对应的寄存器.

#### 关于gpr(int idx)
```c
typedef struct {
  word_t gpr[MUXDEF(CONFIG_RVE, 16, 32)];
  vaddr_t pc;
} MUXDEF(CONFIG_RV64, riscv64_CPU_state, riscv32_CPU_state);
```
这段代码定义了一个名为`gpr`的宏,用于访问CPU的通用寄存器.

想要彻底了解底层,非得搞明白这些宏技巧
##### concat_temp(x,y)
```c
#define concat_temp(x, y) x ## y

```
这段代码定义了一个名为`concat_temp`的预处理器宏,这个宏用于连接两个预处理器标记.

在C语言中,预处理器宏可以接受参数,并在宏展开时替换这些参数.在这个宏的定义中,`x`和`y`是参数,`##`是连接运算符.

连接运算符`##`会将它左边和右边的标记连接成一个新的标记.例如,如果你写`concat_temp(a, b)`,那么预处理器会将这个宏展开为`ab`.

这个宏可以用于生成新的标识符,例如变量名或函数名.例如,如果你有一些变量`var1`、`var2`、`var3`等,你可以使用这个宏和一个循环来访问这些变量,而不需要为每个变量写单独的代码.

请注意,预处理器宏在编译时进行展开,所以它们不能用于动态生成标识符.所有的标识符都必须在编译时确定.
##### #define MUXDEF(macro, X, Y)  MUX_MACRO_PROPERTY(__P_DEF_, macro, X, Y)

`MUXDEF`的含金量就是根据macro的定义与否来选择X或Y

```c
#define CHOOSE2nd(a, b, ...) b
#define MUX_WITH_COMMA(contain_comma, a, b) CHOOSE2nd(contain_comma a, b)
#define MUX_MACRO_PROPERTY(p, macro, a, b) MUX_WITH_COMMA(concat(p, macro), a, b)

```
这段代码定义了一个名为`CHOOSE2nd`的预处理器宏,这个宏接受至少两个参数,并返回第二个参数.

在C语言中,预处理器宏可以接受参数,并在宏展开时替换这些参数.在这个宏的定义中,`a`、`b`和`__VA_ARGS__`是参数.

`a`和`b`分别代表第一个和第二个参数,`__VA_ARGS__`是一个特殊的标记,代表所有其他的参数.这个标记只能在接受可变数量参数的宏中使用.

在这个宏的定义中,只有`b`被使用,所以这个宏会忽略第一个参数和所有其他的参数,只返回第二个参数.

例如,如果你写`CHOOSE2nd(1, 2, 3, 4)`,那么预处理器会将这个宏展开为`2`.

这段代码定义了三个预处理器宏,它们一起实现了一个复杂的宏选择机制.

首先,`CHOOSE2nd(a, b, ...)`宏接受至少两个参数,并返回第二个参数.这个宏在后面的宏中被用作一个辅助工具,用于选择两个选项之一.

然后,`MUX_WITH_COMMA(contain_comma, a, b)`宏使用`CHOOSE2nd`宏来根据`contain_comma`参数是否包含逗号来选择`a`或`b`.如果`contain_comma`包含逗号,那么宏会展开为`a`,否则宏会展开为`b`.这个宏利用了C预处理器的一个特性,即在宏参数中,如果一个参数包含逗号,但是没有被括号包围,那么这个参数会被分割成多个参数.

最后,`MUX_MACRO_PROPERTY(p, macro, a, b)`宏使用`MUX_WITH_COMMA`宏来根据`p`和`macro`连接后的结果是否包含逗号来选择`a`或`b`.这个宏可以用于检查一个宏是否定义了某个属性.例如,如果你有一个名为`FEATURE`的宏,你可以使用`MUX_MACRO_PROPERTY(FEATURE_, CONFIG_FEATURE, a, b)`来检查`CONFIG_FEATURE`是否被定义.如果`CONFIG_FEATURE`被定义,那么宏会展开为`a`,否则宏会展开为`b`.


-------
>`comma`是逗号的意思(哈基米是什么意思呢?)
>`Concat` 是 "concatenate" 的缩写形式,指的是将两个或多个字符串或序列连接在一起的操作.在编程语言和计算机科学中,"concatenate" 是一个常见的术语,用于描述将两个或多个字符串、数组、列表或其他序列按顺序连接成一个更大的序列的操作.
-------

在许多编程语言中,都有专门的函数或操作符用于执行连接操作.例如,在 Python 中,可以使用加号 `+` 运算符来连接字符串或列表.在其他语言中,也可能会有类似的操作符或函数,用于连接不同的数据结构.

因此,"concat" 可以被理解为执行连接操作的动作或功能,通常用于描述在编程中执行这种操作的过程.

当然可以.在C语言中,你可以使用预处理器指令`#define`来定义宏,然后在代码中使用这些宏.以下是如何使用你提供的宏的一个例子：

首先,你需要定义一个`concat`宏,因为`MUX_MACRO_PROPERTY`宏中用到了它.`concat`宏用于连接两个预处理器标记：
### 一些哈批例子

```c
#define concat(a, b) a ## b
```

然后,你可以定义一些特性宏,例如：

```c
#define FEATURE_A
#define FEATURE_B 1,
```

最后,你可以使用`MUX_MACRO_PROPERTY`宏来检查一个特性是否被定义：

```c
MUX_MACRO_PROPERTY(FEATURE_, A, "A is defined", "A is not defined")
MUX_MACRO_PROPERTY(FEATURE_, B, "B is defined", "B is not defined")
MUX_MACRO_PROPERTY(FEATURE_, C, "C is defined", "C is not defined")
```

在这个例子中,`FEATURE_A`被定义,但是没有值,`FEATURE_B`被定义,并且有一个值（1和一个逗号）,`FEATURE_C`没有被定义.

`MUX_MACRO_PROPERTY`宏会将`FEATURE_`和`A`连接,得到`FEATURE_A`,然后检查这个宏是否包含逗号.因为`FEATURE_A`没有值,所以它不包含逗号,所以`MUX_MACRO_PROPERTY`宏会返回"A is not defined".

对于`FEATURE_B`,`MUX_MACRO_PROPERTY`宏会将`FEATURE_`和`B`连接,得到`FEATURE_B`,然后检查这个宏是否包含逗号.因为`FEATURE_B`的值是1和一个逗号,所以它包含逗号,所以`MUX_MACRO_PROPERTY`宏会返回"B is defined".

对于`FEATURE_C`,`MUX_MACRO_PROPERTY`宏会将`FEATURE_`和`C`连接,得到`FEATURE_C`,然后检查这个宏是否包含逗号.因为`FEATURE_C`没有被定义,所以它不包含逗号,所以`MUX_MACRO_PROPERTY`宏会返回"C is not defined".

### 作用
```c
// macro testing
// See https://stackoverflow.com/questions/26099745/test-if-preprocessor-symbol-is-defined-inside-macro
#define CHOOSE2nd(a, b, ...) b
#define MUX_WITH_COMMA(contain_comma, a, b) CHOOSE2nd(contain_comma a, b)
#define MUX_MACRO_PROPERTY(p, macro, a, b) MUX_WITH_COMMA(concat(p, macro), a, b)
// define placeholders for some property
#define __P_DEF_0  X,
#define __P_DEF_1  X,
#define __P_ONE_1  X,
#define __P_ZERO_0 X,
// define some selection functions based on the properties of BOOLEAN macro
#define MUXDEF(macro, X, Y)  MUX_MACRO_PROPERTY(__P_DEF_, macro, X, Y)

typedef struct {
  word_t gpr[MUXDEF(CONFIG_RVE, 16, 32)];
  vaddr_t pc;
} MUXDEF(CONFIG_RV64, riscv64_CPU_state, riscv32_CPU_state);

```
可以根据CONFIG_RV64来命名结构体,这个宏可以根据CONFIG_RVE来选择16或者32,这个宏可以根据CONFIG_RVE来选择16或者32,这个宏可以根据CONFIG_RV64来选择riscv64_CPU_state或者riscv32_CPU_state.但是我发现__P_DEF_0和__P_DEF_1都是X,这样怎么区分?

助教的解答和我的思考,这个主要是跟双井号拼接用法以及【CHOOSE2nd(contain_comma a, b)】（一定注意contain_comma和a之间是没有逗号的）有关
1、如果CONFIG_RV64没有被定义,则有【MUX_MACRO_PROPERTY(__P_DEF_, CONFIG_RV64, riscv64_CPU_state, riscv32_CPU_state)】,其中的【__P_DEF_】和【CONFIG_RV64】都是空,双井号拼接也是空,于是有【CHOOSE2nd(riscv64_CPU_state, riscv32_CPU_state)】,此时取的是riscv32_CPU_state2、同理如果CONFIG_RV64被定义,则有【MUX_MACRO_PROPERTY(__P_DEF_, CONFIG_RV64, riscv64_CPU_state, riscv32_CPU_state)】,字符串拼接后形成【CHOOSE2nd(__P_DEF_1, riscv64_CPU_state, riscv32_CPU_state)】,此时取的是riscv64_CPU_state
总之,__P_DEF_0和__P_DEF_1的值有作用,但是不是直接作用在这个地方,而是通过字符串拼接后的结果来体现作用的.

还要考虑到当选择riscv32时`generate/auto.h`中的`CONFIG_RV64`是没有定义的,如同助教的解答,__P_DEF和CONFIG_RV64都是空,所以最终选择的是riscv32_CPU_state,而当`CONFIG_RV64`被定义时,最终选择的是riscv64_CPU_state.

> 要注意的地方就是当宏没有被定义时,默认是**空**

#### 直接启动出现重复打印命令的问题
```shell
gdb -s /home/along/ysyx-workbench/nemu/build/riscv32-nemu-interpreter --args /home/along/ysyx-workbench/nemu/build/riscv32-nemu-interpreter --log=/home/along/ysyx-workbench/nemu/build/nemu-log.txt  
gdb -s /home/along/ysyx-workbench/nemu/build/riscv32-nemu-interpreter --args /home/along/ysyx-workbench/nemu/build/riscv32-nemu-interpreter 
```
直接启动应该是log导致的,会将本来输出到log的输出到stdin,所以会重复打印
### 内存扫描
内存是什么?
这段代码是NEMU（一个简单的计算机模拟器）中的一部分,用于处理物理地址空间的读写操作.

首先,代码定义了一个名为`pmem`的静态数组,用于模拟物理内存.如果定义了`CONFIG_PMEM_MALLOC`,则`pmem`会被定义为一个指针,并在`init_mem`函数中通过`malloc`分配内存.否则,`pmem`会被定义为一个大小为`CONFIG_MSIZE`的数组.

`guest_to_host`和`host_to_guest`这两个函数用于将客户机地址（即虚拟机中的地址）转换为主机地址（即模拟器中的地址）,反之亦然.这两个函数通过加减`CONFIG_MBASE`（即物理内存的基地址）来进行转换.

`pmem_read`和`pmem_write`这两个函数用于读写物理内存.它们首先将物理地址转换为主机地址,然后调用`host_read`或`host_write`来进行实际的读写操作.

`out_of_bound`函数用于处理地址越界的情况.如果试图访问的地址不在物理内存的范围内,这个函数会被调用,并打印出错误信息.

`init_mem`函数用于初始化物理内存.如果定义了`CONFIG_PMEM_MALLOC`,它会分配内存给`pmem`.如果定义了`CONFIG_MEM_RANDOM`,它会用随机数填充物理内存.

`paddr_read`和`paddr_write`这两个函数是物理地址空间的读写接口.它们首先检查要访问的地址是否在物理内存的范围内.如果是,它们会调用`pmem_read`或`pmem_write`来进行读写操作.如果不是,它们会检查是否定义了`CONFIG_DEVICE`.如果定义了`CONFIG_DEVICE`,它们会调用`mmio_read`或`mmio_write`来进行读写操作.如果地址既不在物理内存的范围内,也没有定义`CONFIG_DEVICE`,它们会调用`out_of_bound`来处理地址越界的情况.

#### pmem_read
这段代码定义了一个名为`host_read`的内联函数,它用于从主机内存中读取数据.这个函数接受两个参数：`addr`和`len`.`addr`是一个指向要读取的内存地址的指针,`len`是要读取的数据的长度（以字节为单位）.

函数体是一个`switch`语句,它根据`len`的值来决定如何读取数据.如果`len`是1,那么就读取一个字节（8位）的数据,并将其作为`uint8_t`类型返回.如果`len`是2,那么就读取两个字节（16位）的数据,并将其作为`uint16_t`类型返回.如果`len`是4,那么就读取四个字节（32位）的数据,并将其作为`uint32_t`类型返回.

如果定义了`CONFIG_ISA64`宏（这通常意味着我们正在使用64位的指令集架构）,并且`len`是8,那么就读取八个字节（64位）的数据,并将其作为`uint64_t`类型返回.

如果`len`的值不是1、2、4或8（在使用64位指令集架构时）,那么就会执行`default`分支.如果定义了`CONFIG_RT_CHECK`宏（这通常意味着我们正在进行运行时检查）,那么就会执行`assert(0)`,这会导致程序在此处断言失败并终止.如果没有定义`CONFIG_RT_CHECK`宏,那么就会返回0.

总的来说,`host_read`函数是一个通用的内存读取函数,它可以读取不同长度的数据,并将其作为适当的无符号整数类型返回.
调用关系链条
vaddr_read->paddr_read->pmem_read->host_read
我想用好宏=>根据事先定义好的宏来决定vaddr_read(expr,len)中的len


#### host和guest
在虚拟化技术中,"host"和"guest"是常用的术语.

"Host"通常指的是运行虚拟化软件的物理机器或操作系统.例如,如果你在Windows系统上运行一个虚拟机软件（如VMware或VirtualBox）,那么Windows就是host.

"Guest"则指的是运行在虚拟机上的操作系统.继续上面的例子,如果你在VMware或VirtualBox中安装了一个Linux系统,那么这个Linux系统就是guest.

在你提到的NEMU模拟器中,"host"指的是运行NEMU的操作系统,"guest"指的是被NEMU模拟的计算机系统."guest_to_host"和"host_to_guest"这两个函数用于在host和guest之间转换地址,因为虚拟机中的地址（guest地址）和物理机器中的地址（host地址）是不同的."Guest"和"Host"的本意来自英语中的日常用法.

"Host"在英语中的原意是“主人”或“主持人”,在特定的环境或事件中,主人通常是提供空间、设施或资源的一方.

"Guest"在英语中的原意是“客人”或“访客”,在特定的环境或事件中,客人通常是被邀请来使用或享受主人提供的空间、设施或资源的一方.

在计算机科学中,这两个词被借用来描述虚拟化环境中的两个角色."Host"是运行虚拟化软件的物理机器或操作系统,提供硬件资源和运行环境,就像一个“主人”."Guest"是运行在虚拟机上的操作系统或应用程序,使用由"Host"提供的资源和环境,就像一个“客人”.

=== *haddr和paddr切换

```c
static uint8_t pmem[CONFIG_MSIZE]PG_ALIGN = {};

*haddr=pmem+paddr-CONFIG_MIZE
```
x86的物理内存是从0开始编址的, 但对于一些ISA来说却不是这样, 例如mips32和riscv32的物理地址均从 0x80000000开始. 因此对于mips32和riscv32, 其 CONFIG_MBASE将会被定义成 0x80000000. 将来CPU访问内存时, 我们会将CPU将要访问的内存地址映射到 pmem中的相应偏移位置, 这是通过 nemu/src/memory/paddr.c中的 guest_to_host()函数实现的. 例如如果mips32的CPU打算访问内存地址 0x80000000, 我们会让它最终访问 pmem[0], 从而可以正确访问客户程序--log=/home/along/ysyx-workbench/nemu/build/nemu-log.txt  却不是这样, 例如mips32和riscv32的物理地址均从0x80000000开始. 因此对于mips32和riscv32, 其CONFIG_MBASE将会被定义成0x8000000

内存通过这个pmem大数组来表示

```
pmem:

CONFIG_MBASE      RESET_VECTOR
      |                 |
      v                 v
      -----------------------------------------------
      |                 |                  |
      |                 |    guest prog    |
      |                 |                  |
      -----------------------------------------------
                        ^
                        |
                       pc

```



### copy-paste
每次写到这种代码的固然有点难蚌,但是还是南蚌.比如给cmd_x写错误处理代码时候,我想直接给每个这样的函数在遇到无参数的时候都整个参数提醒,
其实只要输出这些函数在cmd_table里的usage介绍就行了,但是懒惰让我直接接受copilot给我的copy-paste

### 表达式实现
在TRM中, 寄存器(包括PC)和内存中的值唯一地确定了计算机的一个状态. 因此从道理上来讲, 打印寄存器和扫描内存这两个功能一定可以帮助我们调试出所有的问题. 但为了方便使用, 我们还希望简易调试器能帮我们计算一些带有寄存器和内存的表达式. 所以你需要在简易调试器中添加表达式求值的功能. 为了简单起见, 我们先来考虑数学表达式的求值实现.
数学表达式求值
给你一个表达式的字符串

`5 + 4 * 3 / 2 - 1`
你如何求出它的值? 表达式求值是一个很经典的问题, 以至于有很多方法来解决它. 我们在所需知识和难度两方面做了权衡, 在这里使用如下方法来解决表达式求值的问题:

首先识别出表达式中的单元
根据表达式的归纳定义进行递归求值
词法分析
"词法分析"这个词看上去很高端, 说白了就是做上面的第1件事情, "识别出表达式中的单元". 这里的"单元"是指有独立含义的子串, 它们正式的称呼叫`token`. 具体地说, 我们需要在上述表达式中识别出`5`, `+`, `4`, `*`, `3`, `/`, `2`, `-`, `1`这些`token`. 你可能会觉得这是一件很简单的事情, 但考虑以下的表达式:

``` shell

"0x80100000+   ($a0 +5)*4 - *(  $t1 + 8) + number"

```

它包含更多的功能, 例如十六进制整数(0x80100000), 小括号, 访问寄存器($a0), 指针解引用(第二个*), 访问变量(number). 事实上, 这种复杂的表达式在调试过程中经常用到, 而且你需要在空格数目不固定(0个或多个)的情况下仍然能正确识别出其中的token. 当然你仍然可以手动进行处理(如果你喜欢挑战性的工作的话), 一种更方便快捷的做法是使用正则表达式. 正则表达式可以很方便地匹配出一些复杂的pattern, 是程序员必须掌握的内容. 如果你从来没有接触过正则表达式, 请查阅相关资料. 在实验中, 你只需要了解正则表达式的一些基本知识就可以了(例如元字符).

#### 上次对表达式的实现很蠢
我想有个好一点的实现?二叉树?或者还是什么其他的东西?但是先别急,先看看这个实现.如果你喜欢挑战性的工作,一种更方便快捷的方法是使用正则表达式
#### 如何调试
不要使用"目光调试法", 要思考如何用正确的工具和方法帮助调试
程序设计课上盯着几十行的程序, 你或许还能在大脑中像NEMU那样模拟程序的执行过程; 但程序规模大了之后, 很快你就会放弃的: 你的大脑不可能模拟得了一个巨大的状态机
我们学习计算机是为了学习计算机的工作原理, 而不是学习如何像计算机那样机械地工作
使用`assert()`设置检查点, 拦截非预期情况
例如`assert(p != NULL)`就可以拦截由空指针解引用引起的段错误
结合对程序执行行为的理解, 使用`printf()`查看程序执行的情况(注意字符串要换行)
`printf()`输出任意信息可以检查代码可达性: 输出了相应信息, 当且仅当相应的代码块被执行
`printf()`输出变量的值, 可以检查其变化过程与原因
使用GDB观察程序的任意状态和行为
打印变量, 断点, 监视点, 函数调用栈...
如果你突然觉得上述方法很有道理, 说明你在程序设计课上没有受到该有的训练.

三个对调试有用的宏(在 `nemu/include/debug.h中定义`)
`Log()`是 `printf()`的升级版, 专门用来输出调试信息, 同时还会输出使用 `Log()`所在的源文件, 行号和函数. 当输出的调试信息过多的时候, 可以很方便地定位到代码中的相关位置（非常好用！！！！非常推荐）
`Assert()`是 `assert()`的升级版, 当测试条件为假时, 在`assertion fail`之前可以输出一些信息
panic()用于输出信息并结束程序, 相当于无条件的`assertion fail`

``` c
#define Log(format, ...) \
    _Log(ANSI_FMT("[%s:%d %s] " format, ANSI_FG_BLUE) "\n", \
        __FILE__, __LINE__, __func__, ## __VA_ARGS__)

#define Assert(cond, format, ...) \
  do { \
    if (!(cond)) { \
      MUXDEF(CONFIG_TARGET_AM, printf(ANSI_FMT(format, ANSI_FG_RED) "\n", ## __VA_ARGS__), \
        (fflush(stdout), fprintf(stderr, ANSI_FMT(format, ANSI_FG_RED) "\n", ##  __VA_ARGS__))); \
      IFNDEF(CONFIG_TARGET_AM, extern FILE* log_fp; fflush(log_fp)); \
      extern void assert_fail_msg(); \
      assert_fail_msg(); \
      assert(cond); \
    } \
  } while (0)

```
这段代码是一个在C语言中定义的宏,名为`Log`.宏在C语言中是一种预处理器指令,它们在编译代码之前由预处理器处理.宏可以用来创建可以在代码中重复使用的代码片段.

`Log`宏接受两个参数：一个格式字符串（`format`）和一个可变参数列表（`__VA_ARGS__`）.这个宏的主要功能是生成一条日志消息,并将其打印出来.

`_Log`函数是这个宏的核心,它负责生成和打印日志消息.`_Log`函数的第一个参数是一个格式字符串,它使用`ANSI_FMT`宏和`ANSI_FG_BLUE`宏来设置消息的颜色为蓝色.格式字符串中的`[%s:%d %s]`用于插入文件名、行号和函数名,这些信息由`__FILE__`、`__LINE__`和`__func__`预处理器宏提供.`format`参数是用户提供的消息格式,它被直接插入到格式字符串中.

`## __VA_ARGS__`是一个特殊的预处理器操作符,它用于处理可变参数列表.在这个宏中,它将用户提供的所有参数插入到`_Log`函数的参数列表中.

总的来说,这个`Log`宏提供了一种方便的方式来生成和打印带有文件名、行号和函数名的日志消息.
#### 如何编辑
在VS Code中,你可以使用"多光标编辑"功能来同时编辑多个相同的词.以下是使用这个功能的步骤：

1. 首先,选择你想要编辑的词.

2. 然后,按`Ctrl+D`（在macOS上是`Cmd+D`）.这将在下一个相同的词处添加一个新的光标.

3. 重复按`Ctrl+D`（或`Cmd+D`）,直到你为所有想要编辑的词添加了光标.

4. 现在,你可以同时编辑所有的词.当你输入或删除字符时,所有的光标都会执行相同的操作.

请注意,你也可以使用`Ctrl+Shift+L`（在macOS上是`Cmd+Shift+L`）一次性为所有相同的词添加光标.

#### static 和 inline
+ 什么是translation unit

"Translation unit" 是指编译器处理的最小单元,通常是一个源文件（如 `.c` 文件）以及它所包含的所有包含文件（头文件）的内容.在编译过程中,编译器将每个源文件及其包含的头文件合并成一个称为 translation unit 的单元进行处理.

每个 translation unit 在编译时都会生成一个目标文件（object file）,最终在链接阶段将这些目标文件合并成可执行文件.

一个 translation unit 可以包含如下内容：

1. **全局变量和函数的声明和定义**：包括静态全局变量和函数.

2. **函数的定义**：函数的实现.

3. **宏定义**：在源文件中定义的预处理器宏.

4. **类型定义**：包括结构体、联合体和枚举等类型的定义.

在编译时,编译器将会对每个 translation unit 进行独立编译,并生成对应的目标文件.然后,链接器将这些目标文件合并成一个可执行文件或者库文件.

在 C 和 C++ 中,每个源文件通常对应一个 translation unit.因此,了解 translation unit 的概念对于理解编译和链接过程非常重要.
+ 两个translaton unit同时引用,就导致multiple definition

是的,当两个或多个独立的 translation units（通常是不同的源文件）中都定义了相同的全局变量、函数或其他具有外部链接属性的实体时,就会出现多重定义（multiple definition）错误.

这种错误通常在链接阶段被检测到,因为链接器负责将各个 translation units 的目标文件合并成最终的可执行文件或者库文件.当链接器在合并时发现多个 translation units 中存在相同名称的全局变量或函数时,就会报告多重定义错误.

解决这种错误的方法通常有以下几种：

1. **使用 `static` 修饰符**：如果某个全局变量或函数只在当前文件内部使用,可以使用 `static` 修饰符将其声明为具有内部链接属性,从而避免与其他文件的实体冲突.

2. **使用 `extern` 关键字进行声明**：如果某个全局变量或函数需要在多个文件之间共享,可以在其中一个文件中进行定义,而在其他文件中使用 `extern` 关键字进行声明,这样可以避免多重定义错误.

3. **使用头文件**：将函数声明、全局变量声明以及其他外部实体的声明放在头文件中,在需要使用这些实体的文件中包含相应的头文件,这样可以确保每个文件都使用同一份定义,避免多重定义错误.

4. **使用命名空间（C++）**：在 C++ 中,可以使用命名空间来避免全局作用域中的名称冲突,这样即使在不同的文件中有相同名称的实体,它们也不会发生冲突.

通过以上方法,可以有效地避免多重定义错误,并确保程序能够正确地链接和执行.

+ 什么是extern linkage什么是internal linkage

在 C 和 C++ 中,全局变量、函数和其他实体可以具有不同的链接属性,这些链接属性决定了这些实体的可见性和共享性.主要有两种链接属性：

1. **外部链接（External Linkage）**：具有外部链接的实体在整个程序中可见,可以被其他 translation units（独立的编译单元,通常是不同的源文件）访问和使用.这意味着,如果在一个文件中定义了具有外部链接的实体,其他文件也可以通过声明来引用它,然后在链接时将其合并.通常,全局变量和函数默认具有外部链接.

   ```c
   // 文件1.c
   int global_variable = 10; // 具有外部链接的全局变量

   // 文件2.c
   extern int global_variable; // 引用文件1.c中的全局变量
   ```

2. **内部链接（Internal Linkage）**：具有内部链接的实体只在当前 translation unit 内可见,其他 translation units 无法访问.这意味着,具有内部链接的实体只能在定义它们的文件内部使用,其他文件无法直接引用它们.通常,使用 `static` 修饰符声明的全局变量和函数具有内部链接.

   ```c
   // 文件1.c
   static int internal_variable = 20; // 具有内部链接的静态全局变量
   static void internal_function() {} // 具有内部链接的静态函数
   ```

   ```c
   // 文件2.c
   extern int internal_variable; // 错误：无法引用具有内部链接的变量
   ```

总之,外部链接的实体可以在多个文件之间共享,而内部链接的实体只能在当前文件内部使用.这种区别使得我们能够更好地控制程序中实体的可见性和共享性.

#### 为什么不能在头文件中实现函数?
`a.h`中实现一个`foo`函数,`b.c`和`c.c`都包含了`a.h`,这样会导致`foo`函数在`b.c`和`c.c`中都被定义了一次,这样会导致链接错误.所以在头文件中只能声明函数,不能实现函数.include就是将文件的内容复制到当前文件中,所以如果在头文件中实现函数,那么这个函数的定义就会被复制到每个包含这个头文件的文件中,从而导致多重定义错误.如果把foo的实现定义为static,那么就不会有这个问题,因为static修饰的函数只能在当前文件中使用.

##### -Werror -Wall
-Werror选项会将所有的警告都视为错误,这意味着如果代码中有任何警告,编译器都会报错并停止编译.这样做的好处是可以确保代码的质量,因为任何警告都会被视为错误,开发人员就会更加注意避免产生警告.

-Wall选项会开启所有的警告,这意味着编译器会检查代码中的任何潜在问题并给出警告.这样做的好处是可以帮助开发人员发现代码中的潜在问题,并及时进行修复.

因此,使用-Werror -Wall选项可以确保代码的质量,并帮助开发人员发现潜在的问题,从而提高代码的可靠性和稳定性.

如果你在头文件中实现函数,并且给这个函数添加了`static`修饰符,那么这个函数就变成了静态函数.静态函数在C语言中有特殊的含义,它表示这个函数只在定义它的文件中可见.

这样做可以解决在头文件中实现函数可能出现的链接错误问题,因为每个包含这个头文件的源文件都会得到这个函数的一个私有拷贝,这些拷贝在链接阶段不会互相冲突.

然而,这仍然不能解决编译时间和代码重复的问题.每个包含这个头文件的源文件在编译时都会编译这个函数,这可能会导致编译时间增加.同时,每个源文件都会有这个函数的一份拷贝,这可能会导致生成的可执行文件的大小增加.

此外,这种做法也可能会导致代码的可读性和可维护性降低.通常,我们期望在头文件中看到的是接口声明,而在源文件中看到的是接口的实现.如果在头文件中看到了函数的实现,这可能会让人感到困惑.

因此,虽然在头文件中实现静态函数是可以的,但通常我们还是推荐在头文件中声明函数,在源文件中实现函数.

定义成static inline的函数,可以在头文件中定义,这样可以避免多重定义错误.这种函数会被编译器内联,会增加编译时间.这种函数适合于短小的函数,例如一些简单的getter和setter函数.

什么叫做被编译器内联? 内联函数是一种特殊的函数,它的定义和声明都在一起,通常在头文件中.内联函数的特点是在调用它的地方直接展开,而不是像普通函数一样生成一个函数调用.这样可以减少函数调用的开销,提高程序的执行效率.

#### 关于regex
REGEX(3)                                      Linux Programmer's Manual                                      REGEX(3)

NAME
       regcomp, regexec, regerror, regfree - POSIX regex functions
+ regcomp()
 regcomp()  is  used  to  compile  a  regular  expression into a form that is suitable for subsequent regexec()
       searches.

       regcomp() is supplied with preg, a pointer to a pattern buffer storage area; regex, a pointer to the null-ter‐
       minated string and cflags, flags used to determine the type of compilation.

       All  regular  expression  searching  must be done via a compiled pattern buffer, thus regexec() must always be
       supplied with the address of a regcomp() initialized pattern buffer.
+ regexec()

 ```bash
       The regexec() function is used to match a null-terminated string against the precompiled pattern buffer preg.
       The string is pointed to by string and the length of the string is len.  The eflags argument is used to deter‐
       mine the type of matching that will be performed.
  The regmatch_t structure which is the type of pmatch is defined in <regex.h>.

           typedef struct {
               regoff_t rm_so;
               regoff_t rm_eo;
           } regmatch_t;

       Each  rm_so  element that is not -1 indicates the start offset of the next largest substring match within the string.  The relative rm_eo element indicates the end offset of the match, which is the offset of the first character after the matching text.
```
#### strtol()
strtol()是C语言中的一个标准库函数,用于将字符串转换为长整型数.它的函数原型如下：

```c
long int strtol(const char *nptr, char **endptr, int base);
```
strtol()函数接受三个参数：`nptr`是一个指向待转换的字符串的指针,`endptr`是一个指向指针的指针,`base`是一个整数,表示转换时所使用的进制.

strtol()函数会将`nptr`指向的字符串转换为一个长整型数,并返回这个数.如果`endptr`不是NULL,那么strtol()会将`nptr`中第一个无法转换的字符的指针赋值给`endptr`.如果`endptr`是NULL,那么strtol()会忽略无法转换的字符.

`base`参数表示转换时所使用的进制.如果`base`为0,那么strtol()会根据字符串的格式自动判断进制.如果`base`不为0,那么strtol()会将字符串按照指定的进制进行转换.

strtol()函数通常用于将字符串转换为整数,例如：

```c
char str[] = "12345";
char *endptr;
long int num = strtol(str, &endptr, 10);
```
在这个例子中,`strtol()`函数将字符串`"12345"`转换为一个长整型数,并将结果存入`num`中.如果`endptr`不是NULL,那么`strtol()`会将`str`中第一个无法转换的字符的指针赋值给`endptr`.

总的来说,`strtol()`函数是一个用于将字符串转换为长整型数的标准库函数,它可以根据指定的进制进行转换,并且提供了一种方便的方法来处理转换失败的情况.

#### strcmp()
strcmp()是C语言中的一个标准库函数,用于比较两个字符串.它的函数原型如下：

```c
int strcmp(const char *str1, const char *str2);
```
strcmp()函数接受两个参数：`str1`和`str2`分别是待比较的两个字符串的指针.

strcmp()函数会比较`str1`和`str2`指向的字符串,并返回一个整数.如果`str1`和`str2`相等,那么strcmp()会返回0.如果`str1`小于`str2`,那么strcmp()会返回一个小于0的整数.如果`str1`大于`str2`,那么strcmp()会返回一个大于0的整数.

strcmp()函数通常用于比较字符串,例如：

```c
char str1[] = "abc";
char str2[] = "def";
int result = strcmp(str1, str2);
```
在这个例子中,`strcmp()`函数将字符串`"abc"`和`"def"`进行比较,并将结果存入`result`中.如果`result`的值为0,那么`str1`和`str2`相等.如果`result`的值小于0,那么`str1`小于`str2`.如果`result`的值大于0,那么`str1`大于`str2`.

总的来说,`strcmp()`函数是一个用于比较字符串的标准库函数,它可以方便地比较两个字符串,并根据比较结果返回一个整数.

#### 为什么要用const char *str
const char *str是一个指向字符的指针,它指向的字符是常量,不能被修改.这样做的好处是可以确保函数不会意外地修改传入的字符串,从而提高了代码的可靠性和可维护性.

#### strncmp()
strncmp()是C语言中的一个标准库函数,用于比较两个字符串的前n个字符.它的函数原型如下：

```c
int strncmp(const char *str1, const char *str2, size_t n);
```
strncmp()函数接受三个参数：`str1`和`str2`分别是待比较的两个字符串的指针,`n`是一个无符号整数,表示要比较的字符数.

strncmp()函数会比较`str1`和`str2`指向的字符串的前n个字符,并返回一个整数.如果`str1`和`str2`的前n个字符相等,那么strncmp()会返回0.如果`str1`的前n个字符小于`str2`的前n个字符,那么strncmp()会返回一个小于0的整数.如果`str1`的前n个字符大于`str2`的前n个字符,那么strncmp()会返回一个大于0的整数.

strncmp()函数通常用于比较字符串的前n个字符,例如：

```c
char str1[] = "abc";
char str2[] = "def";
int result = strncmp(str1, str2, 2);
```
在这个例子中,`strncmp()`函数将字符串`"abc"`和`"def"`的前2个字符进行比较,并将结果存入`result`中.如果`result`的值为0,那么`str1`和`str2`的前2个字符相等.如果`result`的值小于0,那么`str1`的前2个字符小于`str2`的前2个字符.如果`result`的值大于0,那么`str1`的前2个字符大于`str2`的前2个字符.

总的来说,`strncmp()`函数是一个用于比较字符串的前n个字符的标准库函数,它可以方便地比较两个字符串的前n个字符,并根据比较结果返回一个


### 表达式扩展
表达式扩展的含金量 => 本来是仅支持常数运算,但是这并不能用在后面的调试工程 => 要支持,对Hex的识别,对-1,+1,对指针解引用,对寄存器的识别.
`+-*/`(二元运算),`+-`(单元运算),
##### 对重复代码的再次抽象
```c

word_t eval(int p,int q, bool *sucess){
  if(p>q){
    // bad expression
    *sucess = false;
    return 0;
  }
  else if(p==q){
    /* Single token.
     * For now this token should be a number.
     * Return the value of the number.
     */
    if(tokens[p].type != TK_NUM){
      *sucess = false;
      return 0;
    }
    return atoi(tokens[p].str);
  }
  else if(check_parentgeses(p,q)==true){
    // remove the outermost brackets
    return eval(p+1,q-1,sucess);
  }else{
    int op,val1,val2;
    op = prioty(p,q);
    if(op==-1){
      *sucess = false;
      return 0;
    }
    // val1 = eval(p,op-1,sucess);
    // if(*sucess==false)return 0;
    // val2 = eval(op+1,q,sucess);
    // if(*sucess==false)return 0;
    switch (tokens[op].type)
    {
    case '+':{
      val1 = eval(p,op-1,sucess);
      if(*sucess==false)return 0;
      val2 = eval(op+1,q,sucess);
      if(*sucess==false)return 0;
      return val1+val2;
    }
    case '-':{
      val1 = eval(p,op-1,sucess);
      if(*sucess==false)return 0;
      val2 = eval(op+1,q,sucess);
      if(*sucess==false)return 0;
      return val1-val2;
    }
    case '*':{
      val1 = eval(p,op-1,sucess);
      if(*sucess==false)return 0;
      val2 = eval(op+1,q,sucess);
      if(*sucess==false)return 0;
      return val1*val2;}
    case '/': {
      val1 = eval(p,op-1,sucess);
      if(*sucess==false)return 0;
      val2 = eval(op+1,q,sucess);
      if(*sucess==false)return 0;
      if(val2==0){
        *sucess = false;
        return 0;
      }
      return val1 / val2;
    }
    case TK_NEG:{ 
      // printf("has been here");
      val2 = eval(op+1,q,sucess);
      return -val2;
    }
    default: assert(0);
    }
  }
  return 0;
}

```
我采取的解决方法
```c
    // Makes it easier for the program 
    //to run on machines with different word widths.
    int len = MUXDEF(CONFIG_RV64, 8, 4);
    int width = MUXDEF(CONFIG_RV64, 18, 10);
    printf(ANSI_FMT("%#0*x: ", ANSI_FG_CYAN),width,expr);
    for (j = 0; i < n && j < len; i++, j++) {
      word_t w = vaddr_read(expr, len);
      expr += len;
      printf("%#0*x ",width,w);
    }
    puts(""); 
```
一开始我不理解为什么要写一个双层循环
```shell
(nemu) x 10 0x80000000
0x80000000: 0x00000297 0x00028823 0x0102c503 0x00100073 
0x80000010: 0xdeadbeef 0x65656565 0x65656565 0x65656565 
0x80000020: 0x65656565 0x65656565 

```
要保证这样的输出,就需要一个双层循环,第一个循环是为了保证输出的行数,第二个循环是为了保证输出的列数,这样就可以保证输出的格式是正确的.
##### 关于risc32=>4,riscv64=>8
处理右结合的指针解引用(de+reference),需要注意的是
```c
static word_t calunary(word_t val,int op,bool *sucess){
  switch (op)
  {
  case TK_DEREF:
    return vaddr_read(val,4);
  case TK_NEG:
    return -val;
  default:
    *sucess = false;
    return 0;
  }
}

```



### watch_point
请告诉我gdb中是如何使用监视点功能的
在 GDB 中,你可以使用以下步骤来使用监视点（watchpoint）功能：

1. **设置监视点**：使用 `watch` 命令来设置一个监视点.这个命令的格式是 `watch expr`,其中 `expr` 是你想要监视的表达式.当这个表达式的值发生改变时,程序的执行就会暂停.例如,如果你想要在变量 `x` 的值发生改变时暂停程序的执行,你可以使用以下命令：

   ```
   watch x
   ```

2. **查看监视点**：使用 `info watchpoints` 命令来查看当前设置的所有监视点.这个命令会显示一个列表,列出所有当前设置的监视点.对于每个监视点,它会显示监视点的编号,监视的表达式,以及表达式的当前值.

3. **删除监视点**：使用 `delete` 命令来删除一个监视点.你需要在 `delete` 命令后面加上你想要删除的监视点的编号.例如,如果你想要删除编号为 1 的监视点,你可以使用以下命令：

   ```
   delete 1
   ```

   你也可以使用 `delete` 命令而不带任何参数来删除所有的断点和监视点.

这就是在 GDB 中使用监视点功能的基本步骤.请注意,监视点是一种强大的工具,但是它们可能会显著地降低你的程序的运行速度,特别是当你监视的表达式涉及到复杂的计算或者大量的内存访问时.

## PA2
yemu模拟器
```c
#include <stdint.h>
#include <stdio.h>

#define NREG 4
#define NMEM 16

// 定义指令格式
typedef union {
  struct { uint8_t rs : 2, rt : 2, op : 4; } rtype;
  struct { uint8_t addr : 4      , op : 4; } mtype;
  uint8_t inst;
} inst_t;
//
#define DECODE_R(inst) uint8_t rt = (inst).rtype.rt, rs = (inst).rtype.rs
#define DECODE_M(inst) uint8_t addr = (inst).mtype.addr

uint8_t pc = 0;       // PC, C语言中没有4位的数据类型, 我们采用8位类型来表示
uint8_t R[NREG] = {}; // 寄存器
uint8_t M[NMEM] = {   // 内存, 其中包含一个计算z = x + y的程序
  0b11100110,  // load  6#     | R[0] <- M[y]
  0b00000100,  // mov   r1, r0 | R[1] <- R[0]
  0b11100101,  // load  5#     | R[0] <- M[x]
  0b00010001,  // add   r0, r1 | R[0] <- R[0] + R[1]
  0b11110111,  // store 7#     | M[z] <- R[0]
  0b00010000,  // x = 16
  0b00100001,  // y = 33
  0b00000000,  // z = 0
};

int halt = 0; // 结束标志

// 执行一条指令
void exec_once() {
  inst_t this;
  this.inst = M[pc]; // 取指
  switch (this.rtype.op) {
  //  操作码译码       操作数译码           执行
    case 0b0000: { DECODE_R(this); R[rt]   = R[rs];   break; }
    case 0b0001: { DECODE_R(this); R[rt]  += R[rs];   break; }
    case 0b1110: { DECODE_M(this); R[0]    = M[addr]; break; }
    case 0b1111: { DECODE_M(this); M[addr] = R[0];    break; }
    default:
      printf("Invalid instruction with opcode = %x, halting...\n", this.rtype.op);
      halt = 1;
      break;
  }
  pc ++; // 更新PC
}

int main() {
  while (1) {
    exec_once();
    if (halt) break;
  }
  printf("The result of 16 + 33 is %d\n", M[7]);
  return 0;
}
```
被PA2.1难哭了,看不明白这个巨量的宏(what can i say?),晚上回去把要交到邮箱的东西都填完.

### GCC提供的标签地址

使用了GCC提供的标签地址[扩展功能](https://gcc.gnu.org/onlinedocs/gcc/Labels-as-Values.html)

### 告诉GCC总是尝试内联

`__attribute__((always_inline))` 是GCC编译器的一个特性,用于告诉编译器总是尝试将某个函数内联.这是一个函数属性,通常在函数声明之前使用.

内联函数是一种优化技术,它通过将函数调用替换为函数体的内容来减少函数调用的开销.然而,内联函数并不总是提高程序的性能,因为它可能会增加程序的大小,并可能影响指令缓存的效率.因此,编译器通常会根据函数的大小和复杂性来决定是否将其内联.

然而,`__attribute__((always_inline))`属性告诉编译器无论函数的大小和复杂性如何,都应该尝试将其内联.这对于一些关键的、性能敏感的函数可能是有用的,但应该谨慎使用,以避免增加程序的大小和降低指令缓存的效率.

请注意,这个属性是GCC特有的,不是标准C++的一部分.如果你的代码需要在其他编译器上编译,你可能需要找到一个等效的属性或者使用其他方法来控制函数的内联行为. 

### 查看复杂工程中某个特定文件的编译过程

在C语言中,宏是一种预处理器指令,用于在编译时将代码片段替换为指定的文本.宏展开是指将宏调用替换为宏定义的文本的过程.

如果你想查看C文件中复杂宏的展开结果,可以使用GCC的预处理器选项`-E`.这个选项会让GCC在编译过程中只进行预处理,然后将预处理的结果输出到标准输出.

例如,如果你有一个名为`main.c`的文件,你可以使用以下命令来查看宏的展开结果：

```bash
gcc -E main.c
```

这个命令会将`main.c`文件中的所有宏都展开,并将结果输出到控制台.如果你想将结果保存到一个文件中,你可以使用重定向：

```bash
gcc -E main.c > preprocessed.c
```

这个命令会将宏展开的结果保存到`preprocessed.c`文件中.

对于复杂的工程文件,你仍然可以使用GCC的预处理器选项`-E`来查看宏的展开结果.但是,你可能需要指定一些额外的编译选项,比如头文件的路径、预定义的宏等.

如果你使用的是make工具来构建你的项目,你可以修改Makefile,添加一个新的目标来生成预处理的文件.例如：

```makefile
.PHONY: preprocess
preprocess:
    gcc -E $(CFLAGS) main.c -o preprocessed.c
```

在这个例子中,`$(CFLAGS)`是一个变量,它包含了你的项目需要的所有编译选项.`main.c`是你想要预处理的文件,`preprocessed.c`是生成的预处理文件.

然后,你可以使用`make preprocess`命令来生成预处理文件.

如果你的项目使用的是其他构建工具（如CMake、Bazel等）,你可能需要查阅相应的文档来了解如何生成预处理文件. 根据你想要展开的文件的复杂性,你可能需要调整编译选项和预处理器选项.

#### make -nB 
`make -n`或者`make --just-print`选项是用来显示make命令将要执行的操作,但是并不真正执行这些操作.这个选项通常用于查看make命令的执行过程,以便检查makefile文件是否正确配置.

通过`make -nB | vim -`来定位某个特定文件的编译过程.

```shell
gcc -O2 -MMD -Wall -Werror -I/home/along/ysyx-workbench/nemu/include -I/home/along/ysyx-workbench/nemu/src/isa/riscv32/include -I/home/along/ysyx-workbench/nemu/src/engine/interpreter -O2  -Og -ggdb3  -DITRACE_COND=true -D__GUEST_ISA__=riscv32 -c -o /home/along/ysyx-workbench/nemu/build/obj-riscv32-nemu-interpreter/src/isa/riscv32/inst.o src/isa/riscv32/inst.c

```
把`-o`=>`-E`再把最后面加个`| vim -`经过简单的修改后可以很方便的看到宏展开的样子.


```c
  { const void ** __instpat_end = &&__instpat_end_;;
  do { uint64_t key, mask, shift; pattern_decode("0000000 00001 00000 000 00000 11100 11", (sizeof("0000000 00001 00000 000 00000 11100 11") - 1), &key, &mask, &shift); 
    if ((((uint64_t)((s)->isa.inst.val) >> shift) & mask) == key) { 
      { 
        decode_operand(s, &rd, &src1, &src2, &imm, TYPE_R); 
        set_nemu_state(NEMU_END, s->pc, (cpu.gpr[check_reg_idx(10)])) ; 
      }; 
      goto *(__instpat_end); 
    } 
  } while (0);
  do { uint64_t key, mask, shift; pattern_decode("0000000 00001 00000 000 00000 11100 11", (sizeof("0000000 00001 00000 000 00000 11100 11") - 1), &key, &mask, &shift); 
    if ((((uint64_t)((s)->isa.inst.val) >> shift) & mask) == key) { 
      { 
        decode_operand(s, &rd, &src1, &src2, &imm, TYPE_N); 
        set_nemu_state(NEMU_END, s->pc, (cpu.gpr[check_reg_idx(10)])) ; 
      }; 
      goto *(__instpat_end); 
    } 
  } while (0);
  
  do { uint64_t key, mask, shift; pattern_decode("??????? ????? ????? ??? ????? ????? ??", (sizeof("??????? ????? ????? ??? ????? ????? ??") - 1), &key, &mask, &shift); 
    if ((((uint64_t)((s)->isa.inst.val) >> shift) & mask) == key) { 
      { 
        decode_operand(s, &rd, &src1, &src2, &imm, TYPE_N); 
        invalid_inst(s->pc) ; 
      }; 
      goto *(__instpat_end); 
    } 
  } while (0);
  
  __instpat_end_: ; };

  (cpu.gpr[check_reg_idx(0)]) = 0;

  return 0;
}

int isa_exec_once(Decode *s) {
  s->isa.inst.val = inst_fetch(&s->snpc, 4);
  return decode_exec(s);
}

```

### dummy

```shell
➜   make ARCH=$ISA-nemu ALL=dummy run
# Building dummy-run [-nemu]
/home/along/ysyx-workbench/abstract-machine/Makefile:30: *** Expected $ARCH in {loongarch32r-nemu mips32-nemu native riscv32e-nemu riscv32e-npc riscv32-nemu riscv64-nemu spike x86_64-qemu x86-nemu x86-qemu}, Got "-nemu".  Stop.
 dummy
[         dummy] FAIL!

➜   make ARCH=riscv32-nemu ALL=dummy run  
# Building dummy-run [riscv32-nemu]
+ CC tests/dummy.c
cc1: error: ‘-march=rv32im_zicsr’: unsupported ISA subset ‘z’
make[1]: *** [/home/along/ysyx-workbench/abstract-machine/Makefile:110: /home/along/ysyx-workbench/am-kernels/tests/cpu-tests/build/riscv32-nemu/tests/dummy.o] Error 1
 dummy
[         dummy] FAIL!


```
报错原因是riscv的交叉编译器版本老了.

解决方法就是要么自行编译一个新的交叉编译器,要么直接下压缩包

`/opt` 是 Linux 文件系统中的一个目录,通常用于存放可选的应用程序软件包.

在 Linux 系统中,`/opt` 目录通常用于存放第三方软件或额外的软件包,这些软件包可能不是由系统的包管理器管理的.例如,一些商业软件,如 Oracle 数据库,可能会选择在 `/opt` 目录下安装.

在 `/opt` 目录下安装软件可以使得系统的其他部分保持整洁,因为所有的文件都在 `/opt` 下的一个目录中,这使得卸载软件变得非常简单,只需要删除这个目录即可.

总的来说,`/opt` 是一个用于存放可选软件包的目录.

本来一直没有打算搜索报错的,最后还是搜索了报错.

### dummy运行
```shell
(nemu) s
invalid opcode(PC = 0x80000000):
	13 04 00 00 17 91 00 00 ...
	00000413 00009117...
There are two cases which will trigger this unexpected exception:
1. The instruction at PC = 0x80000000 is not implemented.
2. Something is implemented incorrectly.
Find this PC(0x80000000) in the disassembling result to distinguish which case it is.

If it is the first case, see
       _                         __  __                         _ 
      (_)                       |  \/  |                       | |
  _ __ _ ___  ___ ________   __ | \  / | __ _ _ __  _   _  __ _| |
 | '__| / __|/ __|______\ \ / / | |\/| |/ _` | '_ \| | | |/ _` | |
 | |  | \__ \ (__        \ V /  | |  | | (_| | | | | |_| | (_| | |
 |_|  |_|___/\___|        \_/   |_|  |_|\__,_|_| |_|\__,_|\__,_|_|

for more details.

If it is the second case, remember:
* The machine is always right!
* Every line of untested code is always wrong!

0x80000000: 00 00 04 13 addi	s0, zero, 0
[src/cpu/cpu-exec.c:121 cpu_exec] nemu: ABORT at pc = 0x80000000
[src/cpu/cpu-exec.c:89 statistic] host time spent = 311 us
[src/cpu/cpu-exec.c:90 statistic] total guest instructions = 1
[src/cpu/cpu-exec.c:91 statistic] simulation frequency = 3,215 inst/s
```
学习 RISC-V 的语义和指令格式是理解和使用这一现代指令集架构的关键.RISC-V 是一个开源、简洁、模块化的指令集架构,设计用于各种计算设备,从嵌入式系统到超级计算机都可以使用.以下是 RISC-V 的语义和指令格式的基本介绍：

### RISC-V 指令集概述

1. **简介**：
   - RISC-V（发音为 "risk-five"）是一个基于精简指令集（RISC）原则的开放指令集架构（ISA）.
   - 设计简洁,包含了基本的指令集和标准的扩展,可以根据需求进行自定义扩展.

2. **指令格式**：
   - RISC-V 指令有固定的长度（通常为 32 位）,可以分为不同的类型：R 类型（寄存器-寄存器操作）、I 类型（立即数操作）、S 类型（存储操作）、B 类型（分支操作）、U 类型（无条件跳转）、J 类型（条件跳转）等.

3. **寄存器**：
   - RISC-V 使用整数寄存器（通常有 32 个,命名为 x0 到 x31）,以及一些特殊用途的寄存器（如程序计数器 PC、栈指针 SP 等）.

4. **指令语义**：
   - 每条指令都有明确定义的操作,涉及寄存器之间的数据传输、算术运算、逻辑运算、内存访问和控制流操作等.
   - 指令的语义由操作码（opcode）和操作数（operands）组成,操作数可以是寄存器、立即数或内存地址.

### 指令格式示例

#### R 类型指令（寄存器-寄存器操作）

R 类型指令用于寄存器之间的操作,如加法、减法等.

```
  31    25 24   20 19   15 14   12 11    7 6     0
+-------+-------+-------+-------+-------+-------+
| funct7|  rs2  |  rs1  | funct3|   rd  | opcode|
+-------+-------+-------+-------+-------+-------+
```

- `opcode`: 操作码,指定操作类型（如算术运算、逻辑运算等）.
- `rd`: 目标寄存器.
- `rs1`、`rs2`: 源寄存器.
- `funct3`、`funct7`: 针对操作的具体功能码.

#### I 类型指令（立即数操作）

I 类型指令用于将立即数与寄存器内容进行操作,如加载、存储、移位等.

```
  31    20 19   15 14   12 11    7 6     0
+-------+-------+-------+-------+-------+-------+
| imm[11:0] |  rs1  | funct3|   rd  | opcode|
+-------+-------+-------+-------+-------+-------+
```

- `opcode`: 操作码.
- `rd`: 目标寄存器.
- `rs1`: 源寄存器.
- `imm[11:0]`: 12 位的立即数.

#### S 类型指令（存储操作）

S 类型指令用于存储数据到内存.

```
  31    25 24   20 19   15 14   12 11    7 6     0
+-------+-------+-------+-------+-------+-------+
| imm[11:5]|  rs2  |  rs1  | funct3| imm[4:0]| opcode|
+-------+-------+-------+-------+-------+-------+
```

- `opcode`: 操作码.
- `rs1`、`rs2`: 相关寄存器.
- `imm[11:0]`: 12 位的立即数,用于指定存储的偏移量.

### 学习资源推荐

如果你希望深入了解 RISC-V 的语义和指令格式,以下资源可能会对你有所帮助：

- [RISC-V 指令集手册](https://riscv.org/technical/specifications/)
- [RISC-V 教育基金会的教育资源](https://riscv.org/education/)
- RISC-V 相关的开放课程和在线学习平台（如 Coursera、edX 上的相关课程）

通过这些资源,你可以系统地学习和理解 RISC-V 的指令集架构、指令格式以及操作语义,为进一步的学习和开发应用打下坚实的基础.

### ysyx-workbench/am-kernels/tests/cpu-tests/makefile

`make ARCH=$ISA-nemu ALL=dummy `

#### >.result

在 Makefile 中,`$(shell > $(RESULT))` 这行代码使用了 GNU Make 的 `$(shell ...)` 函数.该函数允许在 Makefile 中执行任意的 shell 命令,并将其输出作为变量值.具体来说,这里执行的命令是 `> .result`,它的作用是创建一个空的 `.result` 文件（如果文件已经存在,则清空它）.

##### `$(shell ...)` 函数

`$(shell ...)` 函数的语法如下：
```makefile
$(shell command)
```

- `command` 是要执行的 shell 命令.
- `$(shell ...)` 函数执行 `command` 并将其输出作为返回值.

##### 创建空文件

在 shell 中,单独使用重定向操作符 `>` 可以创建一个新文件或清空一个已存在的文件.例如：
```sh
> .result
```
这条命令的效果是创建一个名为 `.result` 的空文件,如果 `.result` 文件已经存在,则会清空它的内容.

在 Makefile 中使用 `$(shell ...)` 执行这个命令的目的是确保在开始编译之前,结果文件 `.result` 是空的.这样可以避免将新编译的结果与之前的结果混在一起.

在 Makefile 中,`$(RESULT)` 是一个变量,用于存储编译和运行结果的文件名.在前面的 Makefile 示例中,它被定义为：

```makefile
RESULT = .result
$(shell > $(RESULT))
```

`$(RESULT)` 变量的作用是记录每个测试文件的编译和运行结果.这种做法可以让你在编译多个测试文件后查看它们的结果,而不需要逐个查看编译输出.

### 详细说明

1. **变量定义**：
   ```makefile
   RESULT = .result
   ```

   这行代码将 `RESULT` 变量定义为 `.result`,表示结果文件的名称是 `.result`.

2. **初始化结果文件**：
   ```makefile
   $(shell > $(RESULT))
   ```

   这行代码使用 `$(shell ...)` 函数执行 shell 命令 `> .result`,这会创建一个空的 `.result` 文件（如果文件已经存在,则清空它）.这样可以确保每次运行 Makefile 时,结果文件都是空的,并且不会包含上次运行的结果.

3. **记录编译结果**：
   ```makefile
   @if make -s -f $@ ARCH=$(ARCH) $(MAKECMDGOALS); then \
       printf "[%14s] $(COLOR_GREEN)PASS!$(COLOR_NONE)\n" $* >> $(RESULT); \
   else \
       printf "[%14s] $(COLOR_RED)FAIL!$(COLOR_NONE)\n" $* >> $(RESULT); \
   fi
   ```

   - `printf "[%14s] $(COLOR_GREEN)PASS!$(COLOR_NONE)\n" $* >> $(RESULT);`：如果编译成功,打印一行包含 "PASS!" 的信息,并追加到 `.result` 文件中.
   - `printf "[%14s] $(COLOR_RED)FAIL!$(COLOR_NONE)\n" $* >> $(RESULT);`：如果编译失败,打印一行包含 "FAIL!" 的信息,并追加到 `.result` 文件中.

   这段代码是一个在 Makefile 文件中定义的命令,用于执行构建过程,并根据构建结果输出相应的信息到一个结果文件中.让我们逐步解析这段代码的功能和组成部分.

首先,这段代码使用了 `@` 前缀,这在 Makefile 中用来表示执行命令时,不将该命令本身输出到控制台.这样做可以让输出结果更加清晰,只显示我们关心的信息.($@表示targets,$^表示prerequisites,不如直接去看makefile manual)

接下来,使用 `if` 语句来判断 `make -s -f $@ ARCH=$(ARCH) $(MAKECMDGOALS)` 命令的执行结果.这里的 `-s` 选项表示在执行时不显示命令本身,`-f $@` 表示使用当前目标（`$@`）指定的 Makefile 文件.`ARCH=$(ARCH)` 是向 make 命令传递的一个变量,它的值取决于当前环境或者是在命令行中指定的值.`$(MAKECMDGOALS)` 是一个特殊的变量,包含了 make 命令行中指定的目标列表.

如果 `make` 命令执行成功（即返回值为 0）,则执行 `then` 分支,使用 `printf` 命令将格式化的成功信息追加到 `$(RESULT)` 指定的文件中.这里的 `[%14s]` 是 `printf` 格式字符串,用于将输出的字符串格式化为宽度为 14 个字符的字符串,如果不足则左侧填充空格.`$*` 代表了传递给当前规则的所有参数,`$(COLOR_GREEN)` 和 `$(COLOR_NONE)` 是在 Makefile 中定义的变量,用于设置文本颜色,以便在输出中高亮显示成功信息.

如果 `make` 命令执行失败（即返回值非 0）,则执行 `else` 分支,使用 `printf` 命令将格式化的失败信息追加到 `$(RESULT)` 指定的文件中.失败信息的格式与成功信息相同,不同之处在于使用了 `$(COLOR_RED)` 变量来设置文本颜色,以便在输出中高亮显示失败信息.

总的来说,这段代码通过执行另一个 Makefile 并检查其返回值,来决定是输出构建成功还是失败的信息,并将这些信息以特定的格式追加到一个结果文件中,同时通过颜色的变化使得结果一目了然.

4. **显示结果**：
   ```makefile
   run: all
       @cat $(RESULT)
       @rm $(RESULT)
   ```

   - `@cat $(RESULT)`：显示 `.result` 文件的内容,列出所有测试文件的编译和运行结果.
   - `@rm $(RESULT)`：删除 `.result` 文件,清理临时文件.

#### 示例

假设你有两个测试文件 `tests/dummy.c` 和 `tests/sample.c`,并且你运行 `make ARCH=$ISA-nemu ALL="dummy sample" run`.假设 `dummy.c` 编译成功而 `sample.c` 编译失败,`.result` 文件的内容可能如下：

```
        dummy PASS!
       sample FAIL!
```

在 `run` 目标中,`cat .result` 将显示上述内容,`rm .result` 则会删除 `.result` 文件.

#### 总结

`$(RESULT)` 是一个用于存储和显示编译和运行结果的临时文件名变量.它的定义和使用确保了每次运行 Makefile 时都能记录每个测试文件的结果,并在最终输出结果后清理该文件.


### PA2.1 指令实现
#### 教训 
YSYX的这个开放架构设计之道有一点没写好(例如结构太混乱了,介绍基本指令格式的时候,U型指令的立即数20位左移都没说,就开始介绍减少扇形面积之类的知识看得很一头雾水,怪不得很多blog说不如直接去看官方手册,这点确实,唉)

指令实现就是纯抄手册,补全框架代码里的空白,这个没什么好说的,就是要多看手册,多看手册,多看手册.(例如取立即数的几个imm,)难绷拿copilot生成的取立即数宏居然是错的,当时偷懒了没去看手册,结果浪费了很多时间测试出这个问题.

#### gdb条件调试

mersenne测试未通过,需要使用gdb进行调试.
先break到大概地方再使用,condition来完成条件调试.
(笨办法自然是s N手动找出来那个step会abortion的地方)

```shell
(gdb) break exec_once
(gdb) condition 1 (s->dnpc == 0x7ffffcdc)
```

#### make -nB | vim -
`sed` `grep` ()
``` vim编辑命令
# 只保留gcc或g++开头的行
:%!grep "^\(gcc\|g++\)"

# 将环境变量$NEMU_HOME所指示字符串替换为$NEMU_HOME
:%!sed -e "s+$NEMU_HOME+\$NEMU_HOME+g"

# 将$NEMU_HOME/build/obj-riscv32-nemu-interpreter替换为$OBJ_DIR
:%s+\$NEMU_HOME/build/obj-riscv32-nemu-interpreter+$OBJ_DIR+g

# 将-c之前的内容替换为$CFLAGS
:%s/-O2.*=riscv32/$CFLAGS/g

# 将最后一行的空格替换成换行并缩进两格
:$s/  */\r  /g

```

以下是你列出的几条 Vim 编辑命令的解释及使用场景：

1. **只保留 `gcc` 或 `g++` 开头的行**：
    ```vim
    :%!grep "^\(gcc\|g++\)"
    ```
    这个命令使用 `grep` 工具筛选出当前文件中所有以 `gcc` 或 `g++` 开头的行.`%` 表示整个文件,`!` 表示执行外部命令,`grep "^\(gcc\|g++\)"` 用于匹配行首为 `gcc` 或 `g++` 的行.

2. **将环境变量 `$NEMU_HOME` 所指示字符串替换为 `$NEMU_HOME`**：
    ```vim
    :%!sed -e "s+$NEMU_HOME+\$NEMU_HOME+g"
    ```
    这个命令使用 `sed` 工具将文件中所有 `$NEMU_HOME` 的值替换为字符串 `\$NEMU_HOME`.`s+old+new+g` 是 `sed` 的替换命令,其中 `+` 是分隔符,可以根据需要选择合适的分隔符.

3. **将 `$NEMU_HOME/build/obj-riscv32-nemu-interpreter` 替换为 `$OBJ_DIR`**：
    ```vim
    :%s+\$NEMU_HOME/build/obj-riscv32-nemu-interpreter+$OBJ_DIR+g
    ```
    这个命令使用 Vim 的替换命令将文件中所有 `$NEMU_HOME/build/obj-riscv32-nemu-interpreter` 替换为 `$OBJ_DIR`.`%` 表示整个文件,`s` 是替换命令.

4. **将 `-c` 之前的内容替换为 `$CFLAGS`**：
    ```vim
    :%s/-O2.*=riscv32/$CFLAGS/g
    ```
    这个命令使用 Vim 的替换命令,将文件中 `-O2` 到 `=riscv32` 之间的内容替换为 `$CFLAGS`.`%` 表示整个文件,`s` 是替换命令.

5. **将最后一行的空格替换成换行并缩进两格**：
    ```vim
    :$s/  */\r  /g
    ```
    这个命令使用 Vim 的替换命令,将最后一行中的每个空格段替换为换行符并在新行开始处缩进两格.`$` 表示最后一行,`s` 是替换命令,`\r` 表示换行符,`  ` 表示两个空格.

6. **筛出来.c结尾的行**
    ```vim
    :%!grep "\.c$"
    ```
    这个命令使用 `grep` 工具筛选出当前文件中所有以 `.c` 结尾的行.`%` 表示整个文件,`!` 表示执行外部命令,`grep "\.c$"` 用于匹配行尾为 `.c` 的行.

`sed` 和 `grep` 是两个在 Unix/Linux 环境中广泛使用的文本处理工具.它们有不同的用途和功能,下面是对它们的详细介绍：

##### `grep`（Global Regular Expression Print）

`grep` 是一个用于搜索文本的工具,它通过匹配正则表达式来查找文本中的特定模式.`grep` 可以用于在文件中搜索特定的字符串或模式,并输出匹配的行.

**基本用法**：
```sh
grep [OPTIONS] PATTERN [FILE...]
```

**常用选项**：
- `-i`：忽略大小写.
- `-v`：反转匹配,显示不匹配的行.
- `-r`：递归搜索目录.
- `-l`：仅显示包含匹配行的文件名.
- `-n`：显示匹配行的行号.
- `-E`: 使用扩展正则表达式.
**示例**：
```sh
# 在文件example.txt中搜索包含字符串'pattern'的行
grep 'pattern' example.txt

# 在当前目录及其子目录中递归搜索包含字符串'pattern'的文件
grep -r 'pattern' .
```

##### `sed`（Stream Editor）

`sed` 是一个流编辑器,用于对文本进行过滤和转换.`sed` 可以对文本进行插入、删除、替换和其他操作,通常用于处理文件中的文本数据.

**基本用法**：
```sh
sed [OPTIONS] 'script' [FILE...]
```

**常用选项**：
- `-e script`：添加要执行的脚本.
- `-f script-file`：从脚本文件中读取命令.
- `-n`：抑制自动打印,只有在脚本中使用`p`命令时才打印行.
- `-i`：直接编辑文件,而不是输出到标准输出.

**常用命令**：
- `s/pattern/replacement/`：替换匹配的模式.
- `d`：删除行.
- `p`：打印行.
- `a\text`：在当前行之后插入文本.
- `i\text`：在当前行之前插入文本.

**示例**：
```sh
# 将文件example.txt中的所有'old'替换为'new'
sed 's/old/new/g' example.txt

# 删除文件example.txt中的第2行
sed '2d' example.txt

# 在文件example.txt的第3行后插入'this is a new line'
sed '3a\this is a new line' example.txt
```

### `grep` 和 `sed` 的联合使用

`grep` 和 `sed` 可以结合使用,以实现更强大的文本处理功能.例如,先使用 `grep` 筛选出特定的行,然后用 `sed` 对这些行进行进一步的处理.

**示例**：
```sh
# 先用grep找出包含'pattern'的行,再用sed将'old'替换为'new'
grep 'pattern' example.txt | sed 's/old/new/g'
```

通过 `grep` 和 `sed` 的组合,能够高效地进行复杂的文本处理任务.

#### :tabnew命令
`:tabnew` 命令用于在 Vim 编辑器中打开一个新的标签页.在 Vim 中,标签页是一种方便的多文件管理方式,可以在同一个 Vim 实例中同时编辑多个文件.

`:tabnew` 是 Vim 编辑器中的一个命令,用于在新标签页中打开文件.标签页允许用户在多个文件之间切换,同时保持每个文件的打开状态,从而提高多任务处理的效率.

##### 基本用法

```vim
:tabnew [filename]
```

- `:tabnew`：打开一个空的标签页.
- `:tabnew filename`：在一个新标签页中打开指定的文件.

##### 示例

1. **打开一个空标签页**：
    ```vim
    :tabnew
    ```

2. **在新标签页中打开文件**：
    ```vim
    :tabnew myfile.txt
    ```

##### 标签页操作

以下是一些常用的标签页操作命令：

- `:tabnext` 或 `:tabn`：切换到下一个标签页.
- `:tabprevious` 或 `:tabp`：切换到上一个标签页.
- `:tabfirst`：切换到第一个标签页.
- `:tablast`：切换到最后一个标签页.
- `:tabclose` 或 `:tabc`：关闭当前标签页.
- `:tabs`：列出所有打开的标签页.

##### 快捷键操作

Vim 还提供了快捷键来方便地操作标签页：

- `gt`：切换到下一个标签页.
- `gT`：切换到上一个标签页.
- `{N}gt`：切换到第 N 个标签页（例如,`3gt` 切换到第三个标签页）.

使用标签页功能,可以在 Vim 中更加高效地管理和编辑多个文件.

#### #功能
`#` 是 Vim 编辑器中的一个快捷键,用于在当前文件中查找光标所在位置的单词,并跳转到下一个匹配的位置.这个功能通常用于快速定位和查找文本中的关键字.

`xargs` 是一个非常有用的 Unix 命令,用于将输入转换为命令行参数.它允许你将标准输入的数据传递给其他命令,特别适用于处理长列表的文件或其他项目.下面是对 `xargs` 的详细介绍：

#### xargs基本语法
```bash
xargs [options] [command [initial-arguments]]
```

#### 主要功能
`xargs` 读取标准输入并将其分割成一组参数,然后将这些参数传递给指定的命令.

#### 主要选项
- `-0`：输入项目以空字符（null character）分隔,常用于与 `find -print0` 搭配使用.
- `-d delimiter`：自定义定界符,分割输入项目.
- `-I replace-str`：定义替换字符串,指定每次替换输入中的每个参数.
- `-n number`：指定每次传递给命令的参数数目.
- `-P max-procs`：指定同时运行的最大进程数（并行执行）.
- `-L max-lines`：指定每次读取的输入行数.

#### 使用示例

1. **简单示例**：
    ```bash
    echo "file1.txt file2.txt file3.txt" | xargs rm
    ```
    这个命令会删除 `file1.txt`、`file2.txt` 和 `file3.txt`.

2. **结合 `find` 使用**：
    ```bash
    find . -name "*.log" | xargs rm
    ```
    这个命令会找到当前目录及其子目录下的所有 `.log` 文件并删除它们.

3. **处理带空格的文件名**：
    ```bash
    find . -name "*.log" -print0 | xargs -0 rm
    ```
    `-print0` 选项使 `find` 输出以 null 字符分隔的文件名,`xargs -0` 能正确处理这些文件名,即使它们包含空格或换行符.

4. **限制每次命令行的参数数量**：
    ```bash
    find . -name "*.txt" | xargs -n 5 cp -t /backup
    ```
    这个命令每次会传递 5 个文件给 `cp` 命令,将它们复制到 `/backup` 目录.

5. **并行执行**：
    ```bash
    find . -name "*.log" | xargs -P 4 gzip
    ```
    这个命令会使用最多 4 个并行进程来压缩找到的 `.log` 文件.

6. **使用替换字符串**：
    ```bash
    echo "file1 file2 file3" | xargs -I {} mv {} /new/location
    ```
    这个命令会将 `file1`、`file2` 和 `file3` 移动到 `/new/location` 目录,`{}` 会被每个文件名替换.

#### 总结
`xargs` 是一个强大的工具,能显著提高处理批量文件或参数的效率.通过将标准输入转换为命令行参数,它在许多脚本和自动化任务中扮演着重要角色.

### 取除
在正则表达式中,符号 `^` 有两个主要作用,具体取决于它所在的位置：

#### 1. 行首匹配符

当 `^` 位于正则表达式的开头时,它表示匹配行的开头.也就是说,`^pattern` 只会匹配那些以 `pattern` 开头的行.

示例

- **匹配以 "hello" 开头的行**：
  ```bash
  grep "^hello" file.txt
  ```
  例如,文件 `file.txt` 中有以下内容：
  ```
  hello world
  say hello
  hello there
  ```
  只有第一行和第三行会被匹配.

- **匹配以数字开头的行**：
  ```bash
  grep "^[0-9]" file.txt
  ```
  例如,文件 `file.txt` 中有以下内容：
  ```
  1. This is a test.
  This is another test.
  42 is the answer.
  ```
  只有第一行和第三行会被匹配.

#### 2. 否定字符集

当 `^` 位于方括号 `[]` 中的开头时,它表示否定字符集.也就是说,`[^abc]` 表示匹配任何不在 `abc` 之内的字符.


- **匹配不包含元音字母的字符**：
  ```bash
  grep "[^aeiou]" file.txt
  ```
  例如,文件 `file.txt` 中有以下内容：
  ```
  apple
  banana
  cherry
  ```
  每一行都会匹配,因为每行都有至少一个非元音字母.

- **匹配不包含数字的字符**：
  ```bash
  grep "[^0-9]" file.txt
  ```
  例如,文件 `file.txt` 中有以下内容：
  ```
  1234
  abcd
  a1b2c3
  ```
  第二行和第三行会被匹配,因为它们包含非数字字符.


- `^pattern`：匹配行的开头.
- `[^chars]`：匹配不在 `chars` 内的任意字符.

这些规则使得 `^` 在正则表达式中非常有用,可以用于匹配行的特定位置或排除特定字符集.

#### nemu中的grep

```shell
find . -name "*.[ch]" | grep -E "^\./(src|include)" | \
  grep -E -v "^\./include/config" | grep -v "mips32\|riscv64\|loongarch32r" | wc
find . -name "*.[ch]" | grep -E "^\./(src|include)" | \
  grep -E -v "^\./include/config" | grep -v "mips32\|riscv64\|loongarch32r" | xargs wc

// wc (print newline,word,and the byte counts for each file)
```
### SEXT宏的作用
`SEXT` 宏的作用是将一个有符号数的低位扩展为更高位,以保持其符号位不变.在计算机系统中,有符号数通常使用补码表示,即最高位为符号位,0 表示正数,1 表示负数.当需要将一个低位有符号数扩展为更高位时,需要保持符号位不变,即将符号位复制到更高位.(copilot说的)

其实这些值存在内存里都是32位,只是在显示的时候显示成若干位,所以需要进行符号扩展.

### PA2.2 
### 链接和加载

GCC 提供了许多选项来控制编译过程,每个选项都有特定的功能,许多选项的缩写也直观地反映了它们的功能.以下是一些常见的 GCC 选项及其功能与缩写的关系：

1. **`-o` (output)**：
   - 功能：指定输出文件的名称.
   - 缩写来源：`o` 代表“output”.
   - 示例：`gcc -o my_program foo.c` 将编译 `foo.c` 并将输出的可执行文件命名为 `my_program`.

2. **`-I` (include)**：
   - 功能：指定额外的头文件搜索路径.
   - 缩写来源：`I` 代表“include”.
   - 示例：`gcc -I/my/include/path foo.c` 将在 `/my/include/path` 中搜索头文件.

3. **`-L` (library path)**：
   - 功能：指定额外的库文件搜索路径.
   - 缩写来源：`L` 代表“library path”.
   - 示例：`gcc -L/my/library/path foo.c` 将在 `/my/library/path` 中搜索库文件.

4. **`-l` (library)**：
   - 功能：指定链接时要使用的库.
   - 缩写来源：`l` 代表“library”.
   - 示例：`gcc -o my_program foo.o -lm` 将链接数学库（libm）.

5. **`-g` (debug)**：
   - 功能：生成调试信息,用于调试器（如 GDB）.
   - 缩写来源：`g` 代表“debug”.
   - 示例：`gcc -g foo.c` 将编译 `foo.c` 并生成调试信息.

6. **`-O` (optimize)**：
   - 功能：指定优化级别（如 `-O2`）.
   - 缩写来源：`O` 代表“optimize”.
   - 示例：`gcc -O2 foo.c` 将以优化级别 2 编译 `foo.c`.

7. **`-D` (define)**：
   - 功能：定义预处理宏.
   - 缩写来源：`D` 代表“define”.
   - 示例：`gcc -DDEBUG foo.c` 将在编译 `foo.c` 时定义 `DEBUG` 宏.

8. **`-E` (preprocess)**：
   - 功能：只执行预处理步骤,不进行编译.
   - 缩写来源：`E` 代表“preprocess”.
   - 示例：`gcc -E foo.c` 将预处理 `foo.c` 并输出结果.

9. **`-S` (assembly)**：
   - 功能：生成汇编代码,而不进行汇编和链接.
   - 缩写来源：`S` 代表“assembly”.
   - 示例：`gcc -S foo.c` 将编译 `foo.c` 并生成汇编代码文件 `foo.s`.

10. **`-Wall` (all warnings)**：
    - 功能：启用所有常见的警告.
    - 缩写来源：`Wall` 代表“all warnings”.
    - 示例：`gcc -Wall foo.c` 将编译 `foo.c` 并启用所有常见的警告.

11. ** `-c` (compile)**:
    - 功能：编译源文件,但不进行链接.
    - 缩写来源：`c` 代表“compile”.
    - 示例：`gcc -c foo.c` 将编译 `foo.c` 生成目标文件 `foo.o`.

这些选项提供了对编译过程的细粒度控制,帮助开发者优化和调试他们的程序.

12. **`-ggdb` (GDB debug)**：
    - 功能：生成供 GDB 使用的详细调试信息.
    - 缩写来源：`ggdb` 代表“GDB debug”.
    - 示例：`gcc -ggdb foo.c` 将编译 `foo.c` 并生成详细的调试信息.

 主要选项解释
- **-g**: 生成基本的调试信息。
- **-ggdb**: 生成供 GDB 使用的调试信息，并且比 `-g` 更详细。包含更多的调试信息，比如 GNU 扩展。

假设你有一个源文件 `example.c`，你可以使用以下命令编译它：

```bash
gcc -ggdb -o example example.c
```

这会生成一个名为 `example` 的可执行文件，并包含 GDB 所需的详细调试信息。

优点
- **更详细的调试信息**: 提供了更丰富的调试信息，便于使用 GDB 进行复杂的调试。
- **适用于 GDB**: 特别适合与 GDB 搭配使用，可以更好地利用 GDB 的高级调试功能。

缺点
- **较大的二进制文件**: 由于包含了更多的调试信息，生成的可执行文件会比只使用 `-g` 更大。
- **可能会影响性能**: 虽然调试信息不会影响运行时性能，但在某些情况下，包含调试信息的文件可能会导致编译和链接时间增加。

调试示例
编译后，可以使用 GDB 调试生成的可执行文件：

```bash
gdb example
```

在 GDB 中，你可以设置断点、检查变量、单步执行代码等，这些调试操作都会受益于详细的调试信息。

使用 `-ggdb` 可以大大提高调试的便利性和效率，尤其是在处理复杂的代码或调试难以复现的错误时。
### coreutils 和 binutils(GNU工具链)
`coreutils` 和 `binutils` 是 GNU 工具集中的两个重要组件,分别提供了核心工具和二进制工具的功能.以下是对这两个工具集的简要介绍：

#### coreutils
