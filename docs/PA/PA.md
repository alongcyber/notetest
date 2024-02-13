整个实验逐步引导完成一个计算机系统的构建，包括底层的 NEMU 模拟器，运行时环境 AbstractMachine（AM），在其上的简易操作系统 NanOS-lite，以及操作系统上的应用程序库 Navy-apps。

一共分为 5 个部分，PA0 配置环境，PA1 完善 NEMU 的调试器功能，PA2 模拟 NEMU 指令运行以及补充 AM，PA3 完善 NEMU 的中断 / 异常处理、实现操作系统的系统调用以及简易文件系统功能，PA4 在操作系统中实现多道程序的运行、虚拟内存管理以及外部中断处理。

细分每个实验的内容以及涉及到的源码部分，我整理了如下一个表格供参考：

PS：其中 AM 由五个部分组成，TRM（图灵机模拟）简单而且已经实现好了，IOE 为输入输出扩展，CTE 为上下文管理扩展，VME 为虚拟内存管理扩展，还有一个 MPE（多处理器扩展）在 PA 中不使用。klib 为简单的运行时库，提供给 AM 和操作系统使用。

总而言之有了计算机系统课以及 RISCV 手册阅读的基础，大部分实验还是比较轻松的，没有什么难以理解的地方。较为痛苦的就是 debug 时一些奇怪的问题以及最后 PA4 中之前没有接触过的虚存管理的内容了。后面记录一些还能回忆起来的问题（其实是做的过程中懒得记录）。

## macro magic

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
这段代码是C++中的宏定义，用于在编译时期生成代码。让我们逐行解析：

1. `#include <iostream>` 和 `#include <cstring>` 是包含标准库的头文件，分别提供了输入/输出功能和字符串处理功能。

2. `#define TRACE_STRINGIFY(item) "" #item` 是一个宏定义，用于将输入的参数转换为字符串。`#`操作符在宏定义中被称为"字符串化"操作符，它会将后面的参数转换为字符串。

那么为什么#item前面要加引号呢“”？

在C语言的宏定义中，`#`操作符被称为"字符串化"操作符，它会将后面的宏参数转换为字符串。但是，如果直接使用`#item`，并且`item`是一个宏，那么预处理器会先将`item`转换为它所代表的值，然后再将结果转换为字符串。

例如，假设我们有以下的宏定义：

```c
#define VALUE 10
#define TO_STRING(item) #item
```

如果我们使用`TO_STRING(VALUE)`，预处理器会先将`VALUE`替换为`10`，然后将`10`转换为字符串，得到`"10"`。

但是，如果我们想得到字符串`"VALUE"`而不是`"10"`，我们就需要阻止预处理器对`VALUE`的宏扩展。这就是为什么我们在`#item`前面加上`""`的原因。`"" #item`会先将`item`转换为字符串，然后与前面的空字符串连接，得到的结果就是我们想要的`"VALUE"`。

所以，`#define TRACE_STRINGIFY(item) "" #item`这个宏定义的作用是将输入的参数转换为字符串，同时阻止预处理器对参数进行宏扩展。

3. `#define TRACE(macro, message)` 定义了一个名为`TRACE`的宏，它接受两个参数：`macro`和`message`。

4. `do { ... } while (0)` 是一个常见的宏技巧，用于确保宏的使用方式和函数调用一样，即使在需要使用分号结束的地方也不会出错。`do { ... } while (0)` 是一个常见的宏定义技巧，主要是为了解决宏扩展中可能出现的问题。

在C/C++中，宏是在预处理阶段进行文本替换的，它并不像函数那样有自己的作用域。如果一个宏定义包含多条语句，而在使用这个宏的地方没有正确地用括号包围它，就可能导致预期之外的行为。

例如，考虑以下宏定义和使用：

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

这并不是我们想要的结果，因为`bar(y);`总是会被执行，而不管`condition`的值是什么。

如果我们使用`do { ... } while (0)`，就可以避免这个问题：

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

这样就可以得到我们期望的行为：只有当`condition`为真时，`foo(y)`和`bar(y)`才会被执行。


5. `if (strcmp("" #macro, TRACE_STRINGIFY(macro)))` 这行代码比较复杂。首先，`"" #macro`将`macro`参数字符串化。然后，`TRACE_STRINGIFY(macro)`也将`macro`参数字符串化。`strcmp`函数比较这两个字符串，如果它们相等，返回0，否则返回非0值。因此，这个`if`语句的条件是"如果`macro`参数字符串化后的结果和自身不相等"。

6. `std::cout << message << "\n";` 如果上述`if`条件为真，那么就输出`message`参数，并在其后添加一个换行符。

这个宏的目的是检查`macro`参数是否已经被定义为一个宏。如果`macro`已经被定义为一个宏，那么`TRACE_STRINGIFY(macro)`将返回该宏的值，而不是`macro`本身，因此`strcmp`的结果将不为0，从而触发消息的输出。
The "" # macro expands to the macro name as a string, whereas TRACE_STRINGIFY(macro) first expands the macro, then stringifies the result. If the two differ, macro has to be a preprocessor macro.
"" # macro 将宏名扩展为字符串，而 TRACE_STRINGIFY(macro) 首先扩展宏，然后将结果字符串化。如果两者不同，则 macro 必须是预处理器宏。

