

contention problem in memcached.
after 3-4 cores, throughput significantly degrade.


## Memcached - Slabs.c
-----------------------------------------------------
The *slabclass* is memcached is defined as follow:
```c
	typedef  struct {
		unsigned  int size; /* sizes of items (chunk size)*/
		unsigned  int perslab; /* how many items per slab */
		void *slots; /* list of item ptrs */
		unsigned  int sl_curr; /* total free items in list */
		unsigned  int slabs; /* how many slabs were allocated for this class */
		void **slab_list; /* array of slab pointers */
		unsigned  int list_size; /* size of prev array */
	} slabclass_t;
```

The Memcached maintains a global array of `slabclass` as:

```c
static slabclass_t slabclass[MAX_NUMBER_OF_SLAB_CLASSES];
```
**slabclass_t fields description:**
-  **size** define the chunk size of the slab class (chunk size is 8 bytes aligned in memcached)
-  **perslab** is the number of chunk each slab can hold. (By default each slab (page) is 1MB)
- **slots**

### Relevant Global Parameters:
```c
static size_t mem_limit = 0;
```
- **mem_limit** is set during *slabs_init()*, this controls the maximum allowed number of bytes specified by the user.
```c
static size_t mem_malloced = 0;
```
- **mem_malloced** stores the total memory currently allocated. The mem_malloced get increment by "size" whenever *memory_allocate(size)* is called. This by default, typically will increment 1 MB at a time, because the default page (slab) size is 1MB, memcached allocate 1 slab at a time. In memcached, when *prealloc* option is on, the *slab_init()* will try to allocate until *mem_limit* is reached. 
```c
static bool mem_limit_reached = false;
```
- **[temp]** This indicates wether mem_limit has been hit, when slabs rebalance finished this is set back to false. This variable seems like is for *exstore*. None of the slab function in slab.c is actually depends on this parameters.
```c
static  int  power_largest;
```
- **power_largest** is the index of largest slabclass. The name power_largest is probably because, by default, the chunk size of each slabclass is increase by a factor every time, hence the chunk size is power of the slabclass's index. 

```c
static void *mem_base = NULL;
static  void *mem_current = NULL;
static  size_t  mem_avail = 0;
```
- When you prealloc memory during initialization, then this **mem_base** will serve as pointer to the prealloc memory. The **mem_current** is pointer to the next avaliable memory from prealloc memory. When *memory_allocate()* method is called, *if memory is prealloc*, the function will return mem_current pointer, and move the *mem_current* to next available address. 
- **mem_avail** simply tracks the size of remaining preallocated memory
- *These three parameters are only used when you configure memcached with preallocate memory*


### Relevant Internal Slab Functions:

```c
void  slabs_init(const size_t limit, const double factor, const bool prealloc, const uint32_t *slab_sizes, void *mem_base_external, bool  reuse_mem);
```
- The slab init is called in the initialization phase of memcached in `main()`.
- This function has two primary tasks
	- The main tasks of this function is to initialize the `slabclass[]` array.  Particularly, this function initialize field: `slabclass->size` and 	`slabclass->perslab`.  By defaults, the slabclass start with `size = sizeof(item) + settings.chunk_size` (the default chunk size is 48). Then the size of next slabclass grow by the `factor`.  Note that the size of the last slabclass is always equal to the `slab_chunk_size_max`, which is by default set to half of the slab's size. If the input `slab_sizes` array is not null, the slabclass's size will initialize according to the `slab_sizes` array. Then, the `perslab` field is calculated based on the `size` of each slabclass. *Note that the slab's chunk size is 8 bytes aligned, so if the `slab_sizes` array is not aligned then they adjust the `size` to make it 8 bytes aligned.*
	- If prealloc is set, memcached will preallocate the memory (this could be on external memory specified by the `mem_base_external` pointer) or memcached will start allocates slabs until the memory `limits`. These preallocated memory or slabs will be stored on the first slabclass, `slabclass[0]`. In case the memory is preallocated, this slab class acts as a global free slab pool, which stores all free  unclaimed slabs. 

```c
static int grow_slab_list (const unsigned int id);
```


## Memcached - items.c
-----------------------------------------------------
```c
/**
* Structure for storing items within memcached.
*/
typedef  struct  _stritem {
	/* Protected by LRU locks */
	struct  _stritem  *next;
	struct  _stritem  *prev;
	/* Rest are protected by an item lock */
	struct  _stritem  *h_next; 		/* hash chain next */
	rel_time_t  		time; 		/* least recent access */
	rel_time_t  		exptime; 	/* expire time */
	int  				nbytes; 	/* size of data */
	unsigned  short  	refcount;
	uint16_t  			it_flags; 	/* ITEM_* above */
	uint8_t  			slabs_clsid;	/* which slab class we're in */
	uint8_t  			nkey; 			/* key length, w/terminating null and padding */
	/* this odd type prevents type-punning issues when we do
	* the little shuffle to save space when not using CAS. */
	union {
		uint64_t  cas;
		char  end;
	} data[];

	/* if it_flags & ITEM_CAS we have 8 bytes CAS */
	/* then null-terminated key */
	/* then " flags length\r\n" (no terminating null) */
	/* then data with terminating \r\n (no terminating null; it's binary!) */
} item;
```


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc3NTE5MDkzOSwzMzAyODI3NDQsLTE4OT
YyMjI5MjQsLTg0NTM1NzU2LC0xNDQzNTg0NTg5LDIwMjU2OTEw
NzMsLTE3OTM0MDE5ODIsLTIzNjY5MjgyNiwtMzQ1MTM5NDQ3LD
gyNzU2Mjg1NF19
-->