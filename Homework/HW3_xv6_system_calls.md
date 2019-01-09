# HW3 System calls
第一题比较简单
```c
static char syscalls_name[][10] = {
[SYS_fork]   "fork",
[SYS_exit]   "exit",
[SYS_wait]   "wait",
[SYS_pipe]   "pipe",
[SYS_read]   "read",
[SYS_kill]   "kill",
[SYS_exec]   "exec",
[SYS_fstat]  "fstat",
[SYS_chdir]  "chdir",
[SYS_dup]    "dup",
[SYS_getpid] "getpid",
[SYS_sbrk]   "sbrk",
[SYS_sleep]  "sleep",
[SYS_uptime] "uptime",
[SYS_open]   "open",
[SYS_write]  "write",
[SYS_mknod]  "mknod",
[SYS_unlink] "unlink",
[SYS_link]   "link",
[SYS_mkdir]  "mkdir",
[SYS_close]  "close",
[SYS_date]   "date"
};
void
syscall(void)
{
  int num;
  struct proc *curproc = myproc();

  num = curproc->tf->eax;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    curproc->tf->eax = syscalls[num]();
    cprintf("%s -> %d\n", syscalls_name[num], curproc->tf->eax);
  } else {
    cprintf("%d %s: unknown sys call %d\n",
            curproc->pid, curproc->name, num);
    curproc->tf->eax = -1;
  }

}

```

第二题
通过给的grep命令可以看到一些我们需要模仿的地方
```
grep -n uptime *.[chS]

syscall.c:105:extern int sys_uptime(void);
syscall.c:122:[SYS_uptime]  sys_uptime,
syscall.c:147:[SYS_uptime] "uptime",
syscall.h:15:#define SYS_uptime 14
sysproc.c:83:sys_uptime(void)
user.h:25:int uptime(void);
usys.S:31:SYSCALL(uptime)
```

看完里面的一些系统调用函数后会发现，要手动用arg*()从用户栈里取出我们需要的参数，于是代码就比较清晰了

```c
sysproc.c

int
sys_date(void)
{
    struct rtcdate *r;
    if (argptr(0, (char **)&r, 24) < 0) {
	return -1;
    }
    cmostime(r);
    return 0;
}

extern int sys_date(void);

syscall.h
#define SYS_date   22

user.h
int date(void);

usys.h
SYSCALL(date)
```