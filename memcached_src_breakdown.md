

contention problem in memcached.
after 3-4 cores, throughput significantly degrade.


## Memcached - Slabs.c
-----------------------------------------------------
```c
	typedef  struct {
		unsigned  int size; /* sizes of items */
		unsigned  int perslab; /* how many items per slab */
		void *slots; /* list of item ptrs */
		unsigned  int sl_curr; /* total free items in list */
		unsigned  int slabs; /* how many slabs were allocated for this class */
		void **slab_list; /* array of slab pointers */
		unsigned  int list_size; /* size of prev array */
	} slabclass_t;
```

Memcached maintain an array of `slabclass` as:

`slabclass[MAX_NUMBER_OF_SLAB_CLASSES]`

### Relevant global parameters:
```c
static size_t mem_limit = 0;
```
**mem_limit** is set during *slabs_init()*, this
```c
static size_t mem_malloced = 0;
```
### Relevant internal slab functions:

```c
void  slabs_init(const size_t limit, const double factor, const bool prealloc, const uint32_t *slab_sizes, void *mem_base_external, bool  reuse_mem);
```

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
eyJoaXN0b3J5IjpbLTI2MTg5OTQ1NiwtMTc5MzQwMTk4MiwtMj
M2NjkyODI2LC0zNDUxMzk0NDcsODI3NTYyODU0XX0=
-->