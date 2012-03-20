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
- modify the code generator to invoke the library functions as necessary