# [Devirtualizing Memory in Heterogeneous Systems](http://research.cs.wisc.edu/multifacet/papers/asplos18_dvm.pdf)

###### Swapnil Haria, Mark D. Hill, Michael M. Swift

---

### What is the Problem? [Good papers generally solve *a single* problem]

In ideal heterogeneous systems, accelerators want direct access to host physical memory (PM) to avoid address translation overheads, eliminate expensive data copying and facilitate fine-grained data sharing.

However, host physical memory is conventionally managed by virtual memory (VM), which provides protection and isolation but incurs high overheads.

How can we make it affordable for accelerators to share host memory while preserve useful properties (e.g. protection and isolation) of VM.

### Summary [Up to 3 sentences]

Eliminating address translation on most memory accesses by allocating most memory (page) subject to "virtual address (VA) is the same as physical address (PA)" ---- called Identity Mapping, which is transparent to applications and only requires modest OS changes.

IO memory management unit (IOMMU) sill need to verify if the progress holds valid permission to access an identity mapped page.

The Devirtualized Memory (DVM) can be further optimized by "compact page table" and "parallel preload read" or fallback to conventional address translation if identity mapping failed.

### Key Insights [Up to 2 insights]

- Supporting conventional VM requires MMU like page-table walkers and TLBs, which will complicate the design of accelerators, increase energy consumption. However such mechanism is too "virtualized" to modern conditions: high-performance systems are often have sufficient PM, thus no longer need swapping.

### Notable Design Details/Strengths [Up to 2 details/strengths]

- Store available contiguous permissions at a coarse granularity, result in a compact page table structure (smaller page table). The compact page table enables Access Validation Cache (AVC)  to cache all level Page Table Entries (PTEs). Thus AVC is more efficient than conventional TLB+PWC (Page Walk Cache).
- Preload on Reads, overlap memory access latency and Devirtualized Access Validation (DAV) latency. Will waste a little energy if the page is non-identity mapped.

### Limitations/Weaknesses [up to 2 weaknesses]

- To achieve Identity Mapping, DVM uses eager contiguous memory allocation, which will aggravate memory fragmentation problem and increase physical memory use if programs allocate memory more than they actually use.

- Can't handle copy-on-write (such as fork) gracefully, current solution just fallbacks to conventional VM.

### Summary of Key Results [Up to 3 results]

- Even the simplest DVM implementation outperforms conventional VM implementation. Further optimizations give smaller VM overheads.
- Replace TLB + PWC with AVC significantly decrease the energy consumption. Because AVC is 4-way associative while TLB is fully associative. And AVC cache all level PTEs while PWC excludes L1PTEs (possible no memory access V.S. at least one memory access if TLB misses).
- Authors imply that the risk of memory fragmentation with eager paging is negligible. They run several instances of benchmark to repeatedly allocate small/large chunks of memory with 16/32/64GB of total memory capacity.

### Open Questions [Where to go from here?]

- Be careful to timing channel attacks when implementing virtual memory, including DVM. This paper only mention such kind of potentional design flaws, no further discussion.
- The starting point of this work is: conventional VM is useful but too expensive to be implemented in accelerators. Considering all CPU and all kinds of accelerators need VM support to share memory. Why don't we put VM functionality into memory directly instead of seeking affordable VM solution between accelerators and CPU? Just make the system more "heterogeneous".
- I suspect the 3rd key result summary. They evaluate the risk of fragmentation by continuously allocate memory of variable sizes until identity mapping fails. The higher the percentage of allocated memory is, the lower the risk is. I doubt whether such "percentage" will reflect fragmentation well and whether their "variable sizes allocation" test cases resemble real cases.