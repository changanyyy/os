---
description: 大家在ICS里面也学过TSS吧。当时可能学的一脸懵逼，因为只接触过理论知识，并没有深入机器运行来亲身感受TSS是干什么的。本次实验就有这样的机会！
---

# 中断时的切换

## 发生中断时机器如何切换

前面的基础知识都是为了本节铺垫。说了那么多有关中断的概念，下面来讲讲发生中断机器是怎么切换状态的。

中断会改变程序正常执行的流程。举一个例子，我们称中断到来之前CPU正在执行的程序为A（后面我们会学到，它叫进程，目前姑且叫做程序）。中断到来时，CPU被告知：中断来了（谁告诉CPU的呢？）。CPU不能再执行A了，它应该先去处理到来的中断。因此它应该跳转到一个地方（如何跳转呢？），去执行中断处理的代码，结束之后再恢复A的执行。可以看到，A的执行流程被中断打断了，为了以后能够完美地恢复到被中断打断时的状态（就好像没有处理过这个中断一样），CPU在处理中断之前应该先把A的状态保存起来，等到中断处理结束之后，根据之前保存的信息把计算机恢复到A被打断之前的状态。这样A就可以继续运行了, 在它看来, 中断就好像没有发生过一样。

总而言之，流程大概是这样：

1. CPU被告知有中断要处理；
2. CPU**把当前进行的任务保存**（保存到哪里呢？）；
3. 切换到中断处理程序（往往是内核态）；
4. 中断处理结束，应当**把保存的任务恢复**，返回用户态。

这个过程，就好像你正在写作业，你妈妈叫你去吃饭，你就把写一半的作业放在那（保存在作业本上），等吃完饭回来继续写一样。

## 保存哪些状态？

接下来的问题是, 哪些内容表征了A的状态？ CPU又应该将它们保存到哪里去？

在IA-32中，A的状态中比较重要的有如下几个：

首先当是**`EIP（`instruction pointer）**了，它指示了 A 在被打断的时候正在执行哪条指令；然后就是`EFLAGS`（各种标志位）和`CS`（代码段，里面存着CPL）。_由于一些特殊的原因_，这三个寄存器的内容必须由硬件来保存。此外, 通用寄存器（GPR，general propose register）的值对A来说还是有意义的，而进行中断处理的时候又难免会使用到寄存器. 但硬件并不负责保存它们，因此我们还需要手动保存它们的值。

{% hint style="info" %}
exercise7：请问上面用斜体标出的“一些特殊的原因”是什么？

（Hint：为啥不能用软件保存呢？注：这里的软件是指一些函数或者指令序列，不是gcc这种软件。）
{% endhint %}

## 状态保存到哪里？

要将这些信息保存到哪里去呢？一个合适的地方就是程序的堆栈。中断到来时，硬件会自动将`EFLAGS`，`CS`，`EIP`三个寄存器的值保存到堆栈上。此外, IA-32提供了`pusha`/`popa`指令, 用于把通用寄存器的值压入/弹出堆栈（请回忆NEMU里面，你写过的pusha和popa），但你需要注意压入的顺序（请查阅i386手册）。如果希望支持中断嵌套（即在进行优先级低的中断处理的过程中，响应另一个优先级高的中断，比如在printf的时候，发生除零异常），那么堆栈将是保存信息的唯一选择。如果选择把信息保存在一个固定的地方，发生中断嵌套的时候，第一次中断保存的状态信息将会被优先级高的中断处理过程所覆盖！

{% hint style="info" %}
exercise8：请简单举个例子，什么情况下会被覆盖？（即稍微解释一下上面那句话）
{% endhint %}

**IA-32借助`TR`和TSS来确定保存`EFLAGS`，`CS`，`EIP`这些寄存器信息的新堆栈。**

`TR`（Task state segment Register）是16位的任务状态段寄存器，**结构和`CS`这些段寄存器完全一样**，它存放了GDT的一个索引，可以使用`ltr`指令进行加载，通过`TR`可以在GDT中找到一个TSS（Task state segment Register，可以看出来，也是一个段！）段描述符，索引过程如下：

