
## Problem statement
Why choose between messy input ([`println!`](https://doc.rust-lang.org/std/macro.println.html)) and messy output ([`dbg!`](https://doc.rust-lang.org/std/macro.dbg.html))? `printc!` allows you to debug code with clean input and clean output. Simultaneously.

## Motivation, use-cases
### Short motivation: 
If you want to debug, you're normally forced to choose between writing long boilerplate ([`println!`](https://doc.rust-lang.org/std/macro.println.html)) and having messy output that requires more effort to visually navigate ([`dbg!`](https://doc.rust-lang.org/std/macro.dbg.html)). The ability to produce clean output from clean input allows you to analyze your code more easily and learn faster.

### Long motivation:
If we want to debug code, [the Standard Library](https://doc.rust-lang.org/std/) offers the following options:
* [`dbg!`](https://doc.rust-lang.org/std/macro.dbg.html)
* [`println!`](https://doc.rust-lang.org/std/macro.println.html)
* [`print!`](https://doc.rust-lang.org/std/macro.print.html)
* [`eprint!`](https://doc.rust-lang.org/std/macro.eprint.html)
* [`eprintln!`](https://doc.rust-lang.org/std/macro.eprintln.html)

[`print!`](https://doc.rust-lang.org/std/macro.print.html) and [`eprint!`](https://doc.rust-lang.org/std/macro.eprint.html) are rarely used because the user can instead choose [`println!`](https://doc.rust-lang.org/std/macro.println.html) and [`eprintln!`](https://doc.rust-lang.org/std/macro.eprintln.html) to get the same thing but with clearer structure to more easily distinguish different outputs from each other (although there may be some use cases where [`print!`](https://doc.rust-lang.org/std/macro.print.html) and [`eprint!`](https://doc.rust-lang.org/std/macro.eprint.html) would be preferable). Thus, [`print!`](https://doc.rust-lang.org/std/macro.print.html) and [`eprint!`](https://doc.rust-lang.org/std/macro.eprint.html) are suboptimal choices for producing clean output with minimal boilerplate. 

[`println!`](https://doc.rust-lang.org/std/macro.println.html) and [`eprintln!`](https://doc.rust-lang.org/std/macro.eprintln.html) produce clean output, but they require you to write long boilerplate in the input.

[`dbg!`](https://doc.rust-lang.org/std/macro.dbg.html) takes clean input, but the output is messy and takes more effort to visually navigate. 

For instance, you may want to copy code examples from documentation and forums (like [github](https://github.com/)) and paste them into your local environment (or [Rust Playground](https://play.rust-lang.org/)) to play around with it, to better understand what the code actually does. We will demonstrate how the messy input of [`println!`](https://doc.rust-lang.org/std/macro.println.html) and messy output of [`dbg!`](https://doc.rust-lang.org/std/macro.dbg.html) looks like, and compare it with `printc!`.

Let's take the below [code example](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&code=fn%20main()%20%7B%0A%20%20%20let%20mut%20test1%20%3D%20Test%3A%3Anew(%22test1%22)%3B%0A%20%20%20let%20mut%20test1_pin%20%3D%20unsafe%20%7B%20Pin%3A%3Anew_unchecked(%26mut%20test1)%20%7D%3B%0A%20%20%20Test%3A%3Ainit(test1_pin.as_mut())%3B%0A%0A%20%20%20drop(test1_pin)%3B%0A%20%20%20println!(r%23%22test1.b%20points%20to%20%22test1%22%3A%20%7B%3A%3F%7D...%22%23%2C%20test1.b)%3B%0A%0A%20%20%20let%20mut%20test2%20%3D%20Test%3A%3Anew(%22test2%22)%3B%0A%20%20%20mem%3A%3Aswap(%26mut%20test1%2C%20%26mut%20test2)%3B%0A%20%20%20println!(%22...%20and%20now%20it%20points%20nowhere%3A%20%7B%3A%3F%7D%22%2C%20test1.b)%3B%0A%7D%0Ause%20std%3A%3Apin%3A%3APin%3B%0Ause%20std%3A%3Amarker%3A%3APhantomPinned%3B%0Ause%20std%3A%3Amem%3B%0A%0A%23%5Bderive(Debug)%5D%0Astruct%20Test%20%7B%0A%20%20%20%20a%3A%20String%2C%0A%20%20%20%20b%3A%20*const%20String%2C%0A%20%20%20%20_marker%3A%20PhantomPinned%2C%0A%7D%0A%0A%0Aimpl%20Test%20%7B%0A%20%20%20%20fn%20new(txt%3A%20%26str)%20-%3E%20Self%20%7B%0A%20%20%20%20%20%20%20%20Test%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20a%3A%20String%3A%3Afrom(txt)%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20b%3A%20std%3A%3Aptr%3A%3Anull()%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%20This%20makes%20our%20type%20%60!Unpin%60%0A%20%20%20%20%20%20%20%20%20%20%20%20_marker%3A%20PhantomPinned%2C%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%0A%20%20%20%20fn%20init%3C%27a%3E(self%3A%20Pin%3C%26%27a%20mut%20Self%3E)%20%7B%0A%20%20%20%20%20%20%20%20let%20self_ptr%3A%20*const%20String%20%3D%20%26self.a%3B%0A%20%20%20%20%20%20%20%20let%20this%20%3D%20unsafe%20%7B%20self.get_unchecked_mut()%20%7D%3B%0A%20%20%20%20%20%20%20%20this.b%20%3D%20self_ptr%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20fn%20a%3C%27a%3E(self%3A%20Pin%3C%26%27a%20Self%3E)%20-%3E%20%26%27a%20str%20%7B%0A%20%20%20%20%20%20%20%20%26self.get_ref().a%0A%20%20%20%20%7D%0A%0A%20%20%20%20fn%20b%3C%27a%3E(self%3A%20Pin%3C%26%27a%20Self%3E)%20-%3E%20%26%27a%20String%20%7B%0A%20%20%20%20%20%20%20%20assert!(!self.b.is_null()%2C%20%22Test%3A%3Ab%20called%20without%20Test%3A%3Ainit%20being%20called%20first%22)%3B%0A%20%20%20%20%20%20%20%20unsafe%20%7B%20%26*(self.b)%20%7D%0A%20%20%20%20%7D%0A%7D) from [this](https://rust-lang.github.io/async-book/04_pinning/01_chapter.html) page:
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
   printc!(test1);
   
   let mut test1_pin = unsafe { Pin::new_unchecked(&mut test1) };
   Test::init(test1_pin.as_mut());
   
   drop(test1_pin);
   printc!(test1);

   let mut test2 = Test::new("test2");
   printc!(test1, test2);
   
   mem::swap(&mut test1, &mut test2);
   printc!(test1, test2);
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
