

contention problem in memcached.
after 3-4 cores, throughput significantly degrade.
# Memcached-1.6.10 Source Code Break Down

## Check here for full official documentation of ascii protocol commands.
[https://github.com/memcached/memcached/blob/master/doc/protocol.txt](https://github.com/memcached/memcached/blob/master/doc/protocol.txt)

## Check here for full official documentation of binary protocol commands.
[https://github.com/memcached/memcached/wiki/BinaryProtocolRevamped](https://github.com/memcached/memcached/wiki/BinaryProtocolRevamped) 
-----------------------------------------------------


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
- **slots** is a list of free chunks in the slab class. When a new slab is added to the slab class, slab class will first convert the slab to a list of chunks then prepend these chunks to this freelist (`slots`). When reclaiming chunk from freed item (evict or delete), the chunk will also be added back to this freelist. 
- **sl_curr** tracks size of the freelist (total avaliable chunks in the slab class).
- **slab_list** is an array of pointers to all slabs that belong to this slab class. (when the array is saturated, the size of the slab_list grow by factors of two every time)
- **list_size** is the size of `slab_list`.

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


## Memcached - items.{c, h}
-----------------------------------------------------

### Segmented LRU

- Memcached originally have 255 slab classes avaliable  by default (exclude the "special" slab[0]). The item stores its slab class id in `uint8_t slabs_clsid`. 
- Segment LRU replacement breaks LRU list into four different sub-list.
	- HOT_LRU, WARM_LRU, COLD_LRU, TEMP_LRU
------------------------------------------------------------
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
**_stritem fields description:**
- `slabs_clsid`  
- `refcount`, memcached use counter technique to avoid "early removal". For each item, the refcount is initially 1. The 1 represents that such item is currently referenced by LRU and hashtable.  In current version, the refcount is decremented by calling do_item_remove.


```c
/* See items.c */
uint64_t  get_cas_id(void);
void  set_cas_id(uint64_t  new_cas);
/*@null@*/
item  *do_item_alloc(char  *key, const  size_t  nkey, const  unsigned  int  flags, const  rel_time_t  exptime, const  int  nbytes);
item_chunk  *do_item_alloc_chunk(item_chunk  *ch, const  size_t  bytes_remain);
item  *do_item_alloc_pull(const  size_t  ntotal, const  unsigned  int  id);
void  item_free(item  *it);
bool  item_size_ok(const  size_t  nkey, const  int  flags, const  int  nbytes);
int  do_item_link(item  *it, const  uint32_t  hv); /** may fail if transgresses limits */
void  do_item_unlink(item  *it, const  uint32_t  hv);
void  do_item_unlink_nolock(item  *it, const  uint32_t  hv);
void  do_item_remove(item  *it);
void  do_item_update(item  *it); /** update LRU time to current and reposition */
void  do_item_update_nolock(item  *it);
int  do_item_replace(item  *it, item  *new_it, const  uint32_t  hv);
void  do_item_link_fixup(item  *it);
int  item_is_flushed(item  *it);
unsigned  int  do_get_lru_size(uint32_t  id);
void  do_item_linktail_q(item  *it);
void  do_item_unlinktail_q(item  *it);
item  *do_item_crawl_q(item  *it);
void  *item_lru_bump_buf_create(void);
```



```c
int  lru_pull_tail(const int orig_id, 
				   const int cur_lru,
				   const uint64_t total_bytes, 
				   const uint8_t flags,
				   const rel_time_t max_age, 
				   struct lru_pull_tail_return *ret_it)
```
## Memcached - threads.c
-----------------------------------------------------

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU3OTU1NjE4NiwtOTkxMDE0OTgsLTg0Mz
gxNzE4NywtMTcxMjQ5OTM3NSwxMzc5MTA2MDM0LDMzMDI4Mjc0
NCwtMTg5NjIyMjkyNCwtODQ1MzU3NTYsLTE0NDM1ODQ1ODksMj
AyNTY5MTA3MywtMTc5MzQwMTk4MiwtMjM2NjkyODI2LC0zNDUx
Mzk0NDcsODI3NTYyODU0XX0=
-->