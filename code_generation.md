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

## Prologue

- prologue establishes conventions
- save registers
- capture parameters
- anything else than needs to get setup

## Epilogue

- restore registers
- produce the result
- utility procedures (for instance an external print procedure for stdout)

## Conventions

- self imposed rules to make everybody get along
- how is data going to be represented?
  - how are variables going to be represented? WLPP has `int` and `int *`
  - how are expressions going to be represented?
  - how are registers to be used for __parameters__ and __results__?

## Suggested Conventions
Can also be found [here](http://www.student.cs.uwaterloo.ca/~cs241/))

## Prologue Code Generation

- save registers
- set $11, $4
- allocate memory on stack for __all variables__
  - the __symbol table__ will tell us this information
  - use __stack__
  - subtract `$4 * n` from the stack pointer `$30`

  ```
  // allocating a stack frame
  
  ====== <-$30 (after)  
  (stack frame)  
  ====== <-$30 (initial)  


  ======
  ```
  
## Epilogue Code Generation

It basically has to undo the prologue.

- add `4n` to the stack pointer `$30`
- restore registers (except $3 which will store the procedure's result)
- `jr $31`
- also include library procedures here such as `print`

## Code Generation for Declarations, Statements & Expressions

### factor -> ID

- lookup factor in symbol table
- `offset(ID)` is assigned offset in frame for ID
- get that result into $3

code for factor:

```
lw $3, offset($29)
```

### factor -> NUM

```
lis $3
.word value(NUM)
```

### factor -> NULL

Oh how are we gonna represent pointer? Well an `int *` is an address and `NULL` is represented by address 0.
This is just a convention.

Then all we need to do to represent `NULL` is:

Zero the register:

```
sub $3, $3, $3
```