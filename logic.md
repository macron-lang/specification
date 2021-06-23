# Logic
Logic is the base executable entity in Macron. Logic is made up of logic instructions. Instructions are executed in order except for branching. Logic macros may be used to generate raw logic.

An implementation must support bit widths 8, 16, 32, 64, and 128.

Logic instructions may take arguments. These are the possible argument types, where B is a supported bit width:
- `immB`: An immediate (constant) value consisting of B bits. Written signed or unsigned depending on the instruction.
- `regB`: A register with B bits. 8 registers are available. *(draft)*
- `memB`: A memory location dereference with B bits. Consists of the following (the values of each summed together gives the final memory address to dereference:
  - 64 bit offset
  - Optional register of any size *(draft: does it get truncated? before or after the summation?)*
  - Optional register of any size multiplied by a constant 1, 2, 4, or 8 *(draft: same as above)*

## Logic Machine
*(draft: logic vm, registers of the same number are views like ax and al, etc, should writes to low bit regs overwrite the high bits)*

`r8` is the instruction pointer. `r7` is the stack pointer. *(draft: alias `rip`/`rsp` to `r8`/`r7`?)*

## Logic Instructions
This is a list of all logic instructions. Replace each instance of:
- `T`, `U` and `V` in an instruction with a different argument type (`imm`, `reg`, `mem`)
- `B` and `C` with a supported bitwidth.
- `L` with lvalue argument types (`reg`, `mem`)
Example: `setzx_TB_UC x: TB, y: UC` would turn into `setzx_reg8_reg8 x: reg8, y: reg8`, `setzx_reg16_reg8 x: reg16, y: reg8` etc.

*(draft: arithmetic ops set flags?)*
Note:  All arguments are evaluated before the instruction execution (i.e. `push r7` = )

### Arithmetic Instructions

| Mnemonic | Description + Notes | Pseudocode |
| -- | -- | -- |
| `addB_L_U x: LB, y: UB` | Adds two integer values and stores the result in the first value. Wraps. | `x = x + y` |
| `subB_L_U x: LB, y: UB` | Subtracts the second integer value from the first integer value, storing the result in the first value. Wraps. Behaves equivalent to `addB_L_U x, z` where `z` is the negation of `y` | `x = x - y` |
| `umulB_L_U x: LB, y: UB` | Multiplies two **unsigned** integer values and stores the result in the first value *(draft: specify exact wrapping behavior)* | `x = x * y` |
| `imulB_L_U x: LB, y: UB` | Multiplies two **signed** integer values and stores the result in the first value *(draft: specify exact wrapping behavior)* | `x = x * y` |
| `udivB_L_U x: LB, y: UB` | Divides two **unsigned** integer values and stores the result in the first value. Truncates towards zero *(draft: specify exact wrapping behavior)* | `x = floor(x / y)` |
| `idivB_L_U x: LB, y: UB` | Divides two **signed** integer values and stores the result in the first value. *(draft: specify exact wrapping behavior, how does rounding work?)* | `x = floor(x / y)` |
| *(draft: add modulo, possibly signed and unsigned versions)* | | |

### Bitwise Operations

| Mnemonic | Description + Notes | Pseudocode |
| -- | -- | -- |
| `bitandB_L_U x: LB, y: UB` | Bitwise AND x and y, storing the result in x | `x = x & y` |
| `bitorB_L_U x: LB, y: UB` | Bitwise OR x and y, storing the result in x | `x = x | y` |
| `bitnotB_L x: LB` | Bitwise NOT x, storing the result in x | `x = ~x` |

### Memory Operations

| Mnemonic | Description + Notes | Pseudocode |
| -- | -- | -- |
| `movB_L_U dst: LB, src: UB` | Copies the B-bit value `src` into `dst`. *(draft: overlapping? must be aligned?)*| `dst = src` |
| `pushB_T x: TB` | Pushes the specified value onto the stack. | `r7 -= B / 8; *r7 = x` |
| `popB_L x: LB` | Pops a value from the stack and stores it in the specified value. | `x = *r7; r7 += B / 8` |
| `leaB_L x: LB, y` | `y` is a memory dereference, however the memory is not actually dereferenced. The resulting address is stored into `x` after being truncated to fit the bitwidth. *(draft: zero extension or sign extension?)* | - |
| `memcpy_T_U dst: T64, src: U64, n: V64` | Copies `n` bytes of memory from the address `src` to the address `dst`. If the areas overlap, it will act as if `n` bytes from `src` were copied into a temporary region, and then from there to the destination. | `memmove(dst, src, n)` |
| `memcpynv_T_U dst: T64, src: U64, n: V64` | Copies `n` bytes of memory from the address `src` to the address `dst`. The `nv` stands for non-overlapping. If the areas overlap, it will act like copying from the start to the end. *(draft: clarify with example)* | `memcpy(dst, src, n)` |

### Floating Point Operations

Replace F with 32 or 64. Floats are represented as and follow the rules of IEEE 754 floating point values.

*(draft: many missing operations here)*

| Mnemonic | Description + Notes | Pseudocode |
| -- | -- | -- |
| `addfF x: LF, y: UF` | Adds two float values and stores the result in the first. | `x = x + y` |
| `subfF x: LF, y: UF` | Subtracts the second float value from the first and stores the result in the first. | `x = x - y` |
| `mulfF x: LF, y: UF` | Multiplies two float values and stores the result in the first. | `x = x * y` |
| `divfF x: LF, y: UF` | Divides the first float value by the second and stores the result in the first. | `x = x / y` |
| `sqrtfF x: LF` | Gets the square root of the specified value. | `x = sqrt(x)`  |
| *(draft: add modulo)* | | |

### Misc Operations

| Mnemonic | Description + Notes | Pseudocode |
| -- | -- | -- |
| *(draft)* `cext ext: imm32, inst_num: imm32, args...` | Calls an extension-provided instruction. `ext` is the extension number in the program extension table, `inst_num` is the instruction number in the extension. `args` is a variadic amount of arguments of any type. Errors at runtime if the extension/instruction number is invalid, or if the arguments are the wrong type for the extension. | - |
