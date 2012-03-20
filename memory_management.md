# Memory Management

- how to represent __variables__, values and data structure
- put variables in __frames__
- what about __precedures__, dynamic data structures?
  - WLPP has only the main procedure
  - C/C++ has user-defined procedures
  - Scheme's got nested procedures

## Procedures

```c
// global
int q;

// some procedure
int foo(int x, int y) {
  int a = 3;
  int* b = NULL;
  a = a + q;
  return *(a+b);
}

// anohter procedure
int bar(int w) {
  int z;
  z = z + q;
}
```

- how to represent `x, y, a, b`?
- what about the __global__ `q`?
- how to represent the procedure?

### Parameters

- our convention: `x` is passed as `$1` and `y` is passed as `$2`
- easiest is to put `x, y` in RAM

### Local variables

- create a __frame__ for foo allocated on __stack__ whenever foo is _called_!
- create new frame, set the __frame pointer__

### Local frame

The local frame pointer will point to local variables in the __local frame__, in this case:

```
// foo's local stack frame
---- <- frame pointer (fp)
b
----
a
----
y
----
x
----

// bar's local stack frame
---- <- fp
z
----
w
----
```

### Global frame

```
----
q
----
```

## Nested Procedures

- the most inner stack frame must also have access to previously allocated stack frames

## Non-stack Memory Management

- allocated storage that __persists__ beyond procedure invocation

```c
x = new int[10];  // allocate 10 words
// ....
delete[] x;       // free it up
```

## Garbage Collection

What about Scheme (or almost any other modern language)?

- we only allocate memory
- the language has built in __automatic garbage collection__ which frees unused memory
- we will discuss this concept in details next lecture
- but the idea is we keep track of everything we allocated and find all the reachable data, 
  then get rid of uncreachable data

Another way to get a dangling reference in C:

```c
int* dangle() {
  int x;
  int* y = &x;
  return y;
}

int* a = dangle();

// global fp
----
a
----

// dangle's fp
----
x
----
y
----

// but once dangle executes, the frame will get deleted leaving `a` dangling
```

Anyways the idea is stack based memory management sucks.

## Implementing Non-stack Memory Management

- build a set of library functions that implement the following on a global arena of storage (aka __heap__):
- note this is different that priority queues which is also implemented by a heap (which is a tree data struct)
  - initialization
  - finalization
  - allocation (how we get new memory)
  - reclamation (reuse memory that is no longer in use)
    - identification
    - reuse
- modify the code generator to invoke the library functions as 

__simple approach__

- use heap for __fixed-sized__ allocation unsits
  - eg. Scheme: `(cons a b)`

__complex approach__

- __varaible-sized__ allocation units

### Fixed-size Allocation

- done in the initialization (prologue)
- allocate a "big" area of storage from stack (just like the frame):

```c
---- <- ap

ARENA

---- <- end

.word 0
```

- allocation for fixed size `n`:
  - find available `n` bytes, or __fail__ (this is important)
  - use the __slice of bread__ algorithm:

```c
start                       end
  ^                          ^
  | (USED)  x    (UNUSED)    |
            ^
      first_available

if (first_available + n > end) {
  fail()
}

avail = avail + n
return avail - n
```

- finalization? get rid of `ARENA`
- reuse is trivial (vacuous): `free() {}`
- wastes space, needlessly fails
  - this is due to fragmentation in the heap ie. `XOXOX` where `X` is used and `O` was used but was freed
- fail only if there is not enough unused memory to satisfy the request
- we need to keep track of unused (free) storage using an available space list:

```c
// X = used

// stores pointers to the first byte of each free chunk
// each node points to the next chunk of free space
List = a1 -> a2 -> a3 -> a4;

 a1     a2      a3     a4
|   XXXX     XXX    XXX           |
                              ^
                            avail

if asl != null then
  tmp = asl
  asl = *(asl)
else if avail + n > end then fail
else avail = avail + n
  return avail - n
```