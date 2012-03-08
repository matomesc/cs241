# Context-sensitive Analysis

- semantic analysis
- how to do A9

- all checking of valid input that can't be done _easily_ by lexical & context-free analysis

Uses:

- syntax-directed translation

What we need to do:

- check variables are declared once and once only
- check that variables are used correctly

an example grammar:

```
// part of WLPP language specification
1. procedure → INT WAIN LPAREN dcl COMMA dcl RPAREN LBRACE dcls statements RETURN expr SEMI RBRACE 
2. type → INT // typeof(type) = INT
3. type → INT STAR // typeof(type) = INT STAR
4. dcls → 
5. dcls → dcls dcl BECOMES NUM SEMI
6. dcls → dcls dcl BECOMES NULL SEMI
7. dcl → type ID
```

## Building a symbol table
- it will hold a collection of symbols, with information about each symbol
  - for now just: `type ∈ { int, int* }`
- check for duplicates
- check all __uses__ of symbols
- type of every expression & subexpression in input

__multiset__ symbol table: `syms(N) = { <id, type>[i] }`

example symbol table for the above grammar:

```
// symbol table for procedure
syms(procedure) = syms(dcls) + syms(dcl1) + syms(dcl2)

// an easy one symbols for dcls - the empty set
syms(dcls) = {}

// for rule 7
syms(dcl) = { <lexeme(ID), typeof(type)> }
```

- check to make sure that `id1 != id2` for all distinct `<id1, type1>, <id2, type2> ∈ syms(procedure)`
- now we have the following functions:
  
  ```
  typeof(type) - declared type
  typ(E) - type of any particular expression E ∈ { int, int*, error }
  ```

more grammar:

```
statements →
statements → statements statement  
statement → lvalue BECOMES expr SEMI

lvalue → ID
typ(ID) = t if <lexeme(ID), t> in symbol_table else error

lvalue → STAR factor
typ(lvalue) = if typ(factor) == int* then int else error

expr → term
typ(expr) = typ(term)

expr → expr PLUS term

term → factor
typ(term) = typ(factor)

factor → lvalue
typ(factor) = typ(lvalue)

factor → NUM
typ(NUM) = int

factor → ID
typ(ID) = t if <lexeme(ID), t> in symbol_table else error
```