This approach does fail for macros that are defined to themselves, i.e. #define FOO FOO. Such macros are not detected as preprocessor macros.
这种方法对于定义为自身的宏（即 #define FOO FOO ）确实失败。这样的宏不会被检测为预处理器宏。

Most compilers should be able to completely optimize away the comparison of two string literals. GNU GCC (g++) 4.8.2 definitely does even with -O0 (as does gcc for C -- the same approach obviously works in C, too).
大多数编译器应该能够完全优化两个字符串的比较。GNU GCC（g++）4.8.2甚至对 -O0 也是如此（gcc对C也是如此--同样的方法显然也适用于C）。

This approach does work for function-like macros, but only if you retain the parentheses (and proper number of commas, if the macro takes multiple parameters) and it is not defined to itself (e.g. #define BAR(x) BAR(x)).
这种方法确实适用于类似函数的宏，但前提是您保留括号（以及适当数量的逗号，如果宏接受多个参数）并且它没有定义为自身（例如 #define BAR(x) BAR(x) ）。

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
换句话说， TEST1 符号不会被识别为已定义（因为它是对自身定义的），没有括号的 TEST11 或 TEST13 也不会被识别为已定义。这些是这种方法的局限性。

Using the parenthesized form works for all parameterless macros (except TEST1, ie. those defined to themselves), and all single-parameter macros. If a macro expects multiple parameters, you need to use the correct number of commas, as otherwise (say, if you tried TRACE(TEST13(), "...")) you get a compile-time error: "macro TEST13 requires 3 arguments, only 1 given" or similar.
使用括号形式适用于所有无参数宏（除了 TEST1 ，即。它们是自己定义的），以及所有单参数宏。如果一个宏需要多个参数，你需要使用正确数量的逗号，否则（比如说，如果你尝试 TRACE(TEST13(), "...") ）你会得到一个编译时错误：“宏 TEST13 需要3个参数，只有1个给定”或类似的。

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

GNU的`readline`库提供了一种方便的方式来处理用户的输入，包括行编辑，历史记录和自动补全。以下是一些基本的使用方法：

1. **初始化readline**：在使用readline之前，你需要先初始化它。这通常在程序开始时完成：

    ```c
    #include <readline/readline.h>
    #include <readline/history.h>
    ```

2. **读取输入**：你可以使用`readline`函数来读取用户的输入。这个函数接受一个提示字符串作为参数，并返回用户输入的字符串。例如：

    ```c
    char* input = readline("Enter your command: ");
    ```

    用户可以使用readline的行编辑功能来修改他们的输入。当他们按下Enter键时，`readline`函数将返回他们输入的字符串。

3. **处理历史记录**：你可以使用`add_history`函数来添加一个命令到历史记录中：

    ```c
    add_history(input);
    ```

    用户可以使用上下箭头键来浏览历史记录。

4. **自动补全**：你可以使用`rl_completion_entry_function`和`rl_attempted_completion_function`来定制自动补全的行为。例如，你可以设置一个函数来提供可能的补全，或者改变补全的规则。

5. **清理**：当你完成了readline的使用，你应该释放它返回的字符串：

    ```c
    free(input);
    ```

以上就是GNU readline的基本用法。请注意，readline库的功能非常强大，这里只是介绍了一些基本的用法。如果你想了解更多，你可以查阅readline的官方文档。

#### 举例说明readline
以下是一个使用GNU readline库的简单C程序示例。这个程序会提示用户输入命令，然后将命令添加到历史记录中，并打印出来。用户可以使用readline的行编辑和历史功能。

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

在这个程序中，我们使用`readline`函数读取用户的输入。我们传递一个提示字符串"Enter command: "给`readline`函数，然后它会返回用户输入的字符串。(而且readline会打印提示字符串)

我们检查返回的字符串是否为NULL，如果是NULL，那么我们就退出循环。否则，我们就打印出用户输入的字符串，并使用`add_history`函数将它添加到历史记录中。

最后，我们使用`free`函数释放`readline`函数返回的字符串。

要编译这个程序，你需要链接readline库。在gcc中，你可以使用`-lreadline`选项来做到这一点：

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
`strtok`是一个C语言库函数，用于在字符串中查找并分割出一系列的标记（tokens）。它的原型如下：

```c
char *strtok(char *str, const char *delim);
```

这个函数接受两个参数：一个是要分割的字符串`str`，另一个是包含分隔符的字符串`delim`。函数会在`str`中查找`delim`中的任何字符，并将其作为标记的结束。然后，它会在`str`中插入一个null字符来结束这个标记，并返回一个指向这个标记的指针。

在连续的调用中，你可以传递NULL作为第一个参数，`strtok`会继续在上次的字符串上进行操作，查找下一个标记。

例如，以下的代码会分割一个由空格分隔的字符串：

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

请注意，`strtok`函数会修改它的输入字符串，所以如果你需要保留原始字符串，你应该在调用`strtok`之前创建一个副本。另外，`strtok`函数不是线程安全的，如果你在多线程环境中需要分割字符串，你应该使用`strtok_r`函数．