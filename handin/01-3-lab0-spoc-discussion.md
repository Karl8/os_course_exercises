# lab0 SPOC思考题

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

- 硬件设计上的支持：
    - 进程的切换需要硬件支持时钟中断，操作系统通过响应时钟中断来实现进程的并发。
    - 虚存管理需要地址映射，从而需要TLB、MMU等CPU的硬件结构。
    - 对于文件系统，需要硬件有稳定的存储介质来保证操作系统的持久性。
  - 特权指令：
    - 对于进程：
      - 允许或禁止中断，以响应时钟中断；
      - 允许切换进程
    - 对于虚存：
      - 允许访问TLB；
      - 清理内存；
      - 内存保护；
    - 对于文件系统：
      - 执行I/O操作
- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？
  - 实模式和保护模式的区别：
    - 可访问的物理内存空间大小不同。实模式1MB，保护模式4GB。
    - 进程的内存有区别：
      - 实模式下，系统程序和用户程序没有区别，用户程序可以访问并修改系统程序的内存区域，进程的地址与物理地址一一对应，用户程序可以轻易地直接访问任何一块内存区域。
      - 保护模式下，用户程序不可以访问或修改系统程序的内存区域，进程的虚拟地址需要操作系统转化为物理地址，用户程序无法直接得到物理内存地址。
  - 物理地址：计算机内存或外设在总线上的地址。
  - 线性地址：逻辑地址到物理地址变换的中间层，由段机制所形成的地址空间。
  - 逻辑地址：访存指令给出的地址/进程的虚拟地址。

- 理解list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```
每一个成员变量在struct中占的bit位数。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？
intr内实际存储的值是`0010 0000 0000 0000 0011 0000 0000 0000` 
转化为16进制是`0x30002`

