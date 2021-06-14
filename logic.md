# Logic
Logic is the base executable entity in Macron. Logic is split into two layers, layer 1 and 2, each of which have instructions. Each layer 1 logic instruction is essential and unexpandable. Each layer 2 logic instruction can be converted into a series of layer 1 logic instructions and keep the same behavior, but possibly with differing performance or efficiency. An implementation must support atleast the layer 1 and 2 instructions specified below. Implementation specific instructions are strongly discouraged but allowed if they are prefixed with an underscore. *(draft: should we completely disallow impl-specific instructions)*

An implementation must support bit widths equivalent to 8 times (2 to the power of N) for each N ranging from zero to an implementation specified limit. N must be greater than or equal to zero. *(draft: too wordy?)*

Logic instructions may take arguments. These are the possible argument types, where B is a supported bit width:
- `immB`: An immediate (constant) value consisting of B bits. Syntax: A decimal number, or the prefix `0x` followed by a hexadecimal number, with capital or lowercase letters. Underscores are supported to delimit the number.
- `regB`: A register with B bits. The amount of registers available is unspecified. The syntax for it: `rN_B`
- `memB`: A memory location dereference with B bits. Consists of the following (the values of each summed together gives the final memory address to dereference:
  - Offset
  - Optional register of any size *(draft: does it get truncated? before or after the summation?)*
  - Optional register of any size multiplied by a constant 1, 2, 4, or 8 *(draft: same as above)*

## Logic Machine
*(draft: logic vm, registers of the same number are views like ax and al, etc)*

## Layer 1 Logic
This is a list of all layer 1 logic instructions. Replace every instance of the capital letter M with the maximum supported bit width:
| Mnemonic | Description | Pseudocode |
| -- | -- | -- |
| `sub_M x: regM/memM, y: regM/memM/immM` | Subtracts the second integer value from the first value, placing the result in the first value. Behaves like M-bit unsigned subtraction, including overflow behavior. *(draft: sets flags?)* | `x = x - y` |
| *(draft: TODO)* |

## Layer 2 Logic
This is a list of all layer 2 logic instructions. Replace each instance of every different capital letter with each the supported bitwidths *(draft: ambiguous)*
| Mnemonic | Description | Pseudocode / L1 Expansion |
| -- | -- | -- |
| `add x: regM/memM, y: regM/memM/immM` | Equivalent to casting the second value to a value of X bits via zero extension or truncation and calling `add_X v` where v is the resulting value | `x = x + y` |
| *(draft: TODO)* |