```
                      +-------------------------+
                      |                         |
                      |                         |
                      |       TASK STATE        |
                      |        SEGMENT          |<---------+
                      |                         |          |
                      |                         |          |
                      +-------------------------+          |
       16-BIT VISIBLE             ^                        |
          REGISTER                |   HIDDEN REGISTER      |
   +--------------------+---------+----------+-------------+------+
TR |      SELECTOR      |      (BASE)        |       (LIMT)       |
   +---------+----------+--------------------+--------------------+
             |                    ^                     ^
             |                    +-----------------+   |
             |          GLOBAL DESCRIPTOR TABLE     |   |
             |        +-------------------------+   |   |
             |        |     TSS DESCRIPTOR      |   |   |
             |        +------+-----+-----+------+   |   |
             |        |      |     |     |      |---+   |
             |        |------+-----+-----+------|       |
             +------->|            |            |-------+
                      +------------+------------+
                      |                         |
                      +-------------------------+
```

TSS是任务状态，不同于代码段、数据段，TSS是一个系统段，用于存放任务的状态信息，主要用在硬件上下文切换。

TSS提供了3个堆栈位置（`SS`和`ESP`），当发生堆栈切换的时候，CPU将根据目标代码特权级的不同，从TSS中取出相应的堆栈位置信息进行切换，例如我们的中断处理程序位于ring0，因此CPU会从TSS中取出`SS0`和`ESP0`进行切换。

为了让硬件在进行堆栈切换的时候可以找到新堆栈，内核需要将新堆栈的位置写入TSS的相应位置，TSS中的其它内容主要在硬件上下文切换中使用，但是因为效率的问题大多数现代操作系统都不使用硬件上下文切换，因此TSS中的大部分内容都不会使用。

TSS的结构如下图所示（你看它是不是更像个......结构体，对，你就可以看成结构体）：

```
 31              23              15              7             0
+---------------+---------------+---------------+-------------+-+
|          I/O MAP BASE         | 0 0 0 0 0 0 0   0 0 0 0 0 0 |T|64
|---------------+---------------+---------------+-------------+-|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              LDT              |60
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              GS               |5C
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              FS               |58
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              DS               |54
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              SS               |50
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              CS               |4C
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              ES               |48
|---------------+---------------+---------------+---------------|
|                              EDI                              |44
|---------------+---------------+---------------+---------------|
|                              ESI                              |40
|---------------+---------------+---------------+---------------|
|                              EBP                              |3C
|---------------+---------------+---------------+---------------|
|                              ESP                              |38
|---------------+---------------+---------------+---------------|
|                              EBX                              |34
|---------------+---------------+---------------+---------------|
|                              EDX                              |30
|---------------+---------------+---------------+---------------|
|                              ECX                              |2C
|---------------+---------------+---------------+---------------|
|                              EAX                              |28
|---------------+---------------+---------------+---------------|
|                            EFLAGS                             |24
|---------------+---------------+---------------+---------------|
|                    INSTRUCTION POINTER (EIP)                  |20
|---------------+---------------+---------------+---------------|
|                          CR3  (PDPR)                          |1C
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              SS2              |18
|---------------+---------------+---------------+---------------|
|                             ESP2                              |14
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              SS1              |10
|---------------+---------------+---------------+---------------|
|                             ESP1                              |0C
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|              SS0              |8
|---------------+---------------+---------------+---------------|
|                             ESP0                              |4
|---------------+---------------+---------------+---------------|
|0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0|   BACK LINK TO PREVIOUS TSS   |0
+---------------+---------------+---------------+---------------+
```

> **思考一下（不必回答）：ring3的堆栈在哪里?**
>
> IA-32提供了4个特权级, 但TSS中只有3个堆栈位置信息, 分别用于ring0, ring1, ring2的堆栈切换.为什么TSS中没有ring3的堆栈信息？

加入硬件堆栈切换之后, 中断到来/从中断返回的硬件行为如下（通过伪代码的形式展现，可以看成类似C语言逻辑的硬件代码，描述硬件行为）：

