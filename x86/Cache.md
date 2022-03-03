# Cache
## Intro
- A small amount of fast, expensive memory that lives inside and outside the CPU
- Divided into layers (L1, L2, L3 and sometimes L4)
- The second fastest memory type, outperformed only by the CPU registers

## Structure
- Caches are divided into rows, called _cache sets_ 
- Cache sets are divided into blocks of memory of fixed sizes, called _cache lines_ or _cache blocks_
- Cache blocks are indexed by $k$ bits of the $m$-bits wide mapped address
- Cache blocks _may_ contain a _block offset_ in case of multibyte cache lines, that is $o$ 
  the least significant bits of the 
  mapped address and indicates the offset within the block the data is stored at
- Cache blocks also contain unique IDs called _tags_ which are $m - k - o$ upper bits of the mapped addresses
- A _valid bit_ helps to determine whether the cache line is valid or not

## Locality
### Temporal
- The accessed address will most likely be accessed again in the near future
- Example are loops, accumulator variables

### Spatial
- The address nearby accessed address will most likely by accessed in the near future
  - The data structures usually span multiple memory blocks
- The cache lines are usually $N$ bytes wide

## Placement policies
### Direct-mapped cache
- Each main memory address maps to exactly one cache line
- Cache is organized into multiple sets with a single cache line per set - the cache is effectively 
  $N\cdot1$ column matrix

### Fully associative cache
- Permits data to be stored in any cache line
- Organized into a single cache set with multiple cache lines
- Since there's no index, the tag needs to store the full address as an ID
- Effectively a $1\cdot M$ matrix

### Set-associative cache
- Divided into $N$ sets and each set contains $M$ cache lines
- Memory block is first mapped onto a set and then placed on the arbitrary cache lines
- For set with $2^{x}$ cache lines, the cache is said to be $2^{x}$-way associative
  - If cache has $2^{k}$ blocks and is 1-way associative, it's an equivalent of direct-mapped cache 
  - If cache has $2^{k}$ blocks and is $2^{k}$-way associative, it's an equivalent of fully associative cache
- Indexed with _set index_
- Effectively a $N\cdot M$ matrix

## Replacement policies
- The Least-Recently Used cache entry will be replaced (evicted) by the new entry on a cache miss
- There are many more replacement policies, like LIFO, FIFO, LFU, LFRU, ARC, CAR etc. but they
  are out of scope of this note

## Write policies
- Write-through cache forces all writes to update both the cache and the main memory
- Write-back cache only writes to the main memory, if the cache set needs to be replaced
  - Contains a _dirty bit_, to indicate inconsistency with the main memory
  - Miss penalty will not be applied until the execution of some subsequent instruction after write
- Write-no-allocate cache forces write operations to interact directly with memory, without intermediate cache update
- Write-allocate cache loads the newly written data into the cache

## Memory access
### Cache hit
- On read, the CPU sends a signal with the accessed address to cache controller
- If the lowest $k$ bits match the cache line index and upper $m$ bits match the cache line tag
  and the valid bit is set, then the data is fetched from the cache
- Access time of cached memory is extremely low

### Cache miss
- On the other hand, if the aforementioned conditions aren't met, the data must be fetched directly from
  memory and then copied into the cache, resulting in slower access time

## Performance metrics
- Hit time is how long it takes data to be sent from the cache to the CPU (fast)
- Miss penalty is the time needed to copy data from RAM to the CPU cache (very slow)
- Miss rate is the percentage of misses
- Average Memory Access Time is given by the formula $\text{AMAT} = t_{hit} + (\%_{miss} \cdot t_{miss})$

## Cache friendliness
- Repeated accesses to the same data
- Iterations over linear (vectorized) data structures, like flat arrays or row-wise representations of matrices

## Topology
- Look-Aside
- Look-Through
- Look-Backside

## Sources
- Lectures 14-19 from CSE378 on courses.cs.washington.edu
  - https://courses.cs.washington.edu/courses/cse378/09wi/lectures/lec14.pdf
  - https://courses.cs.washington.edu/courses/cse378/09wi/lectures/lec15.pdf
  - https://courses.cs.washington.edu/courses/cse378/09wi/lectures/lec16.pdf
  - https://courses.cs.washington.edu/courses/cse378/09wi/lectures/lec17.pdf
  - https://courses.cs.washington.edu/courses/cse378/09wi/lectures/lec18.pdf
  - https://courses.cs.washington.edu/courses/cse378/09wi/lectures/lec19.pdf

# Author
- [Harutekku](https://github.com/harutekku)