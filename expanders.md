# Expanders
On the start of compilation, there are two internal functions. The first is called the expander, the second is the evaluator. The default expander perfoms the following (pseudocode):
```
logicMacrosList = ["add8_reg_mem", ...]

function defaultExpander(ast) {
  return _defaultExpander(ast, false)
}

function _defaultExpander(ast, prelude) {
  if ast length > 0 and ast[0] is equivalent to `$no_prelude` {
    assertOrCompileError(ast[1] is a BlockLit)
    ex = emptyLogic()
    for each expression in ast[1] {
      assertOrCompileError(expression in logicMacrosList)
      ex.append(logic version of expression)
    }
    restOfAst = ast[2..]
    interpreter = newLogicInterpreter(ex)
    interpreter.setRegister("r0", (void*) restOfAst)
    interpreter.pushExitToStack() # An address that when jumped to, finishes interpretation.
    interpreter.interpret()
    expandedProgram = (Logic) interpreter.getRegister("r0")
    return expandedProgram
  } else {
    if prelude { compileError() }
    return _defaultExpander(preludeAst.concat(ast), true)
  }
}
```

The expander turns a Macron program into logic. After that, it should be compiled or interpreted, depending on the selected target.

## Special Logic
Expanders have access to special *intrinsics*. An intrinsic is similar to logic, but it is not found in expanded programs, rather in the expander itself. The most important of these intrinsics is called `mkinst`.

`mkinst` takes no arguments, but interprets the value in `r0`  as a pointer to an **instruction descriptor**. In `r0` will be a pointer to the resulting logic.

The structure of an instruction descriptor is as follows:

| Offset | Field | Description |
| -- | -- | -- |
| 0 | `discrim: u32` | The instruction discriminator. Get it using  |
| 4 | `args` | A list of arguments based on the value of `discrim`. |

### Instructions By Descriptor Discriminator + Intrinsics List

Replace N with the amount of logic instructions, as specified in the [*Logic*](./logic.md) section of this specification. Replace M with the maximum unsigned 32 bit integer limit.

| Discriminator | Instruction | Description + Notes | Pseudocode |
| -- | -- | -- | -- |
| 0 up to N | All logic instructions, in the order they are specified in the *Logic* section. *(draft: how to order replacements? `add8_reg_mem` or `add8_mem_reg` fist?)* | - | - |
| M | `mkinst` | Creates logic based on an instruction descriptor. | *(implementation specific)* |
| M - 1 | `lfree_T x: T64` | Frees logic. *(draft: use after free?)* | *(implementation specific)* |
| M - 2 | `lappend_T_U dst: T64, src: U64` | Appends the logic at `src` to `dst`. | *(implementation specific)* |
| M - 3 | `lclone_L_U dst: L64, src: U64` | Clones the logic `src`, placing the result in `dst` (not overriding the logic at `dst`, setting it to a new value entirely) | *(implementation specific)* |
| M - 4 | `lempty_L dst: L64` | Creates logic which does nothing and stores it in `dst`. | *(implementation specific)* |
| M - 5 | `lclear_T dst: T64` | Turns the logic at `dst` into logic which does nothing. | *(implementation specific)* |
| M - 6 | `lclear_T dst: T64` | Turns the logic at `dst` into logic which does nothing. | *(implementation specific)* |
| M - 7 | `ext_add path: immstring` | Loads the package specified by `path` as a post-gen extension. See the *Extensions* section of this specification for more info. | *(implementation specific)* |
