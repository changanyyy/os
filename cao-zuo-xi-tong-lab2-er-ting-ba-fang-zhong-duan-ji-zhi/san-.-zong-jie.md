---
description: 下面我们进行灵魂三问
---

# 总结

在本次实验的末尾，我们将问你几个问题（必须回答），看看你是否真正理解了系统调用和硬件中断的过程。如果你是**自己**认真完成的，一定可以回答的很好，否则......

{% hint style="danger" %}
conclusion1：请回答以下问题

请_**结合代码**_，_**详细描述**_用户程序app调用printf时，从**lib/syscall.c中syscall函数开始执行**到**lib/syscall.c中syscall返回**的全过程，硬件和软件分别做了什么？（实际上就是中断执行和返回的全过程，syscallPrint的执行过程不用描述）
{% endhint %}

{% hint style="danger" %}
conclusion2：请回答下面问题

请_**结合代码**_，_**详细描述**_当产生保护异常错误时，硬件和软件进行配合处理的全过程。
{% endhint %}

{% hint style="danger" %}
conclusion3：请回答下面问题

如果上面两个问题你不会，在实验过程中你一定会出现很多疑惑，有些可能到现在还没有解决。请说说你的疑惑。
{% endhint %}

{% hint style="danger" %}
challenge4：根据框架代码，我们设计了一个比较完善的中断处理机制，而这个框架代码也仅仅是实现中断的海量途径中的一种设计。请找到框架中你认为需要改进的地方进行适当的改进，展示效果（非常灵活的一道题，不写不扣分）。
{% endhint %}
