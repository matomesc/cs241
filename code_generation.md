# Code Generation (completing the compiler)

Summary so far:

```
WLPP Source -> Scanner -> Token List -> Parser -> WLPPI file -> Context-Sensitive analysis (using the parse tree) + Code generation
    -> asm file -> CS241.binasm -> MIPS binary -> mips.twoints
```

Pretty much every single one can error.

```
Procedure -> asm file (stdout):

Prologue
Code fragment code (procedure) from syntax directed translation
Epilogue
```

Prologue:
- prologue establishes conventions
- save registers
- capture parameters
- anything else than needs to get setup

Epilogue:
- restore registers
- produce the result
- utility procedures (for instance an external print procedure for stdout)