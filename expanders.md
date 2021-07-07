# Expanders
On the start of compilation, there are two internal functions. The first is called the expander, the second is the evaluator. The default expander perfoms the following (pseudocode):
```
logicMacrosList = ["add8_reg_mem", ...]

function defaultExpander(code) {
  _defaultExpander(code, false)
}

function _defaultExpander(code, prelude) {
  if code.trim() starts with "$$no_prelude" {
  	if line containing "$$no_prelude" contains anything else other than whitespace {
  		compileError()
  	}
  	c2 = all code after $$no_prelude line
    ex = emptyLogic()
    for each line in c2 {
      if line.trim() is "__expander_end" { break }
      assertOrCompileError(line is valid syntax for logic based on logicMacrosList)
      ex.append(logic version of line) # (draft: specify exactly the right syntax)
    }
    restOfCode = everything after the break
    interpreter = newLogicInterpreter(ex)
    interpreter.push(restOfCode length)
    interpreter.push(restOfCode pointer)
    interpreter.pushExitToStack() # An address that when jumped to, finishes interpretation.
    interpreter.interpret()
  } else {
    if prelude { compileError() }
    _defaultExpander(preludeAst.concat(ast), true)
  }
}
```

Note that this pseudocode is simplified as it takes the `code` argument whole and calls the expander only once. The actual process is as follows:
1. Code is inputted by the user. Either from a file, in a REPL, etc. The rest of this list will assume a REPL. Set the internal *DidPrelude* variable to false.
2. We wait for a whole line of input to arrive (ended by a newline). If it, trimmed of whitespace, is not equal to `$$no_prelude`, then go to step 6.
3. If it is equal, then keep taking in input until a line, when trimmed, is equal to `__expander_end`. When this happens, continue with the following steps where E is the user input, starting from the line after `$$no_prelude` and excludes the `__expander_end` line.
4. Iterate through E's lines and perform the following actions on them, where X is the current line and L is a newly made empty logic:
  - Trim X.
  - If X begins with the character `#`, ignore the line and continue with the rest of the lines in E.
  - Parse X as a logic statement. *(draft: specify parsing process and grammar)*
  - If that fails, compile error.
  - Append the resulting logic to L.
5. Go to step 7.
6. Act as if the entire Macron prelude was inputted instead of the line we just received. Then, the line received is processed, then all of the following input as normal. If the internal *DidPrelude* variable is true, compile error. Set the *DidPrelude* variable to true. Go back to step 2.
7. Create an interpreting context and store it internal variable *GenContext*.
8. Each time a new block of input is received:
  - Push the length of the UTF-8 block in bytes as a u64 to *GenContext*.
  - Push a pointer to the block's UTF-8 bytes as a u64 to *GenContext*.
  - Call the expander (L) in *GenContext*.
  - When the input ends, push two zeroes to *GenContext* and call the expander in *GenContext*.

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
| M - 6 | `lvfroml_L_U dst: L64, src: U64` | Creates an evaluatable logic from a logic. | *(implementation specific)* |
| M - 7 | `lveval_T x: T64` | Evaluates `x` in *GenContext*. | *(implementation specific)* |
| M - 8 | `lvfree_T x: T64` | Frees evaluatable logic. | *(implementation specific)* |
| M - 9 | `ext_add path: immstring` | Loads the package specified by `path` as a post-gen extension. See the *Extensions* section of this specification for more info. | *(implementation specific)* |
