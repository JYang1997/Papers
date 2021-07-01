# Papers

# Caching Related:
TODO :

**Lightweight Robust Size Aware Cache Management**[[Gil Einziger, 2021]
- https://arxiv.org/pdf/2105.08770.pdf 
- size aware extension to tinyLFU

Adaptive Software Cache Management [Gil Einziger, 2018]
- https://dl.acm.org/doi/pdf/10.1145/3274808.3274816

Page Replacement for General Caching Problems (Susanne Albers, 1999)
- https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.5555/314500.314528

Some Mathematical Facts About Optimal Cache Replacement [PIERRE MICHAUD, Inria 2016]
- https://dl.acm.org/doi/pdf/10.1145/3017992

Learning Forward Reuse Distance (2020 Pengcheng Li)
- https://arxiv.org/pdf/2007.15859.pdf

Efficient Miss Ratio Curve Computation for Heterogeneous Content Popularity
- [https://www.usenix.org/system/files/atc20-carra_0.pdf](https://www.usenix.org/system/files/atc20-carra_0.pdf)


Practical Bounds on Optimal Caching with Variable Object Sizes

- [http://www.cs.cmu.edu/~beckmann/publications/papers/2018.sigmetrics.foo.pdf](http://www.cs.cmu.edu/~beckmann/publications/papers/2018.sigmetrics.foo.pdf)

Optimal Data Placement for Heterogeneous Cache, Memory, and Storage Systems

- [https://www.ymsir.com/papers/chopt-sigmetrics.pdf](https://www.ymsir.com/papers/chopt-sigmetrics.pdf)

Caching Is Hard—Even in the Fault Model
- https://core.ac.uk/download/pdf/81860263.pdf

General Caching Is Hard: Even with Small Pages - Lukáš Folwarczný, 2015
- https://arxiv.org/pdf/1506.07905.pdf

Efficient stack distance computation for priority replacement policies, 2011
(generalized min tree algorithm)

- [https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/2016604.2016607](https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/2016604.2016607)

 
Optimal On-Line Computation of Stack Distances for MIN and OPT∗
- [https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/3075564.3075571](https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/3075564.3075571)



counterStack 2014

- [https://www.cs.ubc.ca/~nickhar/papers/CounterStacks/CounterStacks.pdf](https://www.cs.ubc.ca/~nickhar/papers/CounterStacks/CounterStacks.pdf)

Adaptive TTL-Based Caching for Content Delivery

(Expiration related solution for CDN cache 2017) (see arxiv for full work)

- [https://dl.acm.org/doi/pdf/10.1145/3078505.3078560](https://dl.acm.org/doi/pdf/10.1145/3078505.3078560)

 
Segcache: a memory-efficient and scalable in-memory key-value cache for small objects
(follow after twitter trace paper) (TTL)

- https://www.usenix.org/system/files/nsdi21-yang.pdf

  

An Imitation Learning Approach for Cache Replacement (opt attempt)

- https://arxiv.org/pdf/2006.16239.pdf

Tail at scale

- https://cseweb.ucsd.edu/classes/sp18/cse124-a/post/schedule/p74-dean.pdf

## Caching Policies & Design :

  
A Survey of Web Cache Replacement Strategies

- [https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/954339.954341](https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/954339.954341)

  
From FIFO to Predictive Cache Replacement (cache replacement strategy Survey)

- [https://www.net.in.tum.de/fileadmin/TUM/NET/NET-2019-06-1/NET-2019-06-1_06.pdf](https://www.net.in.tum.de/fileadmin/TUM/NET/NET-2019-06-1/NET-2019-06-1_06.pdf)

Cacheus (adaptive caching)
- [https://www.usenix.org/system/files/fast21-rodriguez.pdf](https://www.usenix.org/system/files/fast21-rodriguez.pdf)

LHD

- [https://www.usenix.org/system/files/conference/nsdi18/nsdi18-beckmann.pdf](https://www.usenix.org/system/files/conference/nsdi18/nsdi18-beckmann.pdf)

- github(src): [https://github.com/CMU-CORGI/LHD](https://github.com/CMU-CORGI/LHD)

  
An Imitation Learning Approach for Cache Replacement (opt attempt)
- https://arxiv.org/pdf/2006.16239.pdf


TinyLFU: A Highly Efficient Cache Admission Policy [Gil Einziger, 2015]
- https://arxiv.org/pdf/1512.00727.pdf
- **Main Contributions**:
	1. Novel LFU based **admission** scheme, works with arbitrary replacement policies.
	2.  Proposed a highly space efficient data structure for storing frequency cnt. <ins>They claim that such Counting Bloom Filter based structure can accurately mimic the frequency ordering under perfect LFU i.e. true access frequency distribution.</ins> 
	3.  Developed *W-TinyLFU* which is a optimized version of TinyLFU with LRU based eviction scheme. Multiple works show that *W-TinyLFU* tops out compare to many existing caching scheme. *W-TinyLFU* is implemented in **Caffeine**.
	4.  For skewed workloads and workloads with static distributions, the impact of eviction policy in caches with *TinyLFU* admission policy became marginal. <ins>They state that with even naive eviction scheme the cache perform similar to perfect-LFU. </ins> For dynamic distribution, the eviction policy does impact performance, but less profound compare to without admission policy. 
- **Notes and Implication**:
	1. Sec 2.1, backed the idea that perfect-LFU is an "optimal" online policy when access distribution is static. And, the In-cache LFU performs noticeably worse than perfect  LFU due to its inaccurate frequency distribution. (not actually optimal since there are many other factors not consider in this simplfied context).
	2. Most frequency based solution, use frequency as dominant factor. And use recency or *aging* to  handle dynamic access distribution. 
	3. Their novel CBF structure can be used to implement perfect-LFU like eviction scheme.
	4. TinyLFU Weakness:
		- The eviction victim is guarded by TinyLFU filter, referenced item can enter the cache only if its popular than the eviction victim. This mechanism can be both good and bad. This is essentially the part which weakened the impact of eviction decision. 
		- Under TinyLFU scheme frequency became dominant factor, recent works show that size distribution can have huge impact on cache performance. Thus, frequency dominant admission scheme might weaken the power of smart eviction scheme that uses item size.
	
- 	![TinyLFU OverView](https://github.com/JYang1997/Papers/blob/main/imgs/simpletinylfu.png)

### Expire related solution
Segcache: a memory-efficient and scalable in-memory key-value cache for small objects
(follow after twitter trace paper) (TTL)

- https://www.usenix.org/system/files/nsdi21-yang.pdf

Adaptive TTL-Based Caching for Content Delivery
(Expiration related solution for CDN cache 2017) (see arxiv for full work)
- [https://dl.acm.org/doi/pdf/10.1145/3078505.3078560](https://dl.acm.org/doi/pdf/10.1145/3078505.3078560)

## Cache MRC & Stack:

 

Mattson 1970

- [https://pdfs.semanticscholar.org/4a58/e3066f12bb86d7aef2776e9d8a2a4e4daf3e.pdf](https://pdfs.semanticscholar.org/4a58/e3066f12bb86d7aef2776e9d8a2a4e4daf3e.pdf)

Calculating Stack Distances Efficiently 2001
- [https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.87.9028&rep=rep1&type=pdf](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.87.9028&rep=rep1&type=pdf)

Estimating Cache Misses and Locality Using Stack Distances

- [http://polaris.cs.uiuc.edu/publications/p183-cascaval.pdf](http://polaris.cs.uiuc.edu/publications/p183-cascaval.pdf)


statCache 2003
- [http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.142.399&rep=rep1&type=pdf](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.142.399&rep=rep1&type=pdf)

Statstack 2010
- [https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.220.6752&rep=rep1&type=pdf](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.220.6752&rep=rep1&type=pdf)

 
CounterStack 2014

- [https://www.cs.ubc.ca/~nickhar/papers/CounterStacks/CounterStacks.pdf](https://www.cs.ubc.ca/~nickhar/papers/CounterStacks/CounterStacks.pdf)

- SRC: [https://github.com/jstol/counterstacks](https://github.com/jstol/counterstacks)

DFShards: Effective Construction of MRCs Online for Non-stack
- https://dl.acm.org/doi/pdf/10.1145/3457388.3458810
- **Addressed Issues**
	1.	 There is no current work on constructing MRC for non-stack policy. One way to construct MRC is to simulate fixed number of cache size. 
	2.	 The problem is simulating to many cache size is waste of resources. simulating not enough cache size result in inaccurate MRC.
- ** Contributions**
	1. This work dynamically adjust numbers and size of cache to simulates
	2. Utilize spatial sampling  to reduce simulation cost.  

## Caching Theory

Caching Is Hard—Even in the Fault Model
 - https://core.ac.uk/download/pdf/81860263.pdf

General Caching Is Hard: Even with Small Pages - Lukáš Folwarczný, 2015
- https://arxiv.org/pdf/1506.07905.pdf

Practical Bounds on Optimal Caching with Variable Object Sizes

- [http://www.cs.cmu.edu/~beckmann/publications/papers/2018.sigmetrics.foo.pdf](http://www.cs.cmu.edu/~beckmann/publications/papers/2018.sigmetrics.foo.pdf)

Optimal Data Placement for Heterogeneous Cache, Memory, and Storage Systems

-[https://www.ymsir.com/papers/chopt-sigmetrics.pdf](https://www.ymsir.com/papers/chopt-sigmetrics.pdf)

Efficient stack distance computation for priority replacement policies, 2011
(generalized min tree algorithm)
- [https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/2016604.2016607](https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/2016604.2016607)
- https://link-springer-com.services.lib.mtu.edu/content/pdf/10.1007/s10766-012-0200-2.pdf


Optimal On-Line Computation of Stack Distances for MIN and OPT∗

- [https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/3075564.3075571](https://dl-acm-org.services.lib.mtu.edu/doi/pdf/10.1145/3075564.3075571)


##  Caching analysis

## Relevant Data structure:

Cuckoo Filter: Practically Better Than Bloom

[https://www.cs.cmu.edu/~dga/papers/cuckoo-conext2014.pdf](https://www.cs.cmu.edu/~dga/papers/cuckoo-conext2014.pdf)

  
  

Xor Filters: Faster and Smaller Than Bloom and Cuckoo Filters

[https://arxiv.org/pdf/1912.08258.pdf](https://arxiv.org/pdf/1912.08258.pdf)

This data structure doesn’t seems like a online method (not designed for modified, delete), based on authors blog [https://lemire.me/blog/2019/12/19/xor-filters-faster-and-smaller-than-bloom-filters/](https://lemire.me/blog/2019/12/19/xor-filters-faster-and-smaller-than-bloom-filters/)

  
  

Multiple Set Matching and Pre-Filtering with Bloom Multifilters

[https://arxiv.org/pdf/1901.01825.pdf](https://arxiv.org/pdf/1901.01825.pdf)

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExNTgyMjc5NDksMTU4ODU0ODE1MSwtOT
gyNjE0NTg0LDExMzc1ODMxMDYsLTIxODYwNzUxNSwtODgwOTcx
NzYsLTE4MDI2OTA1OTEsMTQ3NzE1NDQ1Myw3NDU3ODU5MzMsLT
Y0MTI4OTg1LDYxNzc1OTUwNSwtODU1MDMxNjMzLDE4ODMzNzU5
NTAsMTc2NDU0MDQzOSw4NTU1MjQ5MTgsODU0NzIzNjgyLC0xMz
YyNzM2MDAwLC00NDUxMTQ5NDEsMTEzMTk0MDU2OSw2ODMzNjU1
MjBdfQ==
-->