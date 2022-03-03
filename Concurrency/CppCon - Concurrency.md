# CppCon - Concurrency

## Index
- [Definitions](#Definitions)
  - [Concurrency](#Concurrency)
  - [Parallelism](#Parallelism)
- [C++ Standard](#C++-Standard)
  - [Issues before C++11](#Issues-before-C++11)
  - [Memory model](#Memory-model)
  - [Static thread-safe initialization](#Static-thread-safe-initialization)
- [Threading](#Threading)
  - [Creating a thread](#Creating-a-thread)
  - [Joining finished threads](#Joining-finished-threads)
  - [Result of thread's execution](#Result-of-thread's-execution)
  - [Data races](#Data-races)
- [Logical synchronization](#Logical-synchronization)
  - [Busy-wait](#Busy-wait)
- [Basic synchronization primitives](#Basic-synchronization-primitives)
  - [`std::mutex`](#`std::mutex`)
  - [`std::lock_guard<T>`](#`std::lock_guard<T>`)
  - [`std::unique_lock<T>`](#`std::unique_lock<T>`)
  - [`std::scoped_lock<T...>`](#`std::scoped_lock<T...>`)
- [Advances synchronization primitives](#Advances-synchronization-primitives)
  - [`std::condition_variable`](#`std::condition_variable`)
    - [`std::mutex` and `std::condition_variable`](#`std::mutex`-and-`std::condition_variable`)
  - [`std::once_flag`](#`std::once_flag`)
  - [`std::shared_mutex`](#`std::shared_mutex`)
  - [`std::counting_semaphore<N>`](#`std::counting_semaphore<N>`)
  - [`std::latch`](#`std::latch`)
  - [`std::barrier<>`](#`std::barrier<>`)
- [Alternatives](#Alternatives)
  - [`std::future<T>` and `std::async()`](#`std::future<T>`-and-`std::async()`)
- [Credits](#Credits)
- [Author](#Author)

# Definitions

## Concurrency
  - Concurrency means doing two things concurrently - "running together".
  - Concurrency is a software problem
  
## Parallelism
  - Parallelism means doing two things in parallel
  - Parallelism is a hardware problem

# C++ Standard

## Issues before C++11
  - Standard couldn't say anything about multithreaded programs
  - Are optimizations legal in multithreaded programs?
  - Hardware can reorder accesses

## Memory model
  - Now a program consists of one or more _threads of execution_
  - Every write to a single memory location must _synchronize-with_ all
    other reads or writes of that memory location, or else program has UB
  - _Synchronizes-with_ relationships can be established by using various standard library
    facilities, such as `std::mutex` and `std::atomic<T>`

## Static thread-safe initialization
  - The first thread to arrive will start initializing the static instance
  - Any other thread will block and wait until the first thread either succeeds and unblocks others
    or fails with an exception, unblocking one of them.

# Threading

## Creating a thread
  - Prior to C++11, you would rely on external libraries, such as pthreads
  - Since C++11, the standard library owns the notion of creating new threads

```cpp
std::thread t{
    []() -> void {
        puts("Thread t");
    }
};
```

## Joining finished threads
  - The new thread starts executing immediately.
  - When its task is done, the thread becomes _joinable_
  - Call `join()` on `std::thread` object before destroying it. This call will block,
    if necessary, until the other thread's job is finished.

```cpp
std::thread t{
    []() -> void {
        puts("Thread t");
    }
};

puts("Thread main");
t.join();
```

## Result of thread's execution
  - Threads don't have explicit results
  - Joining a thread is a synchronization operation

```cpp
int32_t i = 0;

std::thread t{
    [&]() -> void {
        puts("Thread t");
        result = 69;
    }
};

puts("Thread main");
t.join();
printf("i: %d\n", result);
```

  - In the above example, there are no data races. A write is synchronized with a read.

## Data races
  - A data race is a memory error occurring when two or more threads try to access the same memory location at the same time
    and at least one of these accesses is an attempt to write.
  - Data races on any type is an undefined behavior.
  - Example

```cpp
int32_t i = 0;

std::thread t{
    [&]() -> void {
        ++i;        // It's impossible to know whether this access or the next one will be first
    }
};

++i;

t.join()
printf("%d\n", i);
```

  - The data race in the above example can be fixed with C++11 `std::atomic<T>` types

```cpp
std::atomic<int32_t> i = 0;

std::thread t{
    [&]() -> {
        ++i;
    }
};

++i;

t.join();
printf("i: %d\n", i)
```

  - This fixes the physical data race. Every access to a `std::atomic<T>` implicitly synchronizes with every other access to it
    on the hardware level.
  - There's still a _semantic_ data race - different valid executions will produce different outputs. While it's not a UB,
    it might be considered a bug. It depends on how threads are scheduled

# Logical synchronization
  - Problems
    - The thread starts executing immediately. But what if we need to set up a bit more state before
      the execution?
    - Can we tell the other threads to wait until on thread unlocks them?

```cpp
std::thread t{
    [&]() -> void {
        WaitUntilFree();
        puts("Thread t");
    }
};

puts("Thread main");
FreeThreadt();
t.join();
puts("Thread main");
```

  - It's possible with synchronization primitives
  - But first, a retard-tier solution!

## Busy-wait

```cpp
std::atomic<bool> ready = false;

std::thread t{
    [&]() -> void {
        while (!ready);
        puts("Thread t");
    }
};

puts("Thread main");
ready.store(true);
t.join();
puts("Thread main");
```

  - There are several problems with the above example
    - Thread `t` is wasting CPU cycles. It never goes to sleep, it atomically loads the value from `ready` variable,
      checks it and does a jump.
    - Theoretically, the compiler can see that if this thread never sleeps then the value of `ready` will never change and
      hoist the test out of the loop. The value of `ready` can be `mov`ed into a register for optimization purposes.
    - Even though `ready` is a `std::atomic<bool>`, there's still UB
  - **DON'T DO THIS**
  - Solution? `std::mutex`

# Basic synchronization primitives

## `std::mutex`
  - Mutex is a mutual exclusion mechanism
  - Mutex allows the current thread of execution to gain exclusive access to the unique resource
    - `lock()` method acquires the unique resource
    - `unlock()` method releases the unique resource
  - A `std::mutex` is a resource - it must be `unlock()`ed once you're done with it

```cpp
std::mutex mtx;
mtx.lock();              // 1. Thread main acquires the mutex

std::thread t{
    [&]() -> void {
        mtx.lock();      // 2. Thread t tries to acquire the mutex, but fails and goes to sleep.
                         // 4. After thread main released the mutex, thread t can now lock it
        mtx.unlock();    // 5. Thread t releases the mutex because it's no longer needed
        puts("Thread t");
    }
};

puts("Thread main");
mtx.unlock();            // 3. Thread main releases the mutex
t.join();
puts("Thread main");
```

  - `std::mutex` is commonly used to protect some resource against multithreaded access
  - Protection **must be complete**
    - Exception: constructor and destructor

```cpp 
class NumberPool {
public:
    auto GetNumber() -> int32_t {
        _mtx.lock();
        if (_data.empty())
            _data.push_back(0);              // What if there was an exception?
                                             // The exception would propagate up the callstack, but the mutex would remain locked.
        int32_t retVal = _data.back();
        _data.pop_back();
        _mtx.unlock();                       // If there was an exception, unlock() would never be called
        return retVal;
    }

    auto NumbersAvailable() const -> size_t {
        _mtx.lock();                         // If there wasn't a call to lock(), there might have been a data race
        size_t retVal = _data.size();
        _mtx.unlock();
        return retVal;
    }

private:
    std::vector<int32_t> _data;
    mutable std::mutex _mtx;
};
```

## `std::lock_guard<T>`
  - A RAII class template taking as a parameter a mutex
  - The constructor locks the given mutex, and stores a reference to it
  - Its destructor unlocks the mutex
  - Can be used with C++17's CTAD

```cpp
auto NumberPool::GetNumber() -> int32_t {
    std::lock_guard<std::mutex> lk{ _mtx };
    if (_data.empty())
        _data.push_back(0);
    int32_t retVal = _data.back();
    _data.pop_back();
    return retVal;
}

auto NumberPool::NumbersAvailable() const -> size_t {
    std::lock_guard lk{ _mtx };
    return _data.size();
}
```

## `std::unique_lock<T>`
  - Analogue of `std::unique_ptr<T>` that helps with the management of the unique ownership of `std::mutex` locks
  - Just like `std::unique_ptr<T>`, `std::unique_lock<T>` is movable but not copyable

## `std::scoped_lock<T...>`
  - The new and improved `std::lock_guard<T>`, parametrized on multiple mutexes
  - Internally, it uses an algorithm based on address sorting to determine the order of locking the mutexes
  - Introduced to solve deadlock issues
  - Rarely used, but useful when needed

```cpp
auto NumberPool::NumbersAvailable() const -> size_t {
    std::scoped_lock lk{ _mtx };                              // std::scoped_lock<std::mutex>
    return _data.size();
}

auto NumberPool::MergeNumbersFrom(NumberPool& rhs) -> void {
    std::scoped_lock lk{ _mtx, rhs._mtx };                    // std::scoped_lock<std::mutex, std::mutex>
    _data.insert(std::begin(rhs._data), std::end(rhs._data));
}
```

# Advances synchronization primitives

## `std::condition_variable`
  - A condition variable allows to notify waiting threads on changes
    - `notify_one()` notifies one waiting thread
    - `notify_all()` notifies all waiting threads
    - `wait()` puts thread into the sleep
  - Internally, `wait()` will relinquish the lock and go to sleep. Once the thread wakes up,
    it'll re-acquire the lock.
  - It's all atomic

```cpp
class NumberPool {
public:
    auto ReturnNumber(int32_t number) -> void {
        std::unique_lock lk{ _mtx };
        _data.push_back(number);
        lk.unlock();
        _cv.notify_one();                       // Wake up one waiting thread
    }

    auto GetToken() -> int32_t {
        std::unique_lock lk{ _mtx };
        while (_data.empty())
            _cv.wait(lk);                       // Put the thread into sleep
        int32_t retVal = _data.back();
        _data.pop_back();
        return retVal;
    }

private:
    std::vector<int32_t> _data;
    mutable std::mutex _mtx;
    std::condition_variable _cv;
};
```

### `std::mutex` and `std::condition_variable`
  - Whenever you have a producer and a consumer pattern
    - Where consumer waits for the producer
    - Where production and consumption happen **multiple** times
  - Then you almost certainly want a `std::mutex` and `std::condition_variable`
  - If consumption and production happen only **once**, consider `std::promise` and `std::future` with `std::async`
  - If you can, prefer high-level frameworks 

## `std::once_flag`
  - Mimics how C++ does static initialization, but for non-static variables.
  - `std::call_once()` is guaranteed to execute only once on a given `std::once_flag`

## `std::shared_mutex`
  - Read/Write lock
    - `lock_shared()` acquires a "reading" lock
    - `unlock_shared()` releases a "reading" lock

## `std::counting_semaphore<N>`
  - A container of "poker chips"
    - `acquire()` gets a chip
    - `release()` returns a chip acquired previously to the pool, else it's UB
  - Chips are indistinguishable, interchangeable and not tied to any particular thread
    - Something like a `NumberPool`

```cpp
class Auction {
public:
    auto Cancel() -> void {
        _bids.acquire();
    }

    auto Bid() -> void {
        _bids.release();
    }

private:
    std::counting_semaphore<256> _bids{ 0 }
};
```

## `std::latch`
  - A `std::latch` is kind of like a semaphore - it has an integer counter that starts positive and counts down toward zero
    - `wait()` blocks until the counter reaches zero
    - `count_down()` decrements the counter
    - `arrive_and_wait()` decrements and begins waiting
  - Use `std::latch` as a one-shot "starting gate" mechanism.
  - It's impossible to reset a latch - use `std::barrier<>` if you want reusable latch

```cpp
std::latch l{ 1 };

std::thread t{
    [&]() -> void {
        l.wait();
        puts("Thread t");
    }
};

puts("Thread main");
l.arrive();
t.join();
puts("Thread main");
```

## `std::barrier<>`
  - A barrier is a resettable latch
    - `wait()` blocks until the counter reaches zero
    - `arrive()` decrements the counter
      - If counter reaches zero, it unblocks all the waiters
        and atomically resets the counter, thus starting a new cycle
    - `arrive_and_wait()` decrements and waits
  - Use `std::barrier<>` as a "pace car" mechanism.

```cpp
std::barrier bar{ 2, [] -> void { puts("Go!"); } };

std::thread t{
    [&]() -> void {
        puts("Thread t is setting up");
        bar.arrive_and_wait();
        puts("Thread t is running");
    }
};

puts("Thread main is setting up");
bar.arrive_and_wait();
puts("Thread main is running");
t.join();
```

# Alternatives

## `std::future<T>` and `std::async()`
  - `std::async` starts a new thread, executes a callable and immediately returns a `std::promise`
  - `std::future<T>::get()` blocks until thread finishes
  - Internally, it's implemented with `std::mutex` and `std::condition_variable`

```cpp
std::future<int32_t> f1 = std::async([]() -> int32_t {
    return 69;
});

int32_t res = f1.get();
```

# Credits
  - [Back to Basics: Concurrency - Arthur O'Dwyer - CppCon 2020](https://www.youtube.com/watch?v=F6Ipn7gCOsY)

# Author
- [Harutekku](https://github.com/harutekku)