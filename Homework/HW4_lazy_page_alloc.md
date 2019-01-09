# HW4

Part One:

```c
int
sys_sbrk(void)
{
  int addr;
  int n;

  if(argint(0, &n) < 0)
    return -1;
  addr = myproc()->sz;
  myproc()->sz += n;
  return addr;
}

```

Part Two:
一开始没有找到pgdir在哪，然后发现myproc()有个,模仿allocuvm写就玩事儿了
```c
    if (tf->trapno == T_PGFLT) {
	char *mem = kalloc();
	if (mem == 0) {
	    cprintf("allocuvm out of memory\n");
	}
	memset(mem, 0, PGSIZE);
	if (mappages(myproc()->pgdir, (char*)PGROUNDDOWN(rcr2()), 
	    PGSIZE, V2P(mem), PTE_W|PTE_U) < 0) {
	    cprintf("allocuvm out of memory (2)\n");
	    kfree(mem);
	}
	return ;
    }
```