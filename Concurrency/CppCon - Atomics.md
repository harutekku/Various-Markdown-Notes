# CppCon - Atomics

## Index
- [Definitions](#Definitions)
  - [Atomic operation](#Atomic-operation)
  - [Read-Modify-Write](#Read-Modify-Write)
- [Atomics](#Atomics)
  - [`std::atomic<T>`](#std::atomic<T>)
  - [Atomic types](#Atomic-types)
  - [Operations](#Operations)
    - [Special atomic operations](#Special-atomic-operations)
    - [Overloaded operators vs methods](#Overloaded-operators-vs-methods)
    - [Caveats](#Caveats)
  - [Speed](#Speed)
  - [Atomics vs Lock-free](#Atomics-vs-Lock-free)
  - [Waiting](#Waiting)
  - [Memory Guarantees](#Memory-Guarantees)
    - [Memorry Barriers](#Memory-Barriers)
    - [C++ Standard](#C++-Standard)
  - [Purpose](#Purpose)
- [Use cases](#Use-cases)
- [Examples](#Examples)
- [Author](#Author)

# Definitions

## Atomic operation
  - An operation that is guaranteed to execute as a single transaction. Other threads will see the state of the
    system before the operation started or after it finished, but _never_ in the intermediate state.
  - At the low level, atomic operations are special hardware instructions (hardware guarantees atomicity)
    - Atomicity is a general concept - see databases etc.

# Read-Modify-Write
  - How it works
    - The data - the entire cache line, not just a single variable - is fetched from the memory
    - It propagates throught the cache lines - the CPU interacts directly with L1 cache line
    - It's operated on by CPU
    - It dwindles throught the cache lines into the main memory
  - Reads and writes do not have to be atomic
    - They're for built-in types on x86 architecture

# Atomics

## `std::atomic<T>`
  - A class-wrapper around certain types providing them with atomic operations
    - Introduced in C++11
  - During the operation, a execution core gets exclusive access to the memory
  - A core accessing atomic memory is guaranteed _not to_ see the intermediate state
    - That is, as long as any other core does atomics!

## Atomic types
  - Any **trivially copyable** type can be used as a template parameter for `std::atomic<T>`
    - _Trivially copyable_ means it can be copied with `memcpy()`
    - Continuous block of memory
    - No virtual functions, `noexcept` constructor
    - Is a PODType

## Operations
  - Read and Write (assignment)
  - Special atomic operations
  - Depending on the type, there might be some extra atomic operations

### Special atomic operations
  - Increment and decrement for raw pointers
  - Addition, substraction and bitwise logic operations for integral types
  - `std::atomic<bool>` is valid, but doesn't have any special operations
  - `std::atomic<double>` is valid, no special operations and no atomic increment
  - Explicit reads and writes
    - `load()` gets a value atomically
    - `store()` stores a value atomically
  - Value exchange
    - `exchange()` gets an old value, stores a new value and returs the old value
  - Compare and swap
    - `compare_exchange_strong()` takes an expected value and desired value as arguments
      - If the value is not the expected value, it returns false and assigns current value to the first parameter.
      - Otherwise sets the new value and returns true.
        ```cpp
        // Possible implementation - pseudocode
        auto compare_exchange_strong(T& oldValue, T newValue) -> bool {
          T tmp = value;
          if (tmp != oldValue) {
            oldValue = tmp;
            return false;
          }
          _Lock L;               // Double-locking pattern, back from the grave
          tmp = oldValue;
          if (tmp != oldValue) {
            oldValue = tmp;
            return false;
          }
          value = newValue;
          return true;
        }
        ```
    - `compare_exchange_weak()` takes an expected value and desired value as argument
      - If the value is not the expected value, it returns false and assigns current value to the first parameter.
      - Otherwise _tries_ to set the new value and returns true.
      - Can **spuriously** fail if the lock has timed out
        ```cpp
        auto compare_exchange_weak(T& oldValue, T newValue) -> bool {
          T tmp = value;
          if (tmp != oldValue) {
            oldValue = tmp;
            return false;
          }
          _TimedLock L;
          if (!L.locked())
            return false;
          tmp = value;
          if (tmp != oldValue) {
            oldValue = tmp;
            return false;
          }
          value = newValue;
          return true;
        }
        ```
    - Key to the most lock-free algorithms

```cpp
std::atomic<int> x = 0;

int currentX = x.load();

// Key to the most lock-free algorithms - not wait free
while (!x.compare_exchange_strong(currentX, x * 2)); 
```
  - Fetch
    - `fetch_add()` adds atomically and returns the old value
    - `fetch_subs()` substracts atomically and returns the old value
    - `fetch_and()` bitwise ANDs and returns the old value
    - `fetch_or()` bitwise ORs and returns the old value
    - `fetch_xor()` bitwise XORs and returns the old value

### Overloaded operators vs methods
  - As a programmer you speak to two audiences - machines and other programmers
    - The compilers always understand what you mean. It's not necessarily what you meant to say
      but they always understand what you actually said
    - Programmmers tend to see what they thought you meant, not what you really meant
  - Overloaded operators
    - Make writing code easier and faster
    - But are more prone to errors
    - Expressions with atomic variables are not computed atomically
  - Methods
    - More verbose
    - But make atomic operations explicit
    - They call attention - _something weird is going on here_

### Caveats
  - No atomic multiply
  - `std::atomic<T>` where `T` is an integral type provides overloaded operators only for atomic operations
  - Any expression with atomic variables will not be computed atomically

## Speed
  - Performance should be measured
    - Measurement results will be hardware and compiler specific - don't over-generalize
    - Comparing atomic and non-atomic operations may be instructive for understanding of what hardware does,
      but is rarely useful
    - Comparing atomic operation with another thread-safe alternative is valid and useful
  - On x86, atomic operations are usually slower than non-atomic operations
  - On x86, atomic operations are usually faster than `std::mutex`, but still a bit slower than spinlocks

## Atomics vs Lock-free
  - Atomics **are not always lock-free**
  - Results are run-time and platform-dependent
    - `std::atomic<T>::is_lock_free()` checks if type `T` is lock free at runtime
    - `std::atomic<T>::is_always_lock_free()` checks if type `T` is lock free at compile time,
      but it may not be lock-free at run time (usually because of the allignment AND padding)
  
## Waiting
  - Even though atomic variables can be lock-free and maybe even wait-free,
    but that does not mean they don't wait for each other.
  - Cache line sharing - if two cores want to access memory from the same cache line, they must wait for each other
    - There's no lower granularity on cache access than 64 bytes (on x86)
  - Wait-free has nothing to do with time
    - Wait-free refers to the number of compute operations
    - Operations don't have to be of the same duration
  - Atomic operations do wait on each other
    - Write operations and cache line access especially
    - False sharing can occur
      - You can avoid it by aligning per-thread data to separate cache lines
    - Price of sharing data without races
    - Read-Only operations can scale near-perfectly

## Memory Guarantees
  - For acquiring exclusive access - data may be prepared by other threads, must be completed.
  - For releasing into shared access - data is prepared by the owner thread, must become visible

### Memory Barriers
  - Memory barriers control how changes to memory made by one CPU become visible to other CPUs
    - It's a global control, accross all CPUs, provided by the hardware
    - Memory barriers are invoked through processor-specific instructions,
      although it's usually just an attribute to R/W instruction
  - Memory barriers are expensive - may be even more than atomic operations
    - Not all platforms provide all barriers
  - Visibility of non-atomic changes is not guaranteed

### C++ Standard
  - Prior to C++11 - no portable memory barriers
  - Since C++11 - standard memory barriers, closely related to memory order
    - `std::memory_order_relaxed` gives no guarantees of order of accesses
    - `std::memory_order_acquire` guarantees that all memory operations scheduled after the barrier in
      the program order become visible after barrier. Operations can not be reordered.
    - `std::memory_order_release` guarantees that all memory operations scheduled before the barrier in
      the program order become visible before barrier. Operations can not be reordered.
    - `std::memory_order_acq_rel` combines acquire amd release barriers. No operation can move
      accross the barrier.
    - `std::memory_order_seq_cst` removes the requirement and establishes single total modification
      order of atomic variables
  - Sequential consistency is enforced by default as a strictest memory order. 
    C++ guards you from mistakes, but it's not always necessary.
  - Releaser - Acquirer pattern

## Purpose
  - Atomic variables are hardly ever used by themselves
  - Usefull for implementing data structures that are hard to implement with locks
  - Atomics are used to get exclusive access to memory
  - Atomics are used to reveal memory to other threads

# Use cases
  - High-performance concurrent lock-free data structures
  - Data structures that are difficult or expensive to implement with locks (lists, trees)
  - When drawbacks of locks are important (deadlocks, priority conflicts, latency problems)
    - Be explicit, know what you're doing
  - When concurrent synchronization can be achieved by the cheapest atomic operations

# Examples
```cpp
// Atomic queue - the atomic variable is used as an index
int array[128];
std::atomic<size_t> front{ 0 };

auto Push(int value) -> void {
    size_t slot = front.fetch_add(1);
    array[slot] = value;
}

// Atomic list - the atomic variable is used as a pointer to list's head
struct Node {
    int Value;
    Node* Next;
};

std::atomic<Node*> head;

auto PushFront(int value) -> void {
    Node* newNode = new Node();
    newNode->Value = value;
    Node* oldHead = head.load();
    do {
        newHead->Next = oldHead;
    } while (!head.compare_exchange_strong(oldHead, newNode))
}
```

# Credits
  - [CppCon 2017: Fedor Pikus “C++ atomics, from basic to advanced. What do they really do?”](https://www.youtube.com/watch?v=ZQFzMfHIxng)

# Author
- Harutekku