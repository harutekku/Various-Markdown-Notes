# GCC Inline Assembly
## Why
- Sometimes there's a need to place raw assembly, be it x86 or whatever else, inside a function
- Platform-specific optimizations
- A need to do something that stdlib doesn't allow

## Syntax
### AT&T syntax
- An alternative assembly syntax to Intel one
- Used by GNU `as` and inline assembly, since inline assembly is passed directly to the `as` for evaluation
- Rules
  - `<opcode> src, dest` instead of `<opcode> dest, src`
  - Register are prefixed with `%`, i.e. `%rax`, `%eip` or `%ax`
  - Immediate operands are prefixed with `$` and there's no `h` suffix for hexadecimal numbers, i.e. `$0x1` is valid while `1h` is not
  - Operand size is determined by instruction suffix, i.e. `b`, `w`, `l` and `ll` stand for byte, word, dword and qword
  - Addresses are referenced by the `section:offset(base_reg, index_reg, factor)`
    instead of `section:[base_reg + factor * index_reg + offset]`. `offset` and `factor` shouldn't be prefixed with `$`
- You can change syntax to Intel one using `.intel_syntax`, but you must remember to switch back to AT&T syntax using `.att_syntax`
  - You can also pass a `-masm=intel` flag to switch to Intel syntax entirely and forget about AT&T

### Basic syntax
- Both `asm` and `__asm__` syntaxes are allowed
- After both forms you can place a `volatile` keyword to prevent **compiler** optimizations
  - This doesn't prevent instruction rearrangements and OOO execution
- Between parentheses, you place instructions inside strings terminated with `\n`
  - Alternatively, you can use raw string literals `R"()"` in C++

```C
__asm__ (/*Instructions*/);
__asm__ volatile (/*Instructions*/);
```

### Extended syntax
- Since compiler has no idea what you put inside `__asm__` macro, you may clobber some registers you aren't supposed to
- You can prevent this by letting compiler know what exact registers you're using plus what variables you're referencing

```C
__asm__ (
      /*Instructions*/
    : /*Output operands*/
    : /*Input operands*/
    : /*Clobbered registers*/
);
```

- Operands must be prefixed with `%` and start from 0
- Operands must be lvalues
- Registers in the clobbered registers list must be prefixed with a single `%`
- Since GCC 3.1, you can use more readable labels, like

```C
int32_t data;
__asm__ (
  "mov $0x1, %[data]"
  : [data] "=r" (data)
);
```

## Constraints
- Restrictions on input and output operands and hints regarding their use

| Constraint | Description                                                                                                       |
|------------|-------------------------------------------------------------------------------------------------------------------|
| `r`        | Instruct compiler to keep operand in one of the GPR. Available GPRs are: `a`, `b`, `c`, `d`, `S`, `D`             |
| `m`        | Instruct compiler to perform operations directly on the memory                                                    |
| `N`        | Instruct compiler to treat variable as both input and output operand. `N` is a natural number in range `[0 : 10]` |
| `o`        | Instruct compiler to perform operations directly on the offsettable memory                                        |
| `M`        | Instruct compiler to perform operations directly on the non-offsettable memory                                    |
| `i`        | Instruct compiler to treat the operand as immediate integer                                                       |
| `n`        | Instruct compiler to treat the operand as immediate integer with known numeric value                              |
| `g`        | Instruct compiler to treat operand as an immediate integer, register or memory location. Doesn't allow non-GPRs   |

## Constraint modifiers
- You can modify constraints with these modifiers

| Modifier | Description                                                                                |
|----------|--------------------------------------------------------------------------------------------|
| `=`      | Treat operand as write-only                                                                |
| `&`      | Treat operand as early-clobbered operand, which is modified before instruction is finished |

# Credits
- Sandeep S. - "_[GCC-Inline-Assembly-HOWTO](https://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)_"
- OSDev.org - "_[Inline assembly](https://wiki.osdev.org/Inline_assembly)_"

# Author
- [Harutekku](https://github.com/harutekku)