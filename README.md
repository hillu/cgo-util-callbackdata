# Callback data handling for CGO

Calling C code from Go is mostly trivial for simple cases and it is
also simple to pass exported Go functions as callback functions to the
C code. However, many C libraries expect a pointer that can be passed
back to the callback function and it is not possible to directly pass
non-trivial Go data structes to the callback function implemented in
Go -- an attempt results in a "cgo argument has Go pointer to Go
pointer" runtime panic.

The `Pool` type contained in this package implements a container for
such callback data. Its `Put` method stores any Go type and returns an
`unsafe.Pointer` which maps directly to `void*` in function calls
crossing the GO/C boundary. This pointer references a simple numeric
index which is stored in `calloc`-allocated memory that will not be
moved around by the GC, therefore the pointer will not suddenly point
to unrelated data. This numeric index is used by the `Get` method to
access the object from a `[]interface{}` slice which may be moved by
the GC.

## Notes

A previous implementation contained in
[go-yara](https://github.com/hillu/go-yara) used a mutex-protected
`map[uintptr]interface{}` to store callback data. If the `uintptr`
values were simply cast to `unsafe.Pointer`, `go vet` would complain
about "possible misuse of unsafe.Pointer". Passing a reference to this
`uintptr` value instead would not work in all use cases because I
ended up storing pointers to Go data in memory not accessible by the
GC, breaking the CGO pointer rules and leading to crashes.

## License

BSD 2-clause, see LICENSE file in the source distribution.

## Author

Hilko Bengen <bengen@hilluzination.de>

