---
layout: post
title:  "Unit Testing in Go"
date:   2019-12-27 08:25:31 +0200
categories: programming go testing
---

# Unit Testing in Go

Testing with Go is made using the internal package "testing". Contrary to other languages Go does not require any extra tools,
notations or conventions for testing, this makes writing tests in Golang exactly the same as writing ordinary Go code.

Go provides the `go test` tool, which scans a package directory for files who names end with `_test.go`, generates 
a temporary main packages that calls all the special functions in the proper way, builds the temporary package, runs it, 
reports the results and then cleans up.

There are 3 types of functions that are executed by the `go test` tool: tests, benchmarks and examples.

## Test functions

Tests functions have the following signature:
```go
func TestName(t *testing.T) {
    // ...
}
```
Test function names must have the _Test_ suffix and they are your regular tests, they are independent from each other so
it means that one failure does not cause the whole testing to halt, a good approach since it allows multiple errors to be
detected and possibly fixed at once.

The code being tested should *not* call `log.Fatal` or `os.Exit` as those will halt the test execution as well.
Since the testing tool ignores the `main()` function, calling `log.Fatal` or `os.Exit` should remain exclusive to the 
`main()` function

### Writing effective tests
@todo

#### Table-driven testing
@todo

#### Randomized testing
@todo

#### Black-box and white-box testing
A test that's written assuming nothing about the package other than its exposed API and documentation is called a 
*black-box test*.

In the same way, a test that has privileged information about the package being tested and access to the internal 
functions and data structures is called a *white-box test*.

Black-box tests are usually more robust, since as the software evolves and unless the exposed API changes, they require
fewer updates. They can also expose possible flaws in the API design.

@todo examples

#### External test packages

Used to solve a possible problem that might happen when writing tests: circular imports.

Golang does not allow circular imports (when a package X imports a package Y that in turn imports package X for example)

An example of this approach can be found on the official [`fmt` package](https://github.com/golang/go/blob/master/src/fmt/export_test.go).

## Benchmarks
Benchmark functions measure the performance of a program on a fixed workload and have the following signature:
```go
func BenchmarkName(b *testing.B) {
    // ...
}
```
By default, benchmarks are not executed, to tell to `go test` tool to run them, you need to add the `-bench=.` flag.

Benchmark functions also expose an integer field `b.N` which specifies the number of times the operation is being measured,
the benchmark runner does some initial measurements and then extrapolates the value of `N` to a value large enough to get
a stable timing measurement.


By adding the `-benchmem` flag to the `go test` tool

### Comparative benchmarks

A very common approach when writting benchmark functions is to create comparative benchmarks.
This is done when you want to measure how does a certain part of your program will behave as the input grows bigger.


```go
  - func benchmark(b *testing.B, size int) {}
  - func Benchmark10(b *testing.B { benchmark(b,10) }
  - func Benchmark100(b *testing.B { benchmark(b,100) }
```


## Example functions

