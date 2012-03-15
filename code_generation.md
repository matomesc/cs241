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

Then all we need to do to represent `NULL` is zero the result register:

```
sub $3, $3, $3
```

### factor -> ( expr )

`code(factor) = code (expr)`

### statement -> lvalue = expr;

We know how to generate `expr`. But how do we generate `lvalue`? (lvalue is the left hand side of an expression).  
We want the lvalue to return an address in RAM ie. store the address in $3.

code(lvalue):

```
sw $3, -4($30)
sub $30, $30, $4

code(expr) // has result in $3

add $30, $30, $4
lw $5, -4($30)
sw $3, 0($5)
```

This could have easily been done by returning `lvalue` in register `$5` to save us from messing with the stack.

### factor -> & lvalue

```
assert(typ(lvalue) == int)

```

Since the lvalue has already been generated, there is nothing else to do so `code(factor) = code(lvalue)`

### factor1 -> * factor2

```
assert(typ(factor2)) == int *)
lw $3, 0($3)
```

### lvalue -> ID

- find the offset of `ID` in the symbol table. This offset should be relative to `$29` (stack frame pointer)

```
lis $5
.word offset
add $3, $29, $5
```

### lvalue -> * factor

```
assert(typ(factor) == int *)
// since * factor is a pointer and lvalue is also a pointer, there is nothing to be done
code(lvalue) = code(factor)
```

### dcls -> dcls dcl = NUM;

```
assert dcl declares an int variable
```
code(dcl) returns address of returned variable

- do same thing as assigment:

code(dcl):

```
sw $30, -4($30)
sub $3, $30, $4

code(NUM)

add $30, $30, $4
lw $5, -4($30)
sw $3, 0($5)
```

### dcl -> type ID

- same as lvalue -> ID

```
lis $5
.word offset(ID)
add $3, $29, $5
```

### procedure -> INT WAIN ... dcl1 ... dcl1 ... dcls ... statements ... expr

This is the __meat__

```
// this returns an lvalue
// it is also the first parameter to the procedure
code(dcl1)
// store it
sw $1, 0($3)

// 2nd parameter
code(dcl2)
// store it
sw $2, 0($3)

code(dcls)

code(statements)

// final result that goes into $3
code(expr)
```

### expr -> expr - or + token

case 1: `typ(expr) = typ(term) = int`

```
code(expr) // result in $3
sw $3, -4($30)
sub $30, $30, $4 // push $3 on stack
code(term) // result in $3
add $30, $30, $4 // pop stack
lw $5, -4($30) // into $5
sub $3, $5, $3 // for -
add $3, $5, $3 // for +
```

case 2: `typ(expr) = int*, typ(term) = int`

```
code(expr)
sw $3, -4($30)
sub $30, $30, $4
code(term)
add $3, $3, $3
add $3, $3, $3 // multiply by 4
add $30, $30, $4
lw $5, -4($30)
add $3, $5, $3
```

case 3: `typ(expr) = int, typ(term) = int*`

```
code(term)
sw $3, -4($30)
sub $30, $30, $4
code(expr)
add $3, $3, $3
add $3, $3, $3 // multiply by 4
add $30, $30, $4
lw $5, -4($30)
add $3, $5, $3
```

```
int *a;
int *b;

a - b = (addr(a) - addr(b)) / 4
```

### term1 -> term2 * or / or % factor

```
code(term2)
sw $3, -4($30)
sub $30, $30, $4
code(factor)
add $30, $30, $4
lw $5, -4($30)

// for multiplication
mult $5, $3 // result goes into HI, LO so we have to move em from there

// for division
// div $5, $3

// for remainder
// mfhi $3

mflo $3
```

### factor -> NEW INT [ expr ]

- create heap in prologue
- allocate `n` bytes where `n` is result of expr
- how are we gonna accomplish this? two library functions we'll write: `malloc`
- the call to `malloc` needs to be in epilogue

```
code(expr)
add $1, $3, $3
add $1, $1, $1 // put 4 * $3 in $1
// call malloc, and result is in $3
lis $3
.word malloc
jalr $3
```

### statement -> DELETE [ ] expr ;

```
code(expr)
add $1, $3, $0
lis $3
.word free // assume we have a function called free in the epilogue
jalr $3
```

### statement -> PRINTLN ( expr ) ;

```
code(expr)
add $1, $3, $0
lis $3
.word println // again assume println function in epilogue
jalr $3
```

### test -> expr1 == expr2

- test is a boolean expression
- since we don't have booleans in WLPP, we need to define some conventions for true and false
  - a word containing `0` for false
  - a word containing `1` for true
- one implementation:

```
code(expr1)
sw $3, -4($30)
sub $30, $30, $4
code(expr2)
add $30, $30, $4
lw $5, -4($30)
add $1, $11, $0 // put 1 in $1
beq $3, $5, 1
add $1, $0, $0 // put 0 in $1
add $3, $1, $0 // copy $1 to $3
```

### statement -> if (test) { statements1 } else { statements2 }

```
code(test)
code(statements)
code(statements)
```