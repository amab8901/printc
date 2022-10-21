
## Problem statement
Why should the user have to choose between messy input and messy output? My proposed macro will enable the user to debug their code with clean input and clean output.

## Motivation, use-cases
### Short motivation: 
If you want to debug, you're currently forced to choose between writing long boilerplate and having messy output that requires more effort to visually navigate. It's important to be able to produce clean and clear output without having to write long boilerplate, especially for beginners and intermediate-level users (I can't speak for advanced users though, since I haven't reached that level myself).  

### Long motivation:
If we want to debug code, we have the following options available:
* `dbg!`
* `println!`
* `print!`
* `eprint!`
* `eprintln!`

As I understand it, `print!` and `eprint!` are rarely used because the user can instead choose `println!` and `eprintln!` to get the same thing but with clearer structure to more easily distinguish different outputs from each other (although there may be some use cases of `print!` and `eprint!`). Thus, `print!` and `eprint!` are suboptimal choices for producing clean & clear output with minimal boilerplate. 

`println!` and `eprintln!` produce clean and clear output, but they require the user to write long boilerplate.

`dbg!` can be used very concisely and easily, with minimal boilerplate. But the output is messy and takes more effort to visually navigate. 

Beginners and intermediate-level users (such as myself) may want to copy code examples from documentation and forums (like github) and paste them into their local environment (or [Rust Playground](https://play.rust-lang.org/)) to play around with it, to better understand what the code actually does. Using `println!` and `eprintln!` is cumbersome and messy to write, while `cfg!` is cumbersome and messy to read its output. 

I took the below code example from [this](https://rust-lang.github.io/async-book/04_pinning/01_chapter.html) page:
```
fn main() {
   let mut test1 = Test::new("test1");
   let mut test1_pin = unsafe { Pin::new_unchecked(&mut test1) };
   Test::init(test1_pin.as_mut());

   drop(test1_pin);
   println!(r#"test1.b points to "test1": {:?}..."#, test1.b);

   let mut test2 = Test::new("test2");
   mem::swap(&mut test1, &mut test2);
   println!("... and now it points nowhere: {:?}", test1.b);
}
use std::pin::Pin;
use std::marker::PhantomPinned;
use std::mem;

#[derive(Debug)]
struct Test {
    a: String,
    b: *const String,
    _marker: PhantomPinned,
}


impl Test {
    fn new(txt: &str) -> Self {
        Test {
            a: String::from(txt),
            b: std::ptr::null(),
            // This makes our type `!Unpin`
            _marker: PhantomPinned,
        }
    }

    fn init<'a>(self: Pin<&'a mut Self>) {
        let self_ptr: *const String = &self.a;
        let this = unsafe { self.get_unchecked_mut() };
        this.b = self_ptr;
    }

    fn a<'a>(self: Pin<&'a Self>) -> &'a str {
        &self.get_ref().a
    }

    fn b<'a>(self: Pin<&'a Self>) -> &'a String {
        assert!(!self.b.is_null(), "Test::b called without Test::init being called first");
        unsafe { &*(self.b) }
    }
}
```
I tried to play around with this code for educational purposes, using two different approaches: `println!` and `dbg!`. In both cases, I modified only the `main` function as follows:

### `println!` approach
Messy input:
```
fn main() {
   let mut test1 = Test::new("test1");
   println!("test1 = {:#?}", test1);
   println!();
   
   let mut test1_pin = unsafe { Pin::new_unchecked(&mut test1) };
   Test::init(test1_pin.as_mut());
   
   drop(test1_pin);
   println!("test1 = {:#?}", test1);
   println!();

   let mut test2 = Test::new("test2");
   println!("test1 = {:#?}", test1);
   println!();
   println!("test2 = {:#?}", test2);
   println!();
   
   mem::swap(&mut test1, &mut test2);
   println!("test1 = {:#?}", test1);
   println!();
   println!("test2 = {:#?}", test2);
   println!();
}
```
Clean output:
```
test1 = Test {
    a: "test1",
    b: 0x0000000000000000,
    _marker: PhantomPinned,
}

test1 = Test {
    a: "test1",
    b: 0x00007ffd1bdafb68,
    _marker: PhantomPinned,
}

test1 = Test {
    a: "test1",
    b: 0x00007ffd1bdafb68,
    _marker: PhantomPinned,
}

test2 = Test {
    a: "test2",
    b: 0x0000000000000000,
    _marker: PhantomPinned,
}

test1 = Test {
    a: "test2",
    b: 0x0000000000000000,
    _marker: PhantomPinned,
}

test2 = Test {
    a: "test1",
    b: 0x00007ffd1bdafb68,
    _marker: PhantomPinned,
}
```

### `dbg!` approach
Clean input:
```
fn main() {
   let mut test1 = Test::new("test1");
   dbg!(&test1);
   
   let mut test1_pin = unsafe { Pin::new_unchecked(&mut test1) };
   Test::init(test1_pin.as_mut());
   
   drop(test1_pin);
   dbg!(&test1);

   let mut test2 = Test::new("test2");
   dbg!(&test1, &test2);
   
   mem::swap(&mut test1, &mut test2);
   dbg!(&test1, &test2);
}
```
Messy output:
```
[src/main.rs:7] &test1 = Test {
    a: "test1",
    b: 0x0000000000000000,
    _marker: PhantomPinned,
}
[src/main.rs:13] &test1 = Test {
    a: "test1",
    b: 0x00007ffca10e7650,
    _marker: PhantomPinned,
}
[src/main.rs:16] &test1 = Test {
    a: "test1",
    b: 0x00007ffca10e7650,
    _marker: PhantomPinned,
}
[src/main.rs:16] &test2 = Test {
    a: "test2",
    b: 0x0000000000000000,
    _marker: PhantomPinned,
}
[src/main.rs:19] &test1 = Test {
    a: "test2",
    b: 0x0000000000000000,
    _marker: PhantomPinned,
}
[src/main.rs:19] &test2 = Test {
    a: "test1",
    b: 0x00007ffca10e7650,
    _marker: PhantomPinned,
}
```

## Solution: `printc!`
If you use `printc!`, the input and output should both be clean, as follows:

Clean input:
```
fn main() {
   let mut test1 = Test::new("test1");
   pdbg!(&test1);
   
   let mut test1_pin = unsafe { Pin::new_unchecked(&mut test1) };
   Test::init(test1_pin.as_mut());
   
   drop(test1_pin);
   pdbg!(&test1);

   let mut test2 = Test::new("test2");
   pdbg!(&test1, &test2);
   
   mem::swap(&mut test1, &mut test2);
   pdbg!(&test1, &test2);
}
```

Clean output:
```
test1 = Test {
    a: "test1",
    b: 0x0000000000000000,
    _marker: PhantomPinned,
}

test1 = Test {
    a: "test1",
    b: 0x00007ffd1bdafb68,
    _marker: PhantomPinned,
}

test1 = Test {
    a: "test1",
    b: 0x00007ffd1bdafb68,
    _marker: PhantomPinned,
}

test2 = Test {
    a: "test2",
    b: 0x0000000000000000,
    _marker: PhantomPinned,
}

test1 = Test {
    a: "test2",
    b: 0x0000000000000000,
    _marker: PhantomPinned,
}

test2 = Test {
    a: "test1",
    b: 0x00007ffd1bdafb68,
    _marker: PhantomPinned,
}
```