```
old_CS = CS
old_EIP = EIP
old_SS = SS
old_ESP = ESP
target_CS = IDT[vec].selector       //target_cs是中断向量号为vec的中断描述符里的selector
target_CPL = GDT[target_CS].DPL     //target_CPL是目标代码段的DPL
if(target_CPL < GDT[old_CS].DPL)    //特权级检查！！！（请翻找手册前面）
    TSS_base = GDT[TR].base         //检查通过，获得TSS段的基地址（结构体地址）
    switch(target_CPL)              //我要切换到ring几（在本实验中只有ring0）
        case 0:                          
            SS = TSS_base->SS0               
            ESP = TSS_base->ESP0              
        case 1:                              
            SS = TSS_base->SS1              
            ESP = TSS_base->ESP1             
        case 2:                            
            SS = TSS_base->SS2          
            ESP = TSS_base->ESP2     
    push old_SS                     
    push old_ESP                    
push EFLAGS                         //把原来的eflags，cs，eip都push进堆栈（思考：谁的堆栈）
push old_CS
push old_EIP

################### iret ####################
//这个和前面的流程顺序上.....
old_CS = CS                        
pop EIP
pop CS
pop EFLAGS
if(GDT[old_CS].DPL < GDT[CS].DPL)    //???
    pop ESP
    pop SS
```

硬件堆栈切换只会在目标代码特权级比当前堆栈特权级高的时候发生，即`GDT[target_CS].DPL < GDT[SS].DPL`（这里的小于是数值上的），当`GDT[target_CS].DPL = GDT[SS].DPL`时，CPU将不会进行硬件堆栈切换。比如，在内核态发生中断，中断处理还是在内核态，就不用切换堆栈，都是用内核栈。而在用户态产生中断，就需要进行切换进内核态的内核堆栈。

{% hint style="info" %}
exercise9：请解释我在伪代码里用“???”注释的那一部分，为什么要pop ESP和SS？
{% endhint %}

下图显示中断到来后内核堆栈的变化

```
                       WITHOUT PRIVILEGE TRANSITION

D  O      31          0                     31          0
I  F    |-------+-------|                 |-------+-------|
R       |#######|#######|    OLD          |#######|#######|    OLD
E  E    |-------+-------|   SS:ESP        |-------+-------|   SS:ESP
C  X    |#######|#######|     |           |#######|#######|     |
T  P    |-------+-------|<----+           |-------+-------|<----+
I  A    |  OLD EFLAGS   |                 |  OLD EFLAGS   |
O  N    |-------+-------|                 |-------+-------|
N  S    |#######|OLD CS |    NEW          |#######|OLD CS |
   I    |-------+-------|   SS:ESP        |-------+-------|
 | O    |    OLD EIP    |     |           |    OLD EIP    |    NEW
 | N    |---------------|<----+           |---------------|   SS:ESP
 |      |               |                 |  ERROR CODE   |     |
 !      *               *                 |---------------|<----+
        *               *                 |               |
        *               *
        WITHOUT ERROR CODE                 WITH ERROR CODE

                       WITH PRIVILEGE TRANSITION

D  O     31            0                     31          0
I  F    +-------+-------+<----+           +-------+-------+<----+
R       |#######|OLD SS |     |           |#######|OLD SS |     |
E  E    |-------+-------|   SS:ESP        |-------+-------|   SS:ESP
C  X    |    OLD ESP    |  FROM TSS       |    OLD ESP    |  FROM TSS
T  P    |---------------|                 |---------------|
I  A    |  OLD EFLAGS   |                 |  OLD EFLAGS   |
O  N    |-------+-------|                 |-------+-------|
N  S    |#######|OLD CS |    NEW          |#######|OLD CS |
   I    |-------+-------|   SS:ESP        |-------+-------|
 | O    |    OLD EIP    |     |           |    OLD EIP    |    NEW
 | N    |---------------|<----+           |---------------|   SS:ESP
 |      |               |                 |  ERROR CODE   |     |
 !      *               *                 |---------------|<----+
        *               *                 |               |
        *               *
        WITHOUT ERROR CODE                 WITH ERROR CODE
```
