# risc-v
什么是risc-v?reduced instruction sets computer - v(指瑞克五代)

1. 五个阶段分别是如何实现的?

## 取指(Instruction Fetch, IF)

`isa_exec_once()`做的第一件事情就是取指令. 在 `NEMU` 中, 有一个函数inst_fetch()(在`nemu/include/cpu/ifetch.h`中定义)专门负责取指令的工作. inst_fetch()最终会根据参数`len`来调用`vaddr_ifetch()`(在`nemu/src/memory/vaddr.c`中定义), 而目前`vaddr_ifetch()`又会通过`paddr_read()`来访问物理内存中的内容. 因此, 取指操作的本质只不过就是一次内存的访问而已.
```c
int isa_exec_once(Decode *s) {
  s->isa.inst.val = inst_fetch(&s->snpc, 4);
  return decode_exec(s);
}
```
这个4倒是让我觉得可以,因为nemu架构分了几个不同的文件,这样也好,因为magic macro整太多了,看的也脑袋大,直接分成不同的文件也好.(尚未搞明白具体如何编译的,估计是通过makefile来根据架构选择不同文件来进行编译)
设计的精妙在掌握了魔法宏后可以大概掌握


```c
typedef concat(__GUEST_ISA__, _ISADecodeInfo) ISADecodeInfo;

typedef struct {
  union {
    uint32_t val;
  } inst;
} MUXDEF(CONFIG_RV64, riscv64_ISADecodeInfo, riscv32_ISADecodeInfo);
```
这个宏的作用是根据`__GUEST_ISA__`的值来选择不同的结构体定义. 例如, 当`__GUEST_ISA__`的值为`riscv64`时, `MUXDEF`会展开为`riscv64_ISADecodeInfo`, 而当`__GUEST_ISA__`的值为`riscv32`时, `MUXDEF`会展开为`riscv32_ISADecodeInfo`. 这样, 我们就可以通过`__GUEST_ISA__`的值来选择不同的结构体定义, 从而实现了不同指令集的支持.

### inst_fetch

fetch(在计算机科学中，"fetch"通常指的是从内存中获取数据的过程.在处理器的指令周期中，"fetch"阶段是指从内存中获取指令的过程)

在你给出的代码中，`inst_fetch`函数的作用就是从虚拟地址`pc`指向的位置获取长度为`len`的指令，并将`pc`向前移动`len`个单位.这个函数是模拟CPU的指令获取阶段的一部分.

具体来说，`inst_fetch`函数做了以下几件事：

1. 调用`vaddr_ifetch`函数从虚拟地址`pc`获取长度为`len`的指令.
2. 将`pc`向前移动`len`个单位，以便下一次获取指令时能获取到正确的指令.
3. 返回获取到的指令.

这个函数的返回值是获取到的指令，它是一个`uint32_t`类型的值.
```c
#include <memory/vaddr.h>

static inline uint32_t inst_fetch(vaddr_t *pc, int len) {
  uint32_t inst = vaddr_ifetch(*pc, len);
  (*pc) += len;
  return inst;
}

static int decode_exec(Decode *s) {
  int rd = 0;
  word_t src1 = 0, src2 = 0, imm = 0;
  s->dnpc = s->snpc;
```

## 译码(instruction decode, ID)
接下来代码会进入decode_exec()函数, 它首先进行的是译码相关的操作. 译码的目的是得到指令的操作和操作对象, 这主要是通过查看指令的opcode来决定的. 不同ISA的opcode会出现在指令的不同位置, 我们只需要根据指令的编码格式, 从取出的指令中识别出相应的opcode即可.

和YEMU相比, NEMU使用一种抽象层次更高的译码方式: 模式匹配, NEMU可以通过一个模式字符串来指定指令中opcode, 例如在riscv32中有如下模式:

