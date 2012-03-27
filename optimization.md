# Optimization

Source -> Compiler -> Object

`assert m[source] = m[object]`

In general, there is `object1 != object2 != object3` such that `m[object1] = m[object2] = m[object3]`

## goodness[object]

Ideally, find `objecti` such that `m[objecti] = m[source]` and `goodness[objecti]` is maximized.

But what is `goodness`?

- typically __speed__ (minimize running time)

## Static Optimization

Done at compile time based on things that we know already.

## Dynamic Optimization

Done at runtime.

## Space

- often smaller = faster
- also smaller and optimized loops results in faster programs due to instruction and data caching in the CPU
- space is really easy to measured

For example consider optimizing a `beq`:

```
beq $0, $0, foo // only works if branch offset <= 32767 words

// a workaround is to do:
lis $5
.word foo
jr $5
```

But consider a lot of flow control (ie. a lot of branches). Optimizing these branches is an NP complete problem because
the optimization of one branch affects the optimization of the rest. So one solution is approximating optimizations that
are always safe (ie. use long branches everywhere) but some branches could definitely be shortened.

## Approaches to Optimization

### Constant expression evaluation (folding)

  - `x = x + 2 * 3; => x = x + 6;`
  - from syntax directed translation, the code we generate can either:
    - puts result in $3
    - computes result as constant
  - `code(exec) = < code, representation >`

### Dead code elimination

Example:

```C
if (6 != 2 * 3) {
  // ...
}
```

There should be no code generated

### Invariant detection

Treat variables that don't change as constants.

Example:

```C
int x = 2;
// ...
// x never changes
// treate x as constant
```

How do we prove that x is a constant?

- check for `x = .... `, definitely not a constant since x was updated
- check for `y = &x` is tricky because `*y` now points to `x` and if you do `*y = 25`, then this will change `x`
  - this is a big thing in compiler optimization (alias detection)
  - you wanna be as safe and complete as possible
  - very tricky stuff going on with these __invariants__

### Common expression elimination

Copy elimination:

```C
// this can be optimized
x = 2 * y + 10;
z = 2 * y;

// to the following, provided z doesn't cahnge
z = 2 * y;
x = z + 10;
```

Another example:

```C
x = a[10];
a[10] = 2;
```

Since we do a lot of work trying to find the address of `a[10]`, the compiler can then cache the address.

### Loop optimization

Example:

```C
q = 2 * 3 * y;
while (t) {
  x = x + 2 * 3 * y;  // if y is unchaged in loop, move it outside (to q)!
}
```

This _might_ make things faster if we're executing the loop a large number of times.

### Strength reduction

```c
char *a, *b;
strlen(strcat(a,b));
```

- doing concatenation for nothing!
- represent strings as their length and value
- if you only manipulate the length, then their value becomes dead code
- consider the following:

```c
for (i = 0; i < 10; i++) {
  a[i] = 2;
}
```

- when you do a[i], you are storing 2 at `addr(a) + 4 * i`
- the issue here is `4 * i`
- transform it to this:

```c
t = addr(a) + 40
for (i = 0, I = addr(a); I < t; i += 1, I += 4) {
  // store 2 at address I
}
```

- keeping a copy of the original `i` is useful if the value of `i` was used inside the loop (ie. `a[i] = i`)

### Register usage

We have 32 registers available. Suppose we use 16 to store variables.

This means that these 16 variables won't be in the frame, saving us from messing with the stack frame.

So, for instance, consider we need to return a variable that we have in `$22`.

We could copy it to `$3`: `add $3, $22, $0`, but this is a redundant copy. We should just leave it in `$22` and next
when we use it we should expect it in `$22`. This requires a change of our __conventions__:

- result of expr can be any register: `<code, resultreg>`

`expr -> expr + term`:

```
// code(expr1):
sw $r, -4($30) // where r is resultreg(expr1)
```

- then we tell code which registers it is allowed to use: `code(expr, allow)` where `allow` is a set of registers that
can be safely used

```
code(expr1, allow)
code(expr2, allow) // result in is resultreg(expr2)
code(term, allow \ resultreg(expr2)) // set difference
// pick a register $t from allow, unless allow is empty
add $t, $r, $s // where $r is resultreg(expr), $s = resultreg(term)
```

- sometimes if we're out of registers, this is called __register pressure__
- managing registers is also a pretty difficult problem

#### Managing registers

Consider the following production:

`expr -> expr + term;`

Let needregs(expr) = how many registers we need to allocate good code

So, `needregs(expr1) = max(needregs(expr2), needreg(term)) + 1`. Remeber to save a register for the return result
(hence the `+ 1`)