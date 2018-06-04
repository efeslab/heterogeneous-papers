# [NetCache: Balancing Key-Value Storeswith Fast In-Network Caching](https://dl.acm.org/citation.cfm?id=3132747.3132764)

###### Xin Jin, Xiaozhou Li, Haoyu Zhang, Robert Soulé, Jeongkeun Lee, Nate Foster, Changhoon Kim, Ion Stoica
---

### What is the Problem? [Good papers generally solve *a single* problem]

Modern Internet services critically depend on high-performance Key-Value Store. One major challenge in scaling a KVS is coping with skewed, dynamic work loads, which can lead to servere load imbalance and significant performance degradations. Traditional flash-based or disk-based KVS can be balanced by a in memory caching layer. However to achieve high-performance, in-memory KVS has been deployed widely. **In memory** cache can't balance **in memory** KVS.

### Summary [Up to 3 sentences]

Just implement a KVS in the programmable switch, use it as a in network load-balancing write-through cache.

NetCache exploits a theoretical result that caching only $O(N \cdot log N)$ is sufficient to balance the load for a hash-partitioned KVS cluster with $N$ servers.

It identifies hot items by maintaining counters for each cached key, and a heavy-hitter detector (consists of a Count-Min sketch and Bloom filter) for uncached key.

### Key Insights [Up to 2 insights]

The top of rack (ToR) switch is particularly suitable for such a switch-based KVS cache:

- Hot queries for the whole KVS storage rack is enough to match the capability of programmable switch.
- ToR switch can achieve cache coherence easier than a higher-level switch (e.g. ToR aggregation switch). 

- Only need to program ToR switch, other parts of the network are unmodified. Then existing routing protocols and network functions are fully compatible.

### Notable Design Details/Strengths [Up to 2 details/strengths]

- Variable-Length On-Chip Key-Value Store (Section 4.4.2). Use indirect addressing (entries of lookup table record bitmap and index of value table) and dynamic memory management. It need a controller to collect memory when evict a key and allocate new memory when insert a new key. To optimize memory utility, periodic memory reorganization is also necessary.

- How to update cache:
  - If client writes to a cached key, switch invalidates the cache and forwards the write request to the server. The server will send `cache update` message later to update the hot key with new value
  - The controller compares hot keys reported by the heavy jitter detector with counters of cached keys. The value of a new inserting key is explicitly fetched from storage servers.

### Limitations/Weaknesses [up to 2 weaknesses]

This paper discusses limitations in section 5, I select the top two important:

- Can't handle highly-skewed, write-intensive workloads. Because it's a write through cache. While a write back cache will cause possible data loss due to switch failure.
- ToR cache can't deal with rack-level load imbalance. While it is difficult to deal with cache coherence and routing detours when implementing cache on higher-level switches.

### Summary of Key Results [Up to 3 results]

- Write requests do affect the performance significantly (especially when write visits the same hot-key as read queries). When write ratio > 10%, throughput will lose 50% or worse corresponding to skewness (how much write requests are skewed).
- Total system throughput grows as cache size increases. But the more skew the workload is, the less additional throughput will be improved when cache size becomes larger. In another word, larger cache will help if the workload is not so skewed. While a relatively small cache is enough if the workload is highly-skewed.
- NetCache reacts quickly to dynamic workloads. In three types of workloads change emulation (Hot-in, Random, Hot-out, see section 7.4 for detail), the scenario which hurts performance the most (need ~8 seconds to recover) is code keys suddenly becomes the hottest. But such radical changes are unlikely to happen frequently in practice.

### Open Questions [Where to go from here?]

corresponding to the limitations

- How to handle highly-skewed workloads. In another word, how to deal with possible data loss when using write back cache? I think how in-memory KVS deal with data loss (compared with flash-based or disk-based KVS) can provide some inspiration.

- How to implement cache on higher-level switches?
- The current programming API and compiler of programmable switches are low-level. Higher abstraction or even a OS is necessary.