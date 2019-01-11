# HW5
```c
trap.c
case T_IRQ0 + IRQ_TIMER:

    if (myproc() != 0 && (tf->cs & 3) == 3) {
      myproc()->curticks++;
      if (myproc()->alarmticks != -1 && myproc()->curticks == myproc()->alarmticks) {
	myproc()->curticks = 0;

	tf->esp -= 4;
	*((uint *)(tf->esp)) = tf->eip;

	tf->eip = (uint)(myproc()->alarmhandler);
      }
    }

```
函数调用前会先把函数调用后的下一条指令的地址压进栈里，再调用结束时pop到eip里，由于我们目前处于内核态，所以这部分过程只能手动进行
