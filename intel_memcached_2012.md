

contention problem in memcached.
after 3-4 cores, throughput significantly degrade.


## Memcached - Slabs.c
-----------------------------------------------------
``
	typedef  struct {
	unsigned  int size; /* sizes of items */
	unsigned  int perslab; /* how many items per slab */
	void *slots; /* list of item ptrs */

	unsigned  int sl_curr; /* total free items in list */

	  

	unsigned  int slabs; /* how many slabs were allocated for this class */

	  

	void **slab_list; /* array of slab pointers */

	unsigned  int list_size; /* size of prev array */

	} slabclass_t;
``
> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExNTEwMjYyNTcsMjAwNDczNjYxMCw3Mz
A5OTgxMTZdfQ==
-->