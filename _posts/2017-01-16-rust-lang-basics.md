---
layout: post
title: Rust 101
subtitle: Simple Notes on Rust Lang
---

Rust 101
========

Basic observations on Rust.

Boxes
=====

Any allocated memory in rust is `blocked` and therefore becomes a
`Box`. The box denotes safety / isolation.

A box is set up within the function header `Box<i32>`

And `Box:new(4)` allocates space on the for an `i32` and adds `4` to
it.

Pointers
========

Rust provides two pointer types; the owner pointer `~` and the borrowed
reference `&`

Both types point to a box, `~[&str]` is a owned reference to a vector of
borrowed strings.

Dereference
-----------

Similarly, dereferencing a Rust pointer is done with the star operator
`*`

The following block works as anticipated, printing out "10".

Ownership
---------

The following block gives a compilation error:

``` {.rust}
    let x = ~10;
    let y = x;
    println!("{:d}", *x);
```

``` {.bash}
tut2.rs:3:23: 3:24 error: use of moved value: `x`
tut2.rs:3     println!("{:d}", *x);
```

The compiler blocks this due to ownership, as the above would result in
two references to an owned box.

In other languages, such as C and Java, there are no restrictions on
pointer sharing.

clone
-----

We can make a copy of a box using the clone method which copies over the
content of a box and creates a new owned pointer to the copy.

``` {.rust}
let x = ~10;
    let y = x.clone();
    println!("{:d}", *x);
```

But, x and y refer to different boxes now — a modification of \*x will
not be visible through y.

Borrowed References
-------------------

Using the `&` operator creates a temporary reference to some memory that
has already been allocated.

Borrowed references are frequently used as function parameters. For
example,

``` {.rust}
fn borrow(r : &int) -> int {
    *r
}
```

declares the function borrow to take a borrowed pointer to an int. We
can call it by passing in an owned pointer:

``` {.rust}
 let x = ~10;
    println!("borrow(x): {:d}", borrow(x));
```
