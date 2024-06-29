<!-- For license of this file, see LICENSE.md in the base dir. -->

Introduction to Moose64
=======================

Moose64 is like C but with pretty Horse64 syntax and basic plain OOP,
for maintainable high-performance code.
It's part of the [Horse64 ecosystem](https://horse64.org).

[For a 📋 features list, **go here**](/docs/Features.md).

To build Moose64 programs and to manage dependencies, use [horp
](https://codeberg.org/Horse64/core.horse64.org/src/branch/main/docs/Resources.md#horp)
like you would for Horse64 programs. The compiler for Moose64 is [moosec
](https://codeberg.org/Horse64/core.horse64.org/src/branch/main/docs/Resources.md#moosec)
and part of the [📥 SDK](
https://codeberg.org/Horse64/core.horse64.org/src/branch/main/docs/Resources.md#sdk).


How to learn Moose64
--------------------

Since Moose64 is a side language to Horse64, this introduction
expects you to know how Horse64 works. Based on its sibling,
Moose64 was constructed with the following changes:


### Static types and type annotations

Unlike Horse64 which is dynamically typed, Moose64 is
**statically typed** with required type annotations:

```Moose64
import std from m64.horse64.org

func main -> bool {
    var s <- std.str("Hello World!")
    if failed(std.str) {  # Note: this means string alloc failed.
        return no
    }
    defer s.destroy()

    print(s)
    return yes
}
```

For variables and function parameters, `<- ...type desc...` after
the declaration will annotate the type. For function return values,
`-> ...type desc...` right before the code block will annotate
the type.

Function pointers work by inlining the function header inside `(...)`
parenthesis:

```Moose64
func this_returns_a_func_ptr -> ((_<-i32) -> bool) {
    return std.as_ref(my_other_func)
}
```


### Memory management

Moose64 has no garbage collection, instead all used data
must be destroyed manually.

Usually, this is achieved by:

- for structs, by using a `defer x.destroy()`
  right after some [struct instance](#oop-with-structs)
  `x` was successfully initialized via `.init` or an
  init constructor short-hand:

  ```Moose64
  var s <- std.str("Hello World!")  # Short-hand for s.init("Hello World")
  if failed(std.str) {
      return -1
  }
  defer s.destroy()
  # ...use `s` string here for whatever purpose...
  ```

- for allocated buffers or struct [`ref`s or `c_array`s](#ref-types),
  by using a `defer std.unalloc(x)` right after some
  `x = std.alloc(...)` to make sure the allocation is released
  again later:

  ```Moose64
  var s <- byte c_array = std.alloc(256)
  defer std.unalloc(s)
  # ...use `s` array here for whatever purpose...
  ```

- for allocated arrays by using a `defer std.unalloc_array(x)`:

  ```Moose64
  
  var s <- byte array = std.alloc_array(std.size_of(std.deref(s)), 256)
  defer std.unalloc_array(s)
  # ...use `s` array here for whatever purpose...
  ```

Of course you're allowed to keep values around for longer
as well rather than destroy them at the end of the block,
but then you have to ensure they get destroyed or unalloc'ed
somewhere later.

The `defer` keyword
runs a call delayed at the end of the current block,
which can e.g. be used for safe resource cleanup.


### OOP with structs

While Horse64's approach to user types is the `type` statement
which allows complex **object-oriented programming (OOP)**,
Moose64's **default user type is the `struct`** statement.

The biggest difference is that struct instances are **passed
by value,** unless you use an explicit [`ref` variable](#ref-types) to
pass them around, unlike in Horse64 where user types
are passed by reference.

Also, structs don't allow any inheritance when using
object-oriented functions.

However, structs can be populated with func attributes
via `func name_of_struct.func_attr_name ...` just like in
Horse64, and extending structs via `extend struct` also
works just like in Horse64. Therefore, other than type
inheritance, all basic object-oriented pattern are available.


### `ref` types

Types marked with `ref` or with `c_array` are equivalent to
C's pointers.


### Function calls and overrides

Unlike Horse64, Moose64 does **not allow so-called "keyword
arguments"** that specify any default values. However,
Moose64 **has function overriding** for func attributes
on [structs](#oop-with-structs), which works as follows:

```Moose64
struct MyTestStruct {
}

func MyTestStruct.hello failable {
    var s <- std.str("Hello there!")
    if failed(std.str) {
        return failed
    }
    defer s.destroy()

    print(s)
}

func MyTestStruct.hello_with_num(num <- f64) failable {
    var s <- std.str("Hello there! Here's a number: ")
    if failed(std.str) {
        return failed
    }
    defer s.destroy()

    s.add(num)
    if failed(s.add) {
        return failed
    }

    print(s)
}

func this_is_a_test failable {
    var my_struct <- MyTestStruct

    my_struct.hello(5)  # Will call .hello_with_num() override.
    if failed(my_struct.hello) {
        return failed
    }
}
```

Basically, if you call any func attribute (like `.hello`)
where there is a 2nd func attribute that starts with
the same name followed by an underline and a suffix (like
`.hello_with_num`), the 2nd func attribute is considered an
alternate override.

Any alternate override will be picked when the arguments
given, including their types, don't match the ones required
by the original func attribute but do match the ones of the
override.

