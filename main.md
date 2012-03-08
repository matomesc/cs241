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
- now we have the following:
  
  ```
  typeof(type) - declared type
  typ(E) - type of any particular expression E
  ```

more WLPP grammar:

```
statements →
statements → statements statement  
statement → lvalue BECOMES expr SEMI
statement → IF LPAREN test RPAREN LBRACE statements RBRACE ELSE LBRACE statements RBRACE 
statement → WHILE LPAREN test RPAREN LBRACE statements RBRACE 
statement → PRINTLN LPAREN expr RPAREN SEMI
statement → DELETE LBRACK RBRACK expr SEMI
```