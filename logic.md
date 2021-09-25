# Macron Logic
Intrinsics for Macron: these were chosen because most needed operations can be synthesized by stringing a few of these together. Since the stdlib will be pretty much completely abstracting away the intrinsics, the compiler devs (us) who are also writing the stdlib can take elements from the stdlib which do a certain task and hardcode that into the compiler to do the correct instruction.

Operand types:
- `imm`: Immediate/constant.
- `mem`: Equivalent to any other operand type than `mem`, but dereferenced. A constant size (1/2/4/8 bytes) is also provided.
- `var`: A constant opaque reference to a variable.
- `reg`: Refers to either of the two registers: `sp` and `bp`, the stack and base pointers respectively.

L = `var`/`mem`/`reg`, T/U = `var`/`mem`/`reg`/`imm`

Note: where it would be known at comptime, such as `bgt .., 5, 6`, that combination isn't allowed (bgt on imm and imm specifically here)

All IDs increment once used. For example
```
allocvars ffffffffffffffff
allocvars 0
...
```
is the correct way to write a frame contained in another.

```
# This intrinsic is equivalent to the following pseudocode:
#   (amt, var_ids) = internal_allocvars(var_sizes...)
#   sp -= amt
# i.e., it subtracts enough space to store the variables from sp, and generates a unique id. You should use the generated var_ids to access the variables. If you're currently inside an allocvars frame, you should pass its id as `prev_alloc` (otherwise pass -1u64) so that registers aren't reused or the like.
# Accessing variables will only work properly if you set `b` to `s` before calling allocvars as the variables will be based on the base pointer, not the stack pointer. This means that to exit a frame you must simply restore the old base pointer.
allocvars prev_alloc, var_sizes: imm64...

add     result: L, x: L, y: T
nand    result: L, x: L, y: T
shiftl  result: L, x: T, amt: U # shift left
shiftrz result: L, x: T, amt: U # shift right zero extend
shiftrs result: L, x: T, amt: U # shift right sign extend

# labels, 0 = last defined label, -1 = one before it, 1 = one after, etc.

blockaddr result: L, block_id: imm64
```

# Blocks and Functions
Logic in Macron is structured into blocks which are then added into functions. A block is similar to an LLVM basic block, in that it contains logic and at the end is a branch. In the expander, the following intriniscs can be used to end the current block with a branch and start a new one:

```
# Unconditional branch
b block_id

# Branch to address
baddr addr: T

# Branch if equal
beq true_id, false_id x: L, y: T

# Branch if greater than (unsigned)
bgt true_id, false_id, x: T, y: T

# Branch if greater than (signed)
bab true_id, false_id, x: T, y: T
```
