# [KPart: A Hybrid Cache Partitioning-Sharing Technique for Commodity Multicores](https://ieeexplore.ieee.org/document/8327002/)

###### Nosayba El-Sayed, Anurag Mukkara, Po-An Tsai, Harshad Kasture, Xiaosong Ma, Daniel Sanchez

---

### What is the Problem? [Good papers generally solve *a single* problem]

Although cache partitioning is now available in commercial hardware, current systems only implement **way-partitioning**, which is coarse (divide the few cache ways among partitions) and limited in number (more partition leads to lower associativity). So current systems can't benefit a lot from cache partitioning.

Instead of proposing new cache partitioning techniques which current hardware can't afford, this paper want to improve performance based on prior **utility-based cache partitioning (UCP)** techniques.

### Summary [Up to 3 sentences]

KPart groups applications into clusters and partitions the cache among clusters.

By either offline or online profiling, KPart uses hierarchical clustering on performance loss estimation caused by sharing a cache partition (estimated by profiling results).

In the end, KPart use UCP's Lookahead algorithm to determine thr partition size for every cluster.

### Key Insights [Up to 2 insights]

- when doing cache partitioning, a little sacrifice on cache-insensitive (e.g. streaming) applications can help improve cache-sensitive applications a lot
- not only limited to cache miss information (miss curves), richer profiling information (IPC curves, memory bandwidth curves, obtained by miss curves + performance counter) is better
  - due to prefetching and memory-level parallelism, an application can have large reductions in cache misses but little to not change in IPC
  - some partition plans may provide better IPC speedup, but they also cause memory bandwidth saturated -- actual performance gain is not as good as expected

### Notable Design Details/Strengths [Up to 2 details/strengths]

- Once finished, hierarchical clustering can generate plan for any given number of clusters. Just traverse any possible number of clusters to **automatically** choose the number K with best estimated speedup.
- Optimizations for online profiling per application
  - Sample few cache allocations and do interpolation.
  - Only warmup once, and sample decreasing partition sizes. Thus shorten the wait time required between two successive sampling.

### Limitations/Weaknesses [up to 2 weaknesses]

- No isolation within each cluster. Thus, not suitable for performance-critic cache-sensitve applications, which shouldn't be grouped with other cache-sensitive applications.
- In a system with hundreds of cores, KPart will suffer from long online profiling phases.

### Summary of Key Results [Up to 3 results]

- The automatically chosen number of clusters, $K_{auto}$, is pretty close to the real best one $K_{best}$.
- Workloads dominated by cache-insensitive applications benefit more from KPart.
- In simulation, KPart can achieve most of the performance benefits of another fine-grained partitioning technique (unavailable in current hardware)

### Open Questions [Where to go from here?]

- Efficient online hardware monitoring (profiling)
- Except memory bandwidth, KPart can combine with more sophisticated slowdown estimation techniques.
- Can OS integrates this functionality to allocate cache dynamically?