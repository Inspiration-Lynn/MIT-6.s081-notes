# chapter 4 Traps and system calls

## 4.3 Code: Calling system calls

how the user call makes its way to the exec system call’s implementation in the kernel？

1. `initcode.S` invoking the `exec` system call (user/initcode.S)

   ```assembly
   start:
           la a0, init
           la a1, argv
           li a7, SYS_exec
           ecall
   ```

   - 系统调用的参数首先存在寄存器中（像其他函数调用参数一样）

   - 系统调用号被存入寄存器`a7`中（syscall.h中定义系统调用号）

   - `ecall`指令调用

2. 函数`syscall` (kernel/syscall.c:133) 在`a7`中取回系统调用号，索引相应的系统调用函数并调用；返回值存在trapframe->a0中。

   - syscalls[]为函数指针数组

   ![image-20220413153632840](4-TrapsAndSystemCalls.assets/image-20220413153632840.png)

   - `syscalls[7]()`->`sys_exec` (kernel/sysfile.c) ，获取系统调用参数，进入`exec`函数实现。

   ![image-20220414163557015](4-TrapsAndSystemCalls.assets/image-20220414163557015.png)



## 4.4 Code: System call arguments

user code calls system call wrapper functions, the arguments are initially in registers, and trap code saves them to the current process's trap frame.

The functions `argint`, `argaddr`, and `argfd` retrieve the n ’th system call argument from the trap frame as an integer, pointer, or a file descriptor. They all call `argraw` to retrieve the appropriate saved user register (kernel/syscall.c:35).

- pass pointers as arguments 2 challenges

......未完待续