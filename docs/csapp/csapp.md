# CSAPP attack lab
目前很想知道的是，寄存器的名字的英文全称
前六个寄存器(%rax, %rbx, %rcx, %rdx, %rsi, %rdi)称为通用寄存器，有其『特定』的用途：

%rax(%eax) 用于做累加(add)
%rcx(%ecx) 用于计数(count)
%rdx(%edx) 用于保存数据(data)
%rbx(%ebx) 用于做内存查找的基础地址(base)
%rsi(%esi) 用于保存源索引值(source)
%rdi(%edi) 用于保存目标索引值(destination)

shl(shift logical left)
shr(shift logical right)
逻辑右移
sar(shift arithmetic left)
leaq(load effective address)
inc(increament)
dec(decreament)

需要两个操作数的指令

+ addq Src, Dest -> Dest = Dest + Src
+ subq Src, Dest -> Dest = Dest - Src
+ imulq Src, Dest -> Dest = Dest * Src
+ salq Src, Dest -> Dest = Dest << Src
+ sarq Src, Dest -> Dest = Dest >> Src
+ shrq Src, Dest -> Dest = Dest >> Src
+ xorq Src, Dest -> Dest = Dest ^ Src
+ andq Src, Dest -> Dest = Dest & Src
+ orq Src, Dest -> Dest = Dest | Src

## 流程控制

``` c title="在小土刀的博客中复制来的"
// Do While 的 C 语言代码
long pcount_do(unsigned long x)
{
    long result = 0;
    do {
        result += x & 0x1;
        x >>= 1;
    } while (x);
    return result;
}

// Goto 版本
long pcount_goto(unsigned long x)
{
    long result = 0;
loop:
    result += x & 0x1;
    x >>= 1;
    if (x) goto loop;
    return result;
}
// ctrl+shift+[ 折叠光标所处的代码块
```

该函数计算参数x中有多少位是1(突然想到有许多方法可以优化该函数)<https://zhuanlan.zhihu.com/p/341488123>
　　sparse_popcnt 对 iterated_popcnt 做了改进，每次迭代总是将最右边的非零位置零。这是减法的妙用。试想一下，一个仅最高位为1的整数，用此方法的话仅需一次迭代；而 iterated_popcnt 还是会“乖乖地”迭代 32 次。


=== "iterated_popcnt"

    ``` c 
        int iterated_popcnt(uint32_t n)
        {
            int count = 0;
            for(; n; n >>= 1)
                count += n & 1U;
            return count;
        }
    ```

=== "sparse_popcnt"

    ``` c 
    int sparse_popcnt(uint32_t n)
    {
        int count = 0;
        while(n)
        {
            ++count;
            n &= n - 1;
        }
        return count;
    }
    ```
### 该函数的汇编代码
```  x86asm 
        movl    $0, %eax    # result = 0
.L2:                    # loop:
    movq    %rdi, %rdx
    andl    $1, %edx    # t = x & 0x1
    addq    %rdx, %rax  # result += t
    shrq    %rdi        # x >>= 1
    jne     .L2         # if (x) goto loop
    rep; ret
```

### while 和do-while
先来看看并不那么常用的 Do-While 语句以及对应使用 goto 语句进行跳转的版本：
``` c
// Do While 的 C 语言代码
long pcount_do(unsigned long x)
{
    long result = 0;
    do {
        result += x & 0x1;
        x >>= 1;
    } while (x);
    return result;
}

// Goto 版本
long pcount_goto(unsigned long x)
{
    long result = 0;
loop:
    result += x & 0x1;
    x >>= 1;
    if (x) goto loop;
    return result;
}
```

而对于 While 语句的转换，会直接跳到中间，如：
``` c
// C While version
while (Test)
	Body

// Goto Version
	goto test;
loop:
	Body
test:
	if (Test)
		goto loop;
done:
```
如果在编译器中开启 -O1 优化，那么会把 While 先翻译成 Do-While，然后再转换成对应的 Goto 版本，因为 Do-While 语句执行起来更快，更符合 CPU 的运算模型。



