# Logic
Logic is the base executable entity in Macron. Logic is made up of logic instructions. Instructions are executed in order except for branching. Logic macros may be used to generate raw logic.

An implementation must support bit widths 8, 16, 32, 64, and 128.

Logic instructions may take arguments. These are the possible argument types, where B is a supported bit width:
- `immB`: An immediate (constant) value consisting of B bits. Written signed or unsigned depending on the instruction.
- `regB`: A register with B bits. 8 registers are available.
- `memB`: A memory location dereference with B bits. Consists of the following (the values of each summed together gives the final memory address to dereference:
  - 64 bit offset
  - Optional register of any size *(draft: does it get truncated? before or after the summation?)*
  - Optional register of any size multiplied by a constant 1, 2, 4, or 8 *(draft: same as above)*

## Logic Machine
The Logic Machine is an abstract machine/VM which runs logic. A logic machine takes logic and runs it. Execution starts at the beginning of the logic. Memory is not contiguous, there may be regions of memory which cause segfaults or other unusual behavior if accessed, depending on the running environment. Specifically, the null pointer usually causes a segfault when dereferenced on most major operating systems, though this is not a property of the logic machine (i.e. if running in an appropriate environment such as a bootloader, the null pointer may be perfectly valid).

All register values are 0 by default, except for `r8`. `r8` by default is set to point to the top a stack with a capacity of pushing N bytes, where N defaults to at least 4096 but can be specified with the `-S` or `--stacksize` flag. If more than N bytes are pushed to the stack at once, a stack overflow occurs. If interpreting, this causes program termination along with an error. If compiled, the OS will handle it and you shouldn't do anything.

The stack goes downwards. `r8` points to the currently pushed value. Pushing does `r8 -= sizeof(x); *r8 = x`.

`r8` is the stack pointer.

## Logic Instructions
This is a list of all logic instructions. Replace each instance of:
- `T`, `U` and `V` in an instruction with a different argument type (`imm`, `reg`, `mem`)
- `B` and `C` with a supported bitwidth.
- `L` and `M` with lvalue argument types (`reg`, `mem`)
Example: `movzx_LB_UC dst: LB, src: MC` would turn into `movzx_reg8_reg8 dst: reg8, src: reg8`, `movzx_reg16_reg8 dst: reg16, src: reg8` etc.

*(draft: arithmetic ops set flags?)*
Note:  All arguments are evaluated before the instruction execution (i.e. `push r8` = `x = r8; r8 -= 8; *r8 = x`)

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

### Bitwise Instructions

| Mnemonic | Description + Notes | Pseudocode |
| -- | -- | -- |
| `bitandB_L_U x: LB, y: UB` | Bitwise AND x and y, storing the result in x | `x = x & y` |
| `bitorB_L_U x: LB, y: UB` | Bitwise OR x and y, storing the result in x | `x = x | y` |
| `bitnotB_L x: LB` | Bitwise NOT x, storing the result in x | `x = ~x` |

### Memory Instructions

| Mnemonic | Description + Notes | Pseudocode |
| -- | -- | -- |
| `movB_L_U dst: LB, src: UB` | Copies the B-bit value `src` into `dst`. *(draft: overlapping? must be aligned?)*| `dst = src` |
| `movzx_LB_UC x: LB, y: MC` | Copies `src` to `dst`, zero extending. | `dst = zx(src)` |
| `movsx_LB_UC x: LB, y: MC` | Copies `src` to `dst`, sign extending. | `dst = sx(src)` |
| `pushB_T x: TB` | Pushes the specified value onto the stack. | `r8 -= B / 8; *r8 = x` |
| `popB_L x: LB` | Pops a value from the stack and stores it in the specified value. | `x = *r8; r8 += B / 8` |
| `leaB_L x: LB, y: mem*` | `y` is a memory dereference, however the memory is not actually dereferenced. The resulting address is stored into `x` after being truncated to fit the bitwidth. *(draft: zero extension or sign extension?)* | - |
| `memcpy_T_U_V dst: T64, src: U64, n: V64` | Copies `n` bytes of memory from the address `src` to the address `dst`. If the areas overlap, it will act as if `n` bytes from `src` were copied into a temporary region, and then from there to the destination. | `memmove(dst, src, n)` |
| `memcpynv_T_U_V dst: T64, src: U64, n: V64` | Copies `n` bytes of memory from the address `src` to the address `dst`. The `nv` stands for non-overlapping. If the areas overlap, it will act like copying from the start to the end. *(draft: clarify with example)* | `memcpy(dst, src, n)` |

### Floating Point Instructions

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

### Control Flow Instructions

| Mnemonic | Description + Notes | Pseudocode |
| -- | -- | -- |
| `label` | Creates a label, see below. | - |
| `jneB_T_U x: imm32, a: TB, b: UB` | Constant jump if `a != b`, see below. | - |
| `jeB_T_U x: imm32, a: TB, b: UB` | Constant jump if `a == b`, see below. | - |
| `jilB_T_U x: imm32, a: TB, b: UB` | Constant jump if `a < b` (signed 2s complement), see below. | - |
| `jigB_T_U x: imm32, a: TB, b: UB` | Constant jump if `a > b` (signed 2s complement), see below. | - |
| `julB_T_U x: imm32, a: TB, b: UB` | Constant jump if `a < b` (unsigned), see below. | - |
| `jugB_T_U x: imm32, a: TB, b: UB` | Constant jump if `a > b` (unsigned), see below. | - |
| `switchB_L_T_U x: L32, a: TB, b: UB, cases: imm32...` | Computed conditional jump, see below. | - |
| `callB x: imm32` | Calls the specified label. Performs an implementation specific action so that the next time `return` is called, it will return execution to the instruction after this one.  | *(implementation specific)* |
| `return` | Returns to the previous call. | *(implementation specific)* |

Labels are stored in a list at interpretation/compile time. When interpreting:
- The interpreted block is scanned for the `label` instruction, and each time it is encountered, the current instruction pointer is pushed to the label list.
- For jumps:
  - Let `label(n)` be the result of indexing from the table. `n = 0` = last element of the list as of that instruction. Negative = previous labels, positive = next labels.
  - For constant jumps, jump to `label(x)`.
  - For computed jumps, jump to `label(cases[x])`. If `x` is out of bounds, an error occurs.
- The list is preserved across [executor](./expanders.md) calls.
When compiling: (N.B. this list is an example, you may implement it differently as long as it follows the same behavior)
- The logic is scanned for the `label` instruction, and each time it is encountered, the current target instruction pointer is pushed to the label stack.
- The logic is scanned again for jumps:
  - For constant jumps, the list is indexed as described above. The compiled result contains a constant jump to the target instruction pointer offset fetched.
  - For computed jumps, a jump table is created in the target containing an offset for each case in `cases`. This jump table is indexed by the argument `x`. The compiler can assume `x` will never be out of bounds, as this is UB.

### Misc Instructions

| Mnemonic | Description + Notes | Pseudocode |
| -- | -- | -- |
| *(draft)* `cext ext: imm32, inst_num: imm32, args...` | Calls an extension-provided instruction. `ext` is the extension number in the program extension table, `inst_num` is the instruction number in the extension. `args` is a variadic amount of arguments of any type. Errors at runtime if the extension/instruction number is invalid, or if the arguments are the wrong type for the extension. | - |
