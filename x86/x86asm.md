# NASM

# Index
- [Chapter I](#Chapter-I)
  - [Number systems](#Number-systems)
  - [Memory](#Memory)
  - [CPU](#CPU)
    - [80x96 and frens](#80x86-and-frens)
    - [16-bit registers](#16-bit-registers)
    - [32-bit registers](#32-bit-registers)
  - [Modes](#Modes)
    - [16-bit Real Mode](#16-bit-Real-Mode)
    - [16-bit Protected Mode](#16-bit-Protected-Mode)
    - [32-bit Protected Mode](#32-bit-Protected-Mode)
  - [Interrupts](#Interrupts)
  - [ASSembly](#ASSembly)
    - [Instruction operands](#Instruction-operands)
    - [Basic instructions](#Basic-instructions)
    - [Directives](#Directives)
    - [Expressions](#Expressions)
  - [Why learning Assembly?](#Why-learning-Assembly?)
- [Chapter II](#Chapter-II)
  - [Integer representation](#Integer-representation)
    - [Signed magnitude](#Signed-magnitude)
    - [One's complement](#One's-complement)
    - [Two's complement](#Two's-complement)
  - [Sign Extension](#Sign-Extension)
    - [Decreasing the size](#Decreasing-the-size)
    - [Increasing the size](#Increasing-the-size)
  - [Arithmetic](#Arithmetic)
  - [Control instructions](#Control-instructions)
    - [Comparison instructions](#Comparison-instructions)
    - [Jump instructions](#Jump-instructions)
    - [Looping instructions](#Looping-instructions)
- [Chapter-III](#Chapter-III)
  - [Shifts](#Shifts)
  - [Bitwise operations](#Bitwise-operations)
  - [Uses of bitwise operations](#Uses-of-bitwise-operations)
  - [Speculative execution](#Speculative-execution)
    - [Example of avoiding branching](#Example-of-avoiding-branching)
  - [Endianness](#Endianness)
  - [Counting bits](#Counting-bits)
    - [Aproach I](#Aproach-I)
    - [Aproach II](#Aproach-II)
    - [Approach III](#Approach-III)
    - [Approach IV](#Approach-IV)
- [Chapter IV](#Chapter-IV)
  - [Indirect addressing](#Indirect-addressing)
  - [Subprograms](#Subprograms)
  - [The Stack](#The-Stack)
  - [CALL-RET mechanism](#CALL-RET-mechanism)
  - [Calling convention](#Calling-convention)
    - [Handling parameters](#Handling-parameters)
    - [Prologue and epilogue](#Prologue-and-epilogue)
    - [Local variables](#Local-variables)
  - [Modules](#Modules)
  - [C calling conventions](#C-calling-conventions)
    - [Registers](#Registers)
    - [Symbols](#Symbols)
    - [Parameters](#Parameters)
    - [Local variable address](#Local-variable-address)
    - [Returning values](#Returning-values)
  - [Other calling conventions](#Other-calling-conventions)
  - [C storage types](#C-storage-types)
- [Chapter V](#Chapter-V)
  - [Arrays](#Arrays)
  - [Indirect addressing](#Indirect-addressing)
  - [Multidimensional arrays](#Multidimensional-arrays)
  - [String instructions](#String-instructions)
    - [String memory IO](#String-memory-IO)
    - [Comparison](#Comparison)
  - [Instruction prefixes](#Instruction-prefixes)
- [Chapter VI](#Chapter-VI)
  - [IEEE-754](#IEEE-754)
    - [Format](#Format)
    - [Converting from decimal](#Converting-from-decimal)
    - [Special values](#Special-values)
    - [Issues](#Issues)
  - [A math coprocessor](#A-math-coprocessor)
  - [Floating-point instructions](#Floating-point-instructions)
    - [FPU stack management](#FPU-stack-management)
    - [Arithmetic](#Arithmetic)
      - [Addition](#Addition)
      - [Subtraction](#Subtraction)
      - [Multiplication](#Multiplication)
      - [Division](#Division)
    - [Comparison](#Comparison)
    - [Status word instructions](#Status-word-instructions)
    - [Utilities](#Utilities)
- [Extra instructions](#Extra-instructions)
- [Sources](#Sources)
- [Author](#Author)

# Chapter I

## Number systems
- Decimal
  - Digits from `0` to `9`
  - A system that we all know and love
- Binary
  - Digits `0` and `1`
  - A bit quartet is called a _nibble_
  - A bit octet is called a _byte_ (on AMD64)
- Hexadecimal
  - Digits from `0` to `9` and from `A` to `F`
  - A digit in hex maps directly to one nibble, hence to convert bin to hex, simply convert
    all the digits of the hexadecimal number to their binary representations
  - Two-digit hex number is equal in size to one byte
  - To multiply by `F` in hex, simply add a zero on the right side of the number
- Dividing by system base strips the rightmost digit

A helpful table

| HEX | BIN  | HEX | BIN  |
|-----|------|-----|------|
|  0  | 0000 |  8  | 1000 |
|  1  | 0001 |  9  | 1001 |
|  2  | 0010 |  A  | 1010 |
|  3  | 0011 |  B  | 1011 |
|  4  | 0100 |  C  | 1100 |
|  5  | 0101 |  D  | 1101 |
|  6  | 0110 |  E  | 1110 |
|  7  | 0111 |  F  | 1111 |

## Memory
Sections of memory in terms of size

|     Name    |   Size   |
|-------------|----------|
|     Byte    |  1 byte  |
|     Word    |  2 bytes |
| Double Word |  4 bytes |
|  Quad Word  |  8 bytes |
|  Paragraph  | 16 bytes |

- Every byte in memory has an address

## CPU
- _Machine language_ are the instructions directly executed by the CPU
- _Registers_ are special memory areas in the CPU allowing for fast access to data

### 80x86 and frens
| CPU          | Registers                                                                                  | Modes                      | Features                             |
|--------------|--------------------------------------------------------------------------------------------|----------------------------|-------------------------------------|
| 8808/8086    | AX, BX, CX DX, SI, DI, BP, SP, CS, DS, SS, ES, IP, FLAGS                                   | 16-bit Real Mode only      | Segmented memory - 64K per segment   |
| 80286        | AX, BX, CX DX, SI, DI, BP, SP, CS, DS, SS, ES, IP, FLAGS                                   | 16-bit Protected/Real Mode | Segmented memory - new opcodes       |
| 80386        | EAX, EBX, ECX, EDX, ESI, EDI, EBP, ESP, EIP, SP, CS, DS, SS, ES, FS, GS, EFLAGS            | 32-bit Protected/Real Mode | Segmented memory - new opcodes       |
| 80486        | EAX, EBX, ECX, EDX, ESI, EDI, EBP, ESP, EIP, SP, CS, DS, SS, ES, FS, GS, EFLAGS            | 32-bit Protected/Real Mode | Segmented memory - new opcodes       |
| Pentium MMX  | EAX, EBX, ECX, EDX, ESI, EDI, EBP, ESP, EIP, SP, CS, DS, SS, ES, FS, GS, EFLAGS, MMX0-MMX8 | 32-bit Protected/Real Mode | Segmented memory - execution speedup |

### 16-bit registers
| Registers      | Function                            | Description                                            | Purpose    |
|----------------|-------------------------------------|--------------------------------------------------------|------------|
| AX (AL, AH)    | Accumulator register                | For some operations, one of the operands must be in AX | General    |
| BX (BL, HB)    | Base register                       | Can be used for indirect addressing                    | General    |
| CX (CL, CH)    | Count register                      | Determines how many times some operations will be done | General    |
| DX (DL, DH)    | Data register                       | A convenient place to store data, IN/OUT port number   | General    |
| SP             | Stack pointer                       | Points at the top of the stack                         | Special    |
| BP             | Base pointer                        | Can be used for indirect addressing                    | Special    |
| SI             | Source pointer                      | Points to current character being read, BX/BP offset   | Special    |
| DI             | Destination pointer                 | Points to current character being written or an offset | Special    |
| CS             | Code segment                        | Points to memory location where program was loaded at  | Segment    |
| DS             | Data segment                        | Points to data segment                                 | Segment    |
| ES             | Extra segment                       | Can be used with DS when two segments must be accessed | Segment    |
| SS             | Stack segment                       | Points to stack segment                                | Segment    |
| IP             | Instruction pointer                 | Points to next instruction, relative to code segment   | Special    |
| FLAGS          | Flags                               | A collection of 1-bit values, indicating CPU state     | Special    |

### 32-bit registers
| 16-bit Registers | 32-bit Registers |
|------------------|------------------|
| AX               | EAX              |
| BX               | EBX              |
| CX               | ECX              |
| DX               | EDX              |
| SP               | ESP              |
| BP               | EBP              |
| SI               | ESI              |
| DI               | EDI              |
| CS               | CS               |
| DS               | DS               |
| ES               | ES               |
| SS               | SS               |
| IP               | EIP              |
| FLAGS            | EFLAGS           |

## Modes
### 16-bit Real Mode
- Limited to 1 megabyte of memory
- No protections whatsoever - program can write everywhere
- Valid addresses fit in range `[0x0; 0xFFFFF]`
- Physical address given by the formula $16 \cdot selector + offset$
  - Where $selector$ is stored in segment register
- Disadvantages of segmented addresses
  - A single selector can only reference 64K of memory. When more is needed, the program must be split into segments
  - There's no unique segmented address for each byte

### 16-bit Protected Mode
- Guarantees memory protection
- Selector value is an index into a _descriptor table_
- Each segment has an entry in the descriptor table, indicating if it's currently in the memory,
  where it is, access permissions etc.
- Uses virtual memory - segments not needed at the moment are just stored on the disk
- Disadvantages
  - Offsets are still 16-bit entities

### 32-bit Protected Mode
- Offsets are expanded to 32-bits, allowing access to 4 gigabytes of memory per segment
- Segments can be divided into 4K pages

## Interrupts
- A mechanism to handle processes interrupting the currently executed program
- When an interrupt occurs, the control is passed to _interrupt handler_. Interrupt handlers are responsible for dealing with the interrupt. 
  They are assigned an unique integral value and are stored in _interrupt vector_ at the beginning of physical memory that contains 
  their segmented addresses.
- External interrupts can be raised by I/O devices, while internal interrupts can be raised by errors or interrupt instructions. 
  The former ones are called _hardware interrupts_, the latter ones - _traps_ and _exceptions_.
- Most of the interrupt handlers return control back to the program (as if nothing happened), restoring the registers, 
  while exceptions generally abort the program.

## ASSembly
- _ASSembler_ is a program that reads a text file with assembly instructions and converts the assembly into machine code
- Every assembly instruction is an equivalent of a single machine instruction
- Different architectures have different assembly languages (i.e AMD64 and ARMv7, etc.)

### Instruction operands
| Type       | Explanation                                                                                                          |
|------------|----------------------------------------------------------------------------------------------------------------------|
| Register   | Refer directly to CPU registers                                                                                      |
| Memory     | Address of the data in memory, either constant or computed using registers; offset from the beginning of the segment |
| Immediate  | Fixed, hard-coded values, stored in the instructions themselves                                                      |
| Implied    | Not explicitly visible                                                                                               |

### Basic instructions
| Instruction signature | Description                                                                                             |    
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `mov dest,src`        | Copy the contents of `src` into `dest`. Both operands can't be of memory type and must be equal in size |
| `call function`       | Function call. See: Calling Convention for your OS                                                      |
| `byte`                | Size specifier. Can also be `word`, `dword`, `qword`, `tword`, `oword`, `yword`, `zword` or `ptr`       |


### Directives
- Assembler artifacts, commonly used to `%define` macros or inform assembler of something
  - Serve the same purpose as in C

| Directive              | Description                                                                                             |
|------------------------|---------------------------------------------------------------------------------------------------------|
| `equ`                  | Define a constant symbol                                                                                |
| `%define`              | Equivalent of C `#define` macro - define a constant symbol                                              |
| `%include`             | Equivalent of C `#include` macro - include symbols from file                                            |
| `d_`                   | Define and initialize data of type `_`                                                                  |
| `res_`                 | Define uninitialized storage of type `_`                                                                |
| `times`                | Repeat certain directive N times                                                                        |

### Expressions
| Expression             | Description                                     |
|------------------------|-------------------------------------------------|
| `$`                    | The beginning of the line containing expression |
| `$$`                   | The beginning of the current section            |
| `+` and `-`            | Addition and subtraction                        |
| `*` and `\`            | Multiplication and division                     |

## Why learning Assembly?
- Critical-performance code
- Optimizations (like vectorization) that a compiler can't do
- Direct access to hardware features otherwise obscured (abstracted) by programming languages
- To learn how computer works
- To learn how compilers and higher-level languages (C/C++) work

# Chapter II

## Integer representation
- Unsigned integers are represented in binary just as they are
- Signed integers can be represented in three different ways

### Signed magnitude
- The first bit is the sign bit
- The remaining bits are used for storing the actual number
- This approach presents several drawbacks
  - There are two valid representations of zero, i.e `1000000` and `00000000`
  - Arithmetic logic is complicated

### One's complement
- The first bit is the sign bit
- The remaining bits are logically negated bits of the old number
- There's one drawback
  - Again, there exist two valid values of zero, i.e `10000000` and `00000000`
- A handy note:
  - To find one's complement of a hex number, simply subtract each digit from `0xF`

### Two's complement
- To find two's complement of a number
  - Negate all its bits
  - Add one
- Arithmetic operations might produce a carry
- Two negations always produce the original number

## Sign Extension
### Decreasing the size
- For unsigned integers, the size can be decreased if and only if all the bits being removed are zeros
- For signed integers, the size can be decreased if and only if all the bits being removed are either all zeros or ones 
and the bit past the last removed bit must have the same value as the removed bits.

### Increasing the size
- For extending unsigned integers, all the new bits must be zero
- For extending signed integers, all the new bits must have the same value as the sign bit
- x86 provides a new instruction, `movzx` for unsigned integer extension
- x86 provides a new instruction, `movsx` for signed integer extension

| Instruction signature | Description                                                                                             |    
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `movzx dest,src`      | Copy the contents from smaller `src` to bigger `dest` and sign-extend the unsigned value                |
| `movsx dest,src`      | Copy the contents from smaller `src` to bigger `dest` and sign-extend the signed value                  |
| `cbw`                 | Convert `byte` from `al` to `word` and store the result in `ax`                                         |
| `cwd`                 | Convert `word` from `ax` to `dword` and store the upper 16 bits in `dx` and lower 16 bits in `ax`       |
| `cwde`                | Convert `word` from `ax` to `dword` and store the result in `eax`                                       |
| `cdq`                 | Convert `dword` from `eax` to `qword` and store the upper 16 bits in `edx` and lower 16 bits in `eax`   |

## Arithmetic

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `add dest,src`        | Add `src` to `dest` and store the result in `dest`                                                      |
| `adc dest,src`        | Add `src` and `CF` to `dest` and store the result in `dest`                                             |
| `sub dest,src`        | Subtract `src` from `dest` and store the result in `dest`                                               |
| `sbb dest,src`        | Subtract `src`and `CF` from `dest` and store the result in `dest`                                       |
| `mul src`             | Multiply unsigned `rax` by `src` and store the result in `rdx:rax`                                      |
| `imul dest,src`       | Multiply signed `dest` by `src` and store the result in `dest`                                          |
| `imul dest,src1,src2` | Multiply signed `src1` by `src2` and store the result in `dest`                                         |
| `div src`             | Divide unsigned `rdx:rax` by `src` and store the quotient in `rax` and the remainder in `rdx`           |
| `idiv src`            | Divide signed `rdx:rax` by `src` and store the quotient in `rax` and the remainder in `rdx`             |
| `neg src`             | Negate `src` by computing it's two's complement                                                         |
| `clc`                 | Clear carry flag (`CF`)                                                                                 |

- A common bug is to forget to initialize `rdx` (or `edx`, `dx`) before division

## Control instructions
### Comparison instructions

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `cmp dest,src`        | Subtract `src` from `dest` and compare against zero                                                     |

- `cmp` uses different logic for signed and unsigned comparison
  - If comparison of unsigned values results in 0, `ZF` flag is set and `CF` is unset. If the comparison results in a negative value,
    the `CF` flag is set and `ZF` flag is unset. Else, both flags are unset.
  - If comparison of signed values results in 0, `ZF` flag is set and `SF` and `OF` are unset. If the comparison results in a negative
    value, `ZF` is unset and `SF` $\neq$ `OF`. Else, `ZF` is unset and `SF` $=$ `OF`

### Jump instructions

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `jmp dest`            | Unconditionally jump to `dest`. `dest` can be a code label or address                                   |
| `short`               | Specifies short jump, to a label within a 128 bytes range                                               |
| `near`                | Specifies near jump, to a label within a segment. Can use either two or four bytes as a displacement    |
| `far`                 | Specifies a far jump, outside the current code segment.                                                 |
| `jz dest`             | Conditionally jump to `dest` if `ZF` flag is set                                                        |
| `jnz dest`            | Conditionally jump to `dest` if `ZF` flag is unset                                                      |
| `jo dest`             | Conditionally jump to `dest` if `OF` flag is set                                                        |
| `jno dest`            | Conditionally jump to `dest` if `OF` flag is unset                                                      |
| `js dest`             | Conditionally jump to `dest` if `SF` flag is set                                                        |
| `jns dest`            | Conditionally jump to `dest` if `SF` flag is unset                                                      |
| `jc dest`             | Conditionally jump to `dest` if `CF` flag is set                                                        |
| `jnc dest`            | Conditionally jump to `dest` if `CF` flag is unset                                                      |
| `jp dest`             | Conditionally jump to `dest` if `PF` flag is set                                                        |
| `jnp dest`            | Conditionally jump to `dest` if `PF` flag is unset                                                      |
| `je dest`             | Conditionally jump to `dest` if previous comparison of signed values yields zero                        |
| `jne dest`            | Conditionally jump to `dest` if previous comparison of signed values doesn't yield zero                 |
| `jl dest`, `jng dest` | Conditionally jump to `dest` if previous comparison of signed values yields negative value              |
| `jle dest`, `jng dest`| Conditionally jump to `dest` if previous comparison of signed values yields negative value or zero      |
| `jg dest`, `jnle dest`| Conditionally jump to `dest` if previous comparison of signed values yields positive value              |
| `jge dest`, `jnl dest`| Conditionally jump to `dest` if previous comparison of signed values yields positive value or zero      |
| `jb dest`, `jnae dest`| Conditionally jump to `dest` if previous comparison of unsigned values yields negative value            |
| `jbe dest`, `jna dest`| Conditionally jump to `dest` if previous comparison of unsigned values yields negative value or zero    |
| `ja dest`, `jnbe dest`| Conditionally jump to `dest` if previous comparison of unsigned values yields positive value            |
| `jae dest`, `jnb dest`| Conditionally jump to `dest` if previous comparison of unsigned values yields positive value or zero    |

- For "unsigned" jump instructions 'a' and 'b' stand for _above_ and _below_

### Looping instructions

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `loop dest`           | Decrement `rcx` and if `rcx` $\neq$ 0, branch to `dest`                                                 |
| `loope dest`          | Decrement `rcx` and if `rcx` $\neq$ 0 $\land$ `ZF` $=$ 1, branch to `dest`                              |
| `loopne dest`         | Decrement `rcx` and if `rcx` $\neq$ 0 $\land$ `ZF` $=$ 0, branch to `dest`                              |
| `loopz dest`          | Decrement `rcx` and if `rcx` $\neq$ 0 $\land$ `ZF` $=$ 1, branch to `dest`                              |
| `loopnz dest`         | Decrement `rcx` and if `rcx` $\neq$ 0 $\land$ `ZF` $=$ 0, branch to `dest`                              |

# Chapter III
## Shifts

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `shl dest,src`        | Logically shift left value in `dest` by `src` and store the result in `dest`. Adds zeros to the left    |
| `shr dest,src`        | Logically shift right value in `dest` by `src` and store the result in `dest`. Adds zeros to the right  |
| `shl dest,src`        | Arithmetically shift left value in `dest` by `src` and store the result in `dest`. Adds zeros           |
| `sar dest,src`        | Arithmetically shift right value in `dest` by `src` and store the result in `dest`. Adds sign bits      |
| `rol dest,src`        | Rotate left value in `dest` by `src` and store the result in `dest`. Adds popped bits to the right      |
| `ror dest,src`        | Rotate right value in `dest` by `src` and store the result in `dest`. Adds popped bits to the left      |
| `rcl dest,src`        | Rotate left value in `dest` and carry by `src` and store the result in `dest`                           |
| `rcr dest,src`        | Rotate right value in `dest` and carry by `src` and store the result in `dest`                          |

## Bitwise operations

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `and dest,src`        | Bitwise AND `dest` with `src` and store the value in `dest`                                             |
| `or dest,src`         | Bitwise OR `dest` with `src` and store the value in `dest`                                              |
| `xor dest,src`        | Bitwise XOR `dest` with `src` and store the value in `dest`                                             |
| `not dest,src`        | Bitwise NOT `dest` with `src` and store the value in `dest`                                             |
| `test src,dest`       | Bitwise AND `dest` with `src` and set the `SF`, `ZF` and `PF` flags accordingly                         |

## Uses of bitwise operations

| Operation          | What to do                             |
|--------------------|----------------------------------------|
| Turn on bit _i_    | OR the number with $2^{i}$             |
| Turn off bit _i_   | AND the number with $\neg2^{i}$        |
| Complement bit _i_ | XOR the number with $2^{i}$            |

## Speculative execution
- A technique that allows the CPU to execute several instructions in parallel, therefore making computations more efficient
- Processors usually try to predict which branches will be taken and which won't, but these predictions aren't always true
- To mitigate this, one can write code without branches, using certain assembly instructions
  - **Warning**: This approach is not always the most performant one. Before you decide whether solution is the best one
    for your application, ***always measure the performance***

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `set__ dest`          | Set `dest` to 1 if a condition is met; zero otherwise. The two letter code `__` is the same as for jumps|

### Example of avoiding branching
The C code finding the bigger of two numbers
```C
uint32_t max;
if (first < second)
    max = second;
else
    max = first;
```
can be rewritten without an `if`-clause as
```x86asm
mov rax,[second]  ; rax = *second
xor rbx,rbx       ; rbx = 0
cmp rax,[first]   ; second - first
setg bl           ; second - first > 0? 0x1 : 0x0
neg rbx           ; rbx = 0xFFFFFFFFFFFFFFFF | 0x0
mov rcx,rbx       ; rcx = rbx
and rcx,rax       ; rcx = *second | 0x0
not rbx           ; rbx = 0x0 | 0xFFFFFFFFFFFFFFFF
and rbx,[first]   ; rbx = 0x0 | *first
or rcx,rbx        ; rcx = rcx | rbx
```

## Endianness
- The order that individual bytes of multibyte data element is stored in memory
  - Big endian stores the most significant byte first and then the rest of the bytes
  - Small endian stores the least significant byte first and then the rest of the bytes
- The most important reason why Intel x86 is little endian, is backward compatibility
- You usually don't have to care because there's no way to inspect bits stored internally by the CPU, except when
  - Transferring data between devices with different endianness, such as PowerPC and x86
  - Dealing with UTF-16 and UTF-32 (UTF-8 doesn't have this problem)
  - When reading and writing raw bytes between memory and some multibyte data structure (i.e interpreting JVM `.class` files)

## Counting bits
### Aproach I
```x86asm
mov rax,[data]
mov rcx,64
xor rbx,rbx
while_label:
    shl rax,1
    adc rbx,0
    loop while_label
```

### Aproach II
```C
int32_t count = 0;
while (data != 0) {
    data &= (data - 1);
    ++count;
}
```
In NASM, that's just
```x86asm
mov rax,[data]
loop_label:
    cmp rax,0
    jz end_label
    mov rcx,rax
    dec rcx
    and rax,rcx    
    inc rbx
    jmp loop_label
end_label:
```

### Approach III
```C
// Summing bits
data = (data & 0x55u) + ((data >> 1) & 0x55u);
data = (data & 0x33u) + ((data >> 2) & 0x33u);
data = (data & 0x0Fu) + ((data >> 4) & 0x0Fu);
```

### Approach IV
```C
__asm__ volatile (
    "popcnt %0, %1"
    : "=r" (retVal)
    : "r" (in)
);
```
In NASM, that's just
```x86asm
popcnt rax,[data]
```

# Chapter IV

## Indirect addressing
- General purpose registers can be used as "pointers", i.e they can be dereferenced with `[]` operator

## Subprograms
- A subprogram is an independent, _reusable_ unit of code than can be used from different parts of the program
- In C land, that's just a routine/procedure/function.
- A _reentrant_ subprogram is a subprogram that doesn't contain self-modifying code and doesn't alter the state of global data
- A _recursive_ subprogram is a subprogram that calls itself either directly or indirectly. It can have a _termination condition_

## The Stack
- A _LIFO_ data structure, supported by most CPUs
- The two main instructions used for manipulating stack values are `push` and `pop`
- `ss` register specifies the address of the stack segment in memory (in 16-bit mode)
- `rsp` register specifies the address of the value on top of the stack
- Alignment matters

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `push src`            | Decrements `rsp` and stores the `src` on top of the stack                                               |
| `pushad`              | Push `eax`, `ecx`, `ebx`, `edx`, `esi`, `ebi` and `ebp` on top of the stack (only in IA-32 mode)       |
| `pop dest`            | Increments `rsp` and stores the element on top of the stack into `dest`                                 |
| `popad`               | Pop `eax`, `ecx`, `ebx`, `edx`, `esi`, `ebi` and `ebp` from the stack (only in IA-32 mode)             |

## CALL-RET mechanism
- A mechanism introduced in 80x86 for easier flow control
- Calling and returning is easier than doing jumps and return address calculations
- Calling and returning subprograms can be nested
- Before returning, the return address **must** be on top of the stack; everything else must be removed

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `call label`          | Make an unconditional jump to `label` and push the return address on top of the stack                   |
| `ret`                 | Pop off the return address and jump to it                                                               |

## Calling convention
- A _calling convention_ is a set of rules on how the data is passed between subprograms

### Handling parameters
- Parameters can be passed on the stack to the called subprogram
- The caller must get rid of them after the subprogram returns in C calling convention (in Pascal and `stdcall` it's a duty of the callee)
- Since the subprogram will usually change `rsp`, `rbp` register is used to save the original value of `rsp` and `rbp` is pushed on the stack
- Before returning, the original value of `rbp` must be restored
- Parameters can be accessed by adding a positive offset to the original `rsp` value stored in `rbp`

### Prologue and epilogue
- C calling convention requires subprogram to set up a _stack frame_ at the very beginning and destroy it before returning
- A _stack frame_ is an area on the stack specific for a given subprogram. Stack frame holds local variables, subprogram arguments,
  original `rbp` value and return address.
- To correctly set up a stack frame, callee must explicitly or implicitly define a _prologue_ and _epilogue_ of the subprogram.

```x86asm
subprogram:
    push rbp    
    mov rbp,rsp        ; Subprogram prologue - setting up a stack frame
    sub esp,LOCAL_SIZE ; Reserve a stack area for local variables
    ; ...
    pop rbp            ; Subprogram epilogue
    ret
```

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `enter size,0`        | Create a stack frame with buffer of `size` size for local variables                                     |
| `leave`               | Destroy a stack frame                                                                                   |

### Local variables
- Local variables are accessed by adding negative offsets to the `rbp` value
- The space for them can be managed by `add`ing and `sub`tracting the number of bytes from `rsp`

## Modules
- A _multi-module program_ is composed of more than one object file

| Declaration signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `extern`              | Indicate that the referenced symbol is located in different module                                      |
| `global`              | Make the symbol visible externally                                                                      |

## C calling conventions
### Registers
- Values of `rbx`, `rsp`, `rbp`, `r12`, `r13`, `r14` and `r15` must be preserved on the stack
  - In 16/32-bit mode the values of `edi`, `esi` and all segment registers must be preserved too
  - C uses `ebx`, `esi`, `edi` for `register` variables

### Symbols
- Function labels can't have an underscore as the first character on x86-64 Linux
  - On windows, function labels are prepended with the underscore

### Parameters
- Function parameters are passed in the `rdi`, `rsi`, `rdx`, `rcx`, `r8` and `r9`. Next parameters are passed on the stack in reverse order
  - In 16/32-bit mode parameters are passed only on the stack in the reverse order

### Local variable address

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `lea dest,[N * r + M]`| Calculates `[N * r + M]` where `N`, `M` are constants and `r` is a register and stores result in `dest` |

### Returning values
- Return values are stored in `rax` register. Otherwise, if it's 128-bit value, the upper 64 bits are stored in `rdx`. 
  Pointer values are stored in `rax` and floating-point values are stored in the `st0` register used by a math coprocessor
  - In 16/32-bit mode rules are exactly the same, but for 16/32-bit registers with the exception of math coprocessor (it just uses `st0`)

## Other calling conventions
- A calling convention can be specified in GNU GCC as an `__attribute__`
  - Under Microsoft compilers, you can use `__cdecl`, `__stdcall` and some other compiler intrinsics

| Calling convention     | Description                                     |
|------------------------|-------------------------------------------------|
| `cdecl`                | A standard C calling convention                 |
| `stdcall`              | A standard Pascal calling convention            |
| `regparm`              | Pass up to the 3 integer arguments in registers |

## C storage types

| Storage type | Description                                                                                                                |
|--------------|----------------------------------------------------------------------------------------------------------------------------|
| `<global>`   | Applies to variables defined outside any function scope. Accessible by everyone, exist during the entire execution         |
| `static`     | Stored at fixed memory location (either in .data or .bss segment), scoped only to the current function/module              |
| `auto`       | Applies to function-scoped variables. Allocated on the stack when invoking the function, deallocated on exit               |
| `register`   | _Hints_ the compiler to put variable into a register. Can only be integers and can't have their addresses taken            |
| `volatile`   | Tells the compiler that the value of the variable may change at any moment. Can't be optimized; always fetched from memory |

# Chapter V

## Arrays
- An _array_ is a contiguous block of list of data of the same type in memory.
- The element of the array can be accessed by its address or by an index added to the address of the first element
- Arrays can be either declared in .data or .bss sections or on the stack by subtracting the number of bytes to reserve from `rsp`
- Alignment matters

## Indirect addressing
The formula is:

$[base\_reg + factor \cdot index\_reg + constant]$

Where:
- $base\_reg$ is one of the general purpose registers, i.e `rax`, `rbx`, `rcx`, `rdx`, `rsp`, `rbp`, `rsi` or `rdi`
- $factor$ is either 1, 2, 4 or 8
- $index\_reg$ is one of the general purpose registers, i.e `rax`, `rbx`, `rcx`, `rdx`, `rbp`, `rsi` or `rdi`
- $constant$ is a 32-bit constant or a label

## Multidimensional arrays
- In _rowwise_ representation, multidimensional arrays are laid out flat in th memory; they're vectorized
- The position of the element denoted by indices $i_{1}$ to $i_{n}$ in the multidimensional array is

$\displaystyle \sum_{j=1}^{n} \left ( \prod_{k=j+1}^{n} D_{k} \right ) i_{j}$

Where:
- $D_{k}$ are dimensions of the array
- $i_{j}$ are element indices

## String instructions
- String instructions use index registers (`rsi` and `rdi`) for performing operations and incrementing both depending on `eflags` value
- The direction flag (`DF`) determines where the index registers are incremented or decremented
- It's a common bug to forget to set `DF`
- In real mode, some string instructions reference memory using `es:di` or `es:edi`
  - `es` should be initialized by the programmer

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `cld`                 | Clear `DF`. In this state, both index registers will be incremented                                     |
| `std`                 | Set `DF`. In this state, both index registers will be decremented                                       |

### String memory IO

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `lods_`               | Load 8/16/32/64-bit value into `al`/`ax`/`eax`/`rax` and move source index register                     |
| `stos_`               | Store 8/16/32/64-bit value from `al`/`ax`/`eax`/`rax` and move destination index register               |
| `movs_`               | Move 8/16/32/64-bit value from source index register to destination index register and move both        |

### Comparison

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `cmps_`               | Compare 8/16/32/64-bit value from source index register with a value in destination index register; move|
| `scas_`               | Compare 8/16/32/64-bit value from accumulator general purpose register with a value in `rdi` and move   | 

## Instruction prefixes
- An _instruction prefix_ is a special byte that when placed before an instruction, modifies its behavior

| Instruction prefix    | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `rep`                 | Repeat the next instruction a specified number of times. `rcx` is used to count the iterations          |
| `repe`, `repz`        | Repeat the next instruction a specified number of times while `ZF` is set and at most `rcx` times       |
| `repne`, `repnz`      | Repeat the next instruction a specified number of times while `ZF` is unset and at most `rcx` times     |
| `lock`                | Assert LOCK# processor signal. Turns some instructions into atomic ones                                 |

# Chapter VI

## IEEE-754
- A standard created by Institute of Electrical and Electronic Engineers used by most computers worldwide
- Effectively defines three formats
  - A _single-precision_ floating-point format (`float` in C)
  - A _double-precision_ floating-point format (`double` in C)
  - An _extended-precision_ floating-point format (`long double` on some implementations)
- The math coprocessor operates only on the extended-precision format

### Format

| Version | Sign  | Exponent | Fraction | Bias           |
|---------|-------|----------|----------|----------------|
| 32-bit  | 1-bit | 8-bit    | 23-bit   | $\text{0x7F}$  |
| 64-bit  | 1-bit | 11-bit   | 52-bit   | $\text{0x3FF}$ |

General formulas
- For 32-bit single-precision floating-point numbers

$\displaystyle F_{32} = (-1)^{S} \cdot 2^{E - B} \cdot \left ( 1 + \frac{1+F}{2^{23}} \right )$

- For 64-bit single-precision floating-point numbers

$\displaystyle D_{64} = (-1)^{S} \cdot 2^{E - B} \cdot \left ( 1 + \frac{1+F}{2^{52}} \right )$

### Converting from decimal
1. Find the sign bit
2. Convert both integer part and fraction part to binary. Integer part is converted normally, 
   while fraction part uses divide and multiply algorithm. Write them together as one number.
3. Shift the fraction dot up to the leftmost set bit. The number of places to shift is the true exponent
4. Add the exponent bias to the true exponent.
5. Write in order: sign bit, biased exponent and fraction part without the leading 1 padded at the end to 23/52-bits

### Special values

| Exponent        | Fraction | Description                                          |
|-----------------|----------|------------------------------------------------------|
| $E=0$           | $F=0$    | Denotes the number zero, either positive or negative |
| $E=0$           | $F\neq0$ | Denotes a denormalized number                        |
| $E=\text{0xFF}$ | $F=0$    | Denotes infinity, either positive or negative        |
| $E=\text{0xFF}$ | $F\neq0$ | Denots Not-a-Number (NaN) value                      |

### Issues
- Floating-point calculations result in the _approximate_ result
- Floating-point numbers should **never** be compared for equality
- A much safer way to compare floating point values is to check if they're smaller than a very small numeric constant (epsilon)

## A math coprocessor
- In the beginning, x86 had no support for floating-point operations; they were performed by subprograms on regular numbers
- Intel provided a math coprocessor built on x87 architecture, later integrated with the main x86 chip (since Pentium)
- x87 is still programmed like a separate chip - has its own instructions and registers (`st0`...`st7` and status) that act like "stack"
  - `st0` always points to the value on top of the stack
- Nowadays, MMX shares memory with math coprocessor

## Floating-point instructions
### FPU stack management

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `fld src`             | Load floating-point number from `src` on top of the math coprocessor stack                              |
| `fild src`            | Load integer from `src` on top of the math coprocessor stack                                            |
| `fld1`                | Store 0x1 on top of the stack                                                                           |
| `fldz`                | Store 0x0 on top of the stack                                                                           |
| `fst dest`            | Store the `st0` in `dest`                                                                               |
| `fstp dest`           | Store the `st0` in `dest` and pop the top value from the stack                                          |
| `fist dest`           | Store the `st0` converted into an integer in `dest`                                                     |
| `fistp dest`          | Store the `st0` converted into an integer in `dest` and pop the top value from the stack                |
| `fstcw dest`          | Store the FPU control word into `dest`                                                                  |
| `fldcw`               | Load the FPU control word from `dest`                                                                   |
| `fxch src`            | Exchange the value in `st0` and `src` on the stack                                                      |
| `ffree src`           | Free up a `src` register by marking as unused or empty                                                  |

### Arithmetic
#### Addition

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `fadd src`            | Add `src` to `st0`                                                                                      |
| `fadd dest,st0`       | Add `st0` to `dest`                                                                                     |
| `faddp dest`          | Add `st0` to `dest` and pop the top value from the stack                                                |
| `fiaddp src`          | Convert integer in `src` to float and add to `st0` and pop the top value from the stack                 |

#### Subtraction

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `fsub src`            | Subtract `src` from `st0`                                                                               |
| `fsubr src`           | Subtract `st0` from `src`                                                                               |
| `fsub dest,st0`       | Subtract `st0` from `dest`                                                                              |
| `fsubr dest,st0`      | Subtract `dest` from `st0`                                                                              |
| `fisub src`           | Convert integer in `src` to float, subtract it from `st0` and pop the top value from the stack          |
| `fsubp dest`          | Subtract `st0` from `dest` and pop the top value from the stack                                         |
| `fsubrp dest`         | Subtract `dest` from `st0` and pop the top value from the stack                                         |
| `fisubr src`          | Convert integer in `src` to float, subtract `st0` from it and pop the top value from the stack          |

#### Multiplication

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `fmul src`            | Multiply `st0` by `src`                                                                                 |
| `fmul dest,st0`       | Multiply `dest` by `st0`                                                                                |
| `fmulp dest`          | Multiply `dest` by `st0` and pop the top value from the stack                                           |
| `fimul src`           | Convert integer in `src` to float and multiply it by `st0`                                              |

#### Division

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `fdiv src`            | Divide `st0` by `src`                                                                                   |
| `fdivr src`           | Divide `src` by `st0`                                                                                   |
| `fdiv dest,st0`       | Divide `dest` by `st0`                                                                                  |
| `fdivr dest,st0`      | Divide `st0` by `dest`                                                                                  |
| `fdivp dest`          | Divide `dest` by `st0` and pop the top value from the stack                                             |
| `fdivrp dest`         | Divide `st0` by `dest` and pop the top value from the stack                                             |
| `fidiv src`           | Convert integer `src` to float and divide `st0` by it                                                   |
| `fidivr src`          | Convert integer `src` to float and divide it by `st0`                                                   |

### Comparison
- Comparison instructions modify $\text{C}_{0}$, $\text{C}_{1}$, $\text{C}_{2}$, $\text{C}_{3}$ flags of the FPU status word

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `fcom src`            | Compare `st0` with `src`                                                                                |
| `fcomp src`           | Compare `st0` with `src` and pop the top value from the stack                                           |
| `fcompp`              | Compare `st0` with `st1` and pop two top values from the stack                                          |
| `ficom src`           | Convert integer `src` to float and compare with `st0`                                                   |
| `ficomp src`          | Convert integer `src` to float, compare with `st0` and pop the top value from the stack                 |
| `ftst`                | Compare `st0` with 0                                                                                    |
| `fcomi src`           | Compare `st0` with `src` and modify `flags` register                                                    |
| `fcomip src`          | Compare `st0` with `src`, modify `flags` register and pop the top value from the stack                  |

### Status word instructions

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `fstsw dest`          | Store math coprocessor status word in `dest`                                                            |
| `sahf`                | Store `ah` register into `flags`                                                                        |
| `lahf`                | Load `flags` into `ah`                                                                                  |

### Utilities

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `fchs`                | Negate `st0`                                                                                            |
| `fabs`                | Computes the absolute value of `st0`                                                                    |
| `fsqrt`               | Compute square root of `st0`                                                                            |
| `fscale`              | Multiply `st0` by the $2^{\left \lfloor \text{ST1} \right \rfloor}$

# Extra instructions

| Instruction signature | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `cpuid`               | Put informations about CPU into the general-purpose registers basing on values in `rax` and `rcx`       |
| `popcnt dest,src`     | Count set bits in `src` and store the value in `rax`. Introduced _alongside_ SSE4a                      |
| `emms`                | Empty MMX state; quit MMX                                                                               |
| `nop`                 | Do nothing and be happy                                                                                 |
| `fnop`                | Do nothing and be happy, but on FPU                                                                     |

# Sources
- Paul A. Carter - "_[PC Assembly Language](https://pacman128.github.io/static/pcasm-book.pdf)_"
- Intel - "_[Intel 64 and IA-32 Architectures Software Developer's Manual](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf)_"

# Author
- [Harutekku](https://github.com/harutekku)