```c
static int decode_exec(Decode *s) {
  int rd = 0;
  word_t src1 = 0, src2 = 0, imm = 0;
  s->dnpc = s->snpc;

#define INSTPAT_INST(s) ((s)->isa.inst.val)
#define INSTPAT_MATCH(s, name, type, ... /* execute body */ ) { \
  decode_operand(s, &rd, &src1, &src2, &imm, concat(TYPE_, type)); \
  __VA_ARGS__ ; \
}
```

## gcc
-E -S -c -o file -g -v
-nB

## RTL实现处理器

### ALU如何实现



### varilator仿真
好的，我可以为您介绍如何编写 Verilator 的 `main.cpp` 文件。

`main.cpp` 是 Verilator 生成的 C++ 代码的入口点，用于模拟 Verilog 模块的行为。 它通常包含以下几个部分：

**1.  包含头文件：**

```c++
#include "Vyour_module.h"   // Verilator 生成的头文件
#include "verilated.h"
#include "verilated_vcd_c.h" // 用于生成波形文件 (可选)
```

*  `Vyour_module.h` 是 Verilator 为您的 Verilog 模块生成的 C++ 头文件。 将 `your_module` 替换为您实际的模块名称。
*  `verilated.h` 包含 Verilator 的核心函数和类。
*  `verilated_vcd_c.h` 包含用于生成 VCD 波形文件的函数和类 (可选)。

**2.  创建顶层模块实例：**

```c++
Vyour_module* top = new Vyour_module;
```

*  `Vyour_module` 是 Verilator 生成的顶层模块类。
*  `top` 是指向顶层模块实例的指针。

**3.  初始化和仿真循环：**

```c++
vluint64_t main_time = 0; // 当前仿真时间

// 初始化
top->reset = 1;
top->clk = 0;
top->eval(); // 评估初始状态

// 可选：创建波形文件
VerilatedVcdC* tfp = new VerilatedVcdC;
top->trace(tfp, 99); // 跟踪所有信号
tfp->open("your_module.vcd"); // 打开波形文件

// 仿真循环
while (!Verilated::gotFinish()) {
  main_time++;

  // 生成时钟信号
  if (main_time % 2 == 0) {
    top->clk = !top->clk;
  }

  // 应用输入信号
  // ...

  // 评估电路
  top->eval();

  // 可选：记录波形数据
  if (tfp) {
    tfp->dump(main_time);
  }
}

// 清理
if (tfp) {
  tfp->close();
}
top->final();
delete top;
```

*  `main_time` 用于跟踪当前仿真时间。
*  初始化阶段将 `reset` 信号设置为 1，然后评估电路的初始状态。
*  仿真循环中，您可以生成时钟信号，应用输入信号，并评估电路的行为。
*  您可以使用 `VerilatedVcdC` 类生成 VCD 波形文件，以便在仿真后进行分析。

**4.  完整示例 (假设您的模块名为 `my_module`)：**

```c++
#include "Vmy_module.h"
#include "verilated.h"
#include "verilated_vcd_c.h"

int main(int argc, char** argv, char** env) {
  Verilated::commandArgs(argc, argv);

  Vmy_module* top = new Vmy_module;

  vluint64_t main_time = 0;

  // 初始化
  top->reset = 1;
  top->clk = 0;
  top->eval();

  // 创建波形文件 (可选)
  VerilatedVcdC* tfp = new VerilatedVcdC;
  top->trace(tfp, 99);
  tfp->open("my_module.vcd");

  // 仿真循环
  while (!Verilated::gotFinish()) {
    main_time++;

    if (main_time % 2 == 0) {
      top->clk = !top->clk;
    }

    // 应用输入信号
    // ...

    top->eval();

    if (tfp) {
      tfp->dump(main_time);
    }
  }

  // 清理
  if (tfp) {
    tfp->close();
  }
  top->final();
  delete top;

  return 0;
}
```

**编译和运行：**

1.  使用 Verilator 编译您的 Verilog 模块和 `main.cpp` 文件。
2.  运行生成的可执行文件进行仿真。

**请记住，这只是一个基本的示例，您需要根据您的具体 Verilog 模块和仿真需求进行修改。**

希望这些信息对您有所帮助！


