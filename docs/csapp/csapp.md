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

