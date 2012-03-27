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