# Ex5

Ex5让我们补全 pmap.c中mem_init()的剩余部分

```c
	// Map 'pages' read-only by the user at linear address UPAGES
	// Permissions:
	//    - the new image at UPAGES -- kernel R, user R
	//      (ie. perm = PTE_U | PTE_P)
	//    - pages itself -- kernel RW, user NONE
	// Your code goes here:

	boot_map_region(kern_pgdir, UPAGES, npages * sizeof(struct PageInfo), PGSIZE), PADDR(pages), PTE_U);

```
虽然$ npages = 32768$ 是PGSIZE的倍数，为了看得比较清楚，我们还是ROUNDUP了一下对PGSIZE对齐
这部分是为了让用户可以read页表的相关信息

```c
	// Use the physical memory that 'bootstack' refers to as the kernel
	// stack.  The kernel stack grows down from virtual address KSTACKTOP.
	// We consider the entire range from [KSTACKTOP-PTSIZE, KSTACKTOP)
	// to be the kernel stack, but break this into two pieces:
	//     * [KSTACKTOP-KSTKSIZE, KSTACKTOP) -- backed by physical memory
	//     * [KSTACKTOP-PTSIZE, KSTACKTOP-KSTKSIZE) -- not backed; so if
	//       the kernel overflows its stack, it will fault rather than
	//       overwrite memory.  Known as a "guard page".
	//     Permissions: kernel RW, user NONE
	// Your code goes here:

	boot_map_region(kern_pgdir, KSTACKTOP - KSTKSIZE, KSTKSIZE, PADDR(bootstack), PTE_W);
	
```


将bootstack  [0x10d00, 0x115000)部分的物理内存映射到$[KSTACKTOP-KSTKSIZE, KSTACKTOP)$部分

```c
	// Map all of physical memory at KERNBASE.
	// Ie.  the VA range [KERNBASE, 2^32) should map to
	//      the PA range [0, 2^32 - KERNBASE)
	// We might not have 2^32 - KERNBASE bytes of physical memory, but
	// we just set up the mapping anyway.
	// Permissions: kernel RW, user NONE
	// Your code goes here:

	boot_map_region(kern_pgdir, KERNBASE, 0x10000000, 0, PTE_W);
	
```
映射$[KERNBASE, 2^{32})$部分虚拟地址映射到物理地址$[0, 2^{32} - KERNBASE)$部分



所以最后映射的情况如下

|PGDIR|va|pa||
|:-----:|:-----:|:-----:|:-----:|
|960-1023|0xf00000000-0xffc00000|0x0-0xfc00000|[KERNBASE, 2^32)|
|959|0xefff8000|0x10d000|bootstack|
|956|0xef000000|0x119000|UPAGE|

### Question
2. 如上
3. 通过PTE_U，mmu转换
4. 256M，KERNBASE上方的空间与所有物理地址映射
5. 一个PTE是4B + 一个PageInfo 8B， 总共是4GB/4KB×12B = 12M
6. 物理地址[0, 4MB)被映射到到两个虚拟地址[KERNBASE, KERNBASE + 4MB)和[0, 4MB)

跳过了challenge～
