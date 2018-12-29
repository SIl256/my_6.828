# Ex1

Ex1 需要我们完成下面的几个函数

```c
boot_alloc()
mem_init()(only up to the call to check_page_free_list(1))
page_init()
page_alloc()
page_free()
```
boot_alloc()
```c
	if (n == 0) {
	    return (void *)nextfree;
	}
	if (n > 0) {
	    result = nextfree;
	    if (PADDR((void *)(n + result)) >= npages * PGSIZE) {
		panic("boot_alloc: out of memory\n");
	    }
	    nextfree = ROUNDUP(nextfree + n, PGSIZE);
	    return result;
	}
```
根据页表数量我们知道总共的物理地址有npages * PGSIZE

mem_init()
```c
	pages = (struct PageInfo *) boot_alloc(npages * sizeof(struct PageInfo));
	memset(pages, 0, npages * sizeof(struct PageInfo));
```
申请与页表项数量相同的空间并将这部分空间清零

page_init()
```c
	size_t i;
	size_t EXTPHYSMEM_END = PADDR(boot_alloc(0));
	for (i = 1; i < npages_basemem; i++) {
	    pages[i].pp_ref = 0;
	    pages[i].pp_link = page_free_list;
	    page_free_list = &pages[i];
	}

	for (i = EXTPHYSMEM_END / PGSIZE; i < npages; i++) {
	    pages[i].pp_ref = 0;
	    pages[i].pp_link = page_free_list;
	    page_free_list = &pages[i];
	}
```
将页表初始化，，需要跳过IO hole ，kernel 所在的地址，申请的pages的地址和kern_pagedir的地址
kernel的起始地址是EXTPHYSMEM，由boot_alloc申请的地址是接在符号end后面，也就是接在kernel之后
可以直接通过PADDR(boot_alloc(0))获得该页所对应的物理空间起始地址
我一开始把pages所拥有的地址跟kern_pagedir所拥有的地址也放到了page_free_list中，导致了系统崩溃

page_alloc()
```c
	if (page_free_list == NULL) {
	    return NULL;
	}
	struct PageInfo *pp;
	pp = page_free_list;
	page_free_list = pp->pp_link;
	pp->pp_link = NULL;
	if (alloc_flags & ALLOC_ZERO) {
	    memset(page2kva(pp), 0, PGSIZE);
	}
	return pp;
```
page_free()
```c
	if (pp->pp_ref != 0) {
	    panic("page_free: pp->pp_ref is nonzero");
	}
	if (pp->pp_link != NULL) {
	    panic("page_free: pp->pp_link is not NULL");
	}
	pp->pp_link = page_free_list;
	page_free_list = pp;
```
这两部分都是对free_page_list进行链表操作


最后说一下物理内存的一些分配吧
```
physical page
+---------------------------------+ <- 0x8000000
:	free			  :
+---------------------------------+
|	pages			  |
+---------------------------------+
|	kern_pgdir		  |
+---------------------------------+ <- 0x115000
|	kernel			  |(准确的来说kernel与kern_pgdir之间
|				  |有一些空隙，为了向PGSIZE对齐)
+---------------------------------+ <- EXTPHYSMEM, 640K
|	IO hole			  |
+---------------------------------+ <- IOPHYSMEM
:	:			  :
+---------------------------------+
|real-mode IDT and BIOS structures| <- page 0
+---------------------------------+
```

