# gdb 的原理
Linux 下常用的程序调试器 gdb 是什么原理？ - MeteorZ的回答 - 知乎
https://www.zhihu.com/question/578172542/answer/3389041105

ptrace(),


## 查看寄存器的值
我对寄存器值的支持还是有问题,原因是regs的寄存器名字变了相应的匹配规则也要变,有点小丑,因为之前尝试用宏来支持多平台,现在还是得根据架构来变这个寄存器名字匹配的正则规则.

### 监视点
``` shell
make: *** [/home/along/ysyx-workbench/nemu/scripts/native.mk:38: run] Segmentation fault (core dumped)
```
很难蚌的住的是,这些指令
