# Ex4
Ex4需要我们完成这几个函数
```c
pgdir_walk()
boot_map_region()
page_lookup()
page_remove()
page_insert()
```
pgdir_walk()这个函数是最关键的函数。
pgdir_walk()通过给定页目录表pgdir跟虚拟地址va获得va所对应的 page_table_entry(页表项)的虚拟地址。
有个问题很关键，pgdir里面保存的是page_table的虚拟地址还是物理地址呢？答案应该是物理地址。我个人的理解是，如果保存的是虚拟地址，那我们又要再一次对这个虚拟地址寻址，不停的重复，根本停不下来。
pgdir_walk()是本Ex最关键也是最困难的函数，但是知道了pgdir里面的page_table地址是物理地址之后就好办了。
同时这个函数也可以在xv6中的vm.c里面找到。
```c
pte_t *
pgdir_walk(pde_t *pgdir, const void *va, int create)
{
	// Fill this function in
	pde_t *pde = &pgdir[PDX(va)];
	pte_t *pt;
	if (pde != NULL && (*pde & PTE_P)) {
	    pt = (pte_t *)KADDR(PTE_ADDR(*pde));
	} else {
	    if (!create) {
		return NULL;
	    }
	    struct PageInfo *pp = page_alloc(ALLOC_ZERO);
	    if (pp == NULL) {
		return NULL;
	    }
	    pp->pp_ref++;
	    *pde = page2pa(pp) | PTE_P | PTE_W | PTE_U;
	    pt = (pte_t *)KADDR(PTE_ADDR(*pde));
	}
	return &pt[PTX(va)];
}

```
boot_map_region()函数
```c
static void
boot_map_region(pde_t *pgdir, uintptr_t va, size_t size, physaddr_t pa, int perm)
{
	// Fill this function in
	size_t i;
	for (i = 0; i * PGSIZE < size; i++) {
	    uintptr_t currVa = va + i * PGSIZE;
	    physaddr_t currPa = pa + i * PGSIZE;
	    pte_t *pte = pgdir_walk(pgdir, (void *)currVa, 1);
	    if (pte == NULL) {
			panic("boot_map_region: pte is NULL");
	    }
	    *pte = pa | perm | PTE_P;
	}
}

```
page_insert()函数
在该函数的corner_hint里有说到，如果把同一个pp再次插入到同一个va会发生什么情况，注意这种情况的处理。
首先考虑会发生什么情况，如果当前ref==1，那么再进行page_remove()操作后ref==0，当前页会被放回free_page_list，再次插入当前页的时候就会导致free_page_list出现问题。
解决方法：只需要在page_remove()之前进行ref++操作。
```c
int
page_insert(pde_t *pgdir, struct PageInfo *pp, void *va, int perm)
{
	// Fill this function in
	pte_t *pte = pgdir_walk(pgdir, va, 1);
	if (pte == NULL) {
	    return -E_NO_MEM;
	}
	pp->pp_ref++;
	if (pte != 0) {
	    page_remove(pgdir, va);
	}
	*pte = page2pa(pp) | perm | PTE_P;
	return 0;
}

```

page_lookup()
```c
struct PageInfo *
page_lookup(pde_t *pgdir, void *va, pte_t **pte_store)
{
	// Fill this function in
	pte_t *pte = pgdir_walk(pgdir, va, 0);
	if (pte == NULL) {
	    return NULL;
	}
	if (pte_store != NULL) {
	    *pte_store = pte;
	}
	return pa2page(*pte);
	//return NULL;
}
```

page_remove()
```c
void
page_remove(pde_t *pgdir, void *va)
{
	// Fill this function in
	pte_t *pte = pgdir_walk(pgdir, va, 0);
	if (pte == NULL || *pte == 0) {
	    return ;
	}
	struct PageInfo *pp = pa2page(*pte);
	page_decref(pp);
	*pte = 0;
	tlb_invalidate(pgdir, va);
}
```