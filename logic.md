# Macron Logic
Intrinsics for Macron: these were chosen because most needed operations can be synthesized by stringing a few of these together. Since the stdlib will be pretty much completely abstracting away the intrinsics, the compiler devs (us) who are also writing the stdlib can take elements from the stdlib which do a certain task and hardcode that into the compiler to do the correct instruction.

Operand types:
- `imm`: Immediate/constant.
- `mem`: Equivalent to any other operand type than `mem`, but dereferenced. A constant size (1/2/4/8 bytes) is also provided.
- `var`: A constant opaque reference to a variable.
- `reg`: Refers to either of the two registers: `sp` and `bp`, the stack and base pointers respectively.

L = `var`/`mem`/`reg`, T/U = `var`/`mem`/`reg`/`imm`

Note: where it would be known at comptime, such as `jmpgt .., 5, 6`, that combination isn't allowed (jmpgt on imm and imm specifically here)

All IDs increment once used. For example
```
allocvars ffffffffffffffff
allocvars 0
freevars 1
freevars 0
```
is the correct way to do a frame contained in another.

```
# This intrinsic is equivalent to the following pseudocode:
#   (id, amt, var_ids) = internal_allocvars(var_sizes...)
#   sp -= amt
# i.e., it subtracts enough space to store the variables from sp, and generates a unique id. You should use this id and the generated var_ids (you'll have access to them at gentime) to access the variables and to call freevars. If you're currently inside an allocvars frame, you should pass its id as `prev_alloc` (otherwise pass -1u64) so that registers aren't reused or the like.
allocvars prev_alloc, var_sizes: imm64...

# Performs:
#   amt = internal_allocvars_get_amt(allocvars_id)
#   sp += amt
# Use this to end a frame.
freevars allocvars_id

add     result: L, x: L, y: T
nand    result: L, x: L, y: T
shiftl  result: L, x: T, amt: U # shift left
shiftrz result: L, x: T, amt: U # shift right zero extend
shiftrs result: L, x: T, amt: U # shift right sign extend

label addr: L, label_id # store label address from label id

jmp   addr: T
jmpl  label_id
jmpeq label_id, x: L, y: U

# (draft: should do less than and below instead?)

jmpgt label_id, x: T, y: U # unsigned greater than
jmpab label_id, x: T, y: U # signed greater than (draft: can be done relatively easily with jmpgt?)
```
