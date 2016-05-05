Freeze
======

[![GoDoc](https://godoc.org/github.com/lukechampine/freeze?status.svg)](https://godoc.org/github.com/lukechampine/freeze)
[![Go Report Card](https://goreportcard.com/badge/github.com/lukechampine/freeze)](https://goreportcard.com/report/github.com/lukechampine/freeze)

```
go get github.com/lukechampine/freeze
```

Package freeze enables the "freezing" of data, similar to JavaScript's
`Object.freeze()`. A frozen object cannot be modified; attempting to do so
will result in an unrecoverable panic.

To accomplish this, the `mprotect` syscall is used. Sadly, this necessitates
allocating new memory via `mmap` and copying the data into it. This
performance penalty should not be prohibitive, but it's something to be aware
of.

Freezing is useful to providing soft guarantees of immutability. That is: the
compiler can't prevent you from mutating an frozen object, but the runtime
can. One of the unfortunate aspects of Go is its limited support for
constants: structs, slices, and even arrays cannot be declared as consts. This
becomes a problem when you want to pass a slice around to many consumers
without worrying about them modifying it. With freeze, you can guard against
these unwanted or intended behaviors.

Two functions are provided: `Pointer` and `Slice`. (I suppose these could have
been combined, but then the resulting function would just be called `Freeze`,
which stutters.) To freeze a pointer, call `Pointer` like so:

```go
var x int = 3
xp := freeze.Pointer(&x).(*int)
println(*xp) // ok; prints 3
*xp++        // not ok; panics
```

It is recommended that, where convenient, you reassign the returned pointer to
its original variable, as with append. Note that in the above example, `x` can
still be freely modified.

Likewise, to freeze a slice:

```go
xs := []int{1, 2, 3}
xs = freeze.Slice(xs).([]int)
println(xs[0]) // ok; prints 1
xs[0]++        // not ok; panics
```

It may not be immediately obvious why these functions return a value that must
be reassigned. The reason is that we are allocating new memory, and therefore
the pointer must be updated. The same is true of the built-in `append`
function. Well, not quite; if a slice has greater capacity than length, then
`append` will use that memory. For the same reason, appending to a frozen
slice with spare capacity will trigger a panic.

Currently, only Unix is supported. Windows support is not planned, because it
doesn't support a syscall analogous to `mprotect`.
