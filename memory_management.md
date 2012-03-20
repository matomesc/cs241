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
