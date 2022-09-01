# Calling Convention

**LEC 5 (TAs):** [GDB, calling conventions and stack frames RISC-V](https://pdos.csail.mit.edu/6.828/2021/lec/gdb_slides.pdf) (2020: [notes](https://pdos.csail.mit.edu/6.828/2021/lec/l-riscv.txt), [boards](https://pdos.csail.mit.edu/6.828/2021/lec/l-riscv-slides.pdf), [video](https://youtu.be/s-Z5t_yTyTM))
**Preparation**: Read [Calling Convention](https://pdos.csail.mit.edu/6.828/2021/readings/riscv-calling.pdf)
**Assignment**: [Lab pgtbl: Page tables](https://pdos.csail.mit.edu/6.828/2021/labs/pgtbl.html)

目标：熟悉RISC-V处理器、汇编语言、以及RISC-V的calling convention

## C程序到汇编程序的转换

- 任何一个处理器都有一个关联的指令集（Instruction Sets Architecture），每个指令都有对应的二进制编码或者一个Opcode。当处理器运行时，看到这些编码就知道该做什么操作。
- 处理器理解二进制文件
  - C -> Asm -> Binary(object / .ofiles)

![image-20220419142920254](3-CallingConvention.assets/image-20220419142920254.png)

## RISC-V vs. X86

- 不同处理器对应的汇编语言不同
  - RISC-V汇编
- RISC & CISC应用
  - RISC-V处理器（嵌入式设备）、ARM处理器（安卓、Apple）：**精简指令集（RISC）**
  - 大部分PC处理器为x86（Intel、AMD）：**复杂指令集（CISC）**

- RISC vs. CISC
  - 指令的数量（RISC有更少的指令数量）2-8原则
  - RISC指令更简单，趋于完成更简单的工作
  - RISC是唯一一款开源指令集（任何人都可以为RISC-V开发主板）

## gdb和汇编代码执行

![image-20220421192605982](3-CallingConvention.assets/image-20220421192605982.png)

- global表示你可以在其他文件中调用这个函数。text表明这里的是代码

## RISC-V寄存器（用户寄存器）

![image-20220422140832955](3-CallingConvention.assets/image-20220422140832955.png)

- 寄存器关注其ABI名字（汇编代码中用的就是ABI名字）

- RISC-V寄存器

  - 所有寄存器都是64bit；各种数据类型都会被改造放进这64bit中。
    - 比如一个32bit整数，取决于是否有符号，会在前面补32个0或1存在寄存器中。
  - 寄存器是位于CPU上的，预先定义的可以用来存储数据的位置。
  - 汇编代码并不是在内存中运行，而是在寄存器中运行（寄存器最快）。
  - 使用寄存器的方法：

  ![image-20220422141159831](3-CallingConvention.assets/image-20220422141159831.png)

- 函数用于传递参数的寄存器（若超过8个参数，其余用内存来传参）：

  - a0-a7：8个整数寄存器
  - fa0-fa7：8个floating-point寄存器

- 函数返回值：

  - a0-a1
  - fa0-fa1


![image-20220829113855684](C:\Users\11354\AppData\Roaming\Typora\typora-user-images\image-20220829113855684.png)

- 存储者(Saved)：寄存器是全局共用的，但是为了软件的正确性和协同工作，对寄存器进行了Caller和Callee的区分，所有代码都需要遵守以保证正确性。
  - **Caller**-Saved Reg（volatile reg）：函数调用的时候不会保存（Not preseved across fn call），因此需要调用者（Caller）压栈保存
    - 用于保存不需要在各个调用之间保留的临时数据；如果要在过程调用后恢复该值，则调用方有责任将这些寄存器压入堆栈或将其复制到其他位置
    - 一个Caller Saved寄存器可能被其他函数重写。假设我们在函数a中调用函数b，任何被函数a使用的并且是Caller Saved寄存器，调用函数b可能重写这些寄存器。例子: Return address寄存器（注，保存的是函数返回的地址），ra寄存器是Caller Saved，这一点很重要，它导致了当函数a调用函数b的时侯，b会重写Return address。所以调用者需要将ra的值压栈存储
  - **Callee**-Saved Reg（non-volatile reg）：函数调用的时候会保存（Preserved across fn call ），作为被调用方的函数要小心寄存器的值不会相应的变化。![image-20220422142555848](3-CallingConvention.assets/image-20220422142555848.png)

![image-20220829113328432](C:\Users\11354\AppData\Roaming\Typora\typora-user-images\image-20220829113328432.png)

## Stack + Calling Convention

栈：使函数有组织，并且能正常返回

![image-20220422142906320](3-CallingConvention.assets/image-20220422142906320.png)

- 每个区域为一个Stack Frame，每执行一次函数调用就会产生一个Stack Frame。

  - 函数通过移动Stack Pointer来完成Stack Frame的空间分配。

- Stack是从高地址向低地址使用，所有栈总是向下增长。

  - 创建Stack Frame时，对当前Stack Pointer做减法。

- Stack Frame中的两个重要寄存器

  - Stack Pointer：指向Stack底部，代表了当前Stack Frame的位置
  - Frame Pointer：指向当前Stack Frame的顶部
    - 可通过FP得到Return address & 指向前一个Stack Frame的指针

- Stack Frame结构：

  - （固定位置）返回地址、指向前一个Stack Frame的指针（当前函数返回时，将此存到FP寄存器中）
  - 保存的寄存器、本地变量、当函数参数多于8个时，额外的参数会出现在Stack中

- Stack Frame必须要被汇编代码创建，所以是编译器生成了汇编代码，进而创建了Stack Frame。汇编函数结构：

  ![image-20220422145314333](3-CallingConvention.assets/image-20220422145314333.png)

- 汇编函数结构例子：
  1. prologue：Stack Pointer减16，为新的Stack Frame分配16B空间；将Return address保存在Stack Pointer位置
  2. body：调用sum_to并对结果乘以2
  3. Epllogue：将Return address加载回ra寄存器，通过对Stack Pointer加16来删除刚刚创建的Stack Frame，最后ret从函数中退出

![image-20220422152407008](3-CallingConvention.assets/image-20220422152407008.png)

![image-20220422163733134](3-CallingConvention.assets/image-20220422163733134.png)

[其他看视频]

## Struct Layout in memory

- struct在内存中是一段连续的地址
- 可以认为struct像是一个数组，但是里面的不同字段的类型可以不一样