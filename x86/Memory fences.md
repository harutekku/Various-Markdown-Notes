# Memory fences
## Hardware
### Background
- A CPU can reorder operations as it sees fit regardless of OOO (Load Buffer and Store Buffer)
- Ergo some operations can become globally (in RAM) visible in different order

### Intel manual
- Memory ordering refers to the order in which the processor issues loads and stores 
  through the system bus to system memory.
- Types
  - Program ordering - Stores and loads are issued on the system bus in order they occur in the instruction
    stream under all circumstances
  - Processor ordering - Allows performance enhancing operations such as allowing loads to go ahead
    of buffered stores. The goal is to increase instruction execution speeds, while maintaining memory coherency

### Rules
- Loads are not reordered with other loads
- Stores are not reordered with older loads
- Stores are not reordered with other stores
- **Loads might be reordered with older stores to different locations**

### Example
```x86asm
; Processor 0
;------------------------------
mov [x],1
mov rax,[y]

; Processor 1
;------------------------------
mov [y],1
mov rbx,[x]
```
- Because of hardware reordering, `cmp rax,rbx` can set `ZF` not necessarily because of the OOO

### Fixes
```C
void _mm_mfence(void);
#include <emmintrin.h>
```
```x86asm
mfence
```
- An Intel intrinsic/x86 SSE2 instruction that guarantees that every memory access that preceds, 
  in program order, the memory fence instruction is globally visible before any memory instruction 
  which follows the fence in program order

```x86asm
; Assumption
;   CPU 0 is faster than CPU 1
; Processor 0
;------------------------------
mov [x],1   ; This succeeds...
mfence      ; ...but must be published
mov rax,[y] ; Gets the old value of y, because it wasn't published yet. Value of x is now guaranteed to be 1

; Processor 1
;------------------------------
mov [y],1   ; This also succeeds...
mfence      ; ...but must be published
mov rbx,[x] ; Gets the newest value of x, because memory fence guarantees it was published
```

## Software
- Compiler is free to reorder operations as it sees fit as well
- This reordering can be prevented by the `__asm__ volatile("" : : : "memory")` in `gcc` and `clang`

# Credits
- [CoffeBeforeArch](https://www.youtube.com/channel/UCsi5-meDM5Q5NE93n_Ya7GA) - "_[Advanced Topics: Hardware Memory Barriers](https://youtu.be/nh9Af9z7cgE)_"
- Intel - "_[Intel 64 and IA-32 Architectures Software Developer's Manual](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf)_"

# Author
- [Harutekku](https://github.com/harutekku)