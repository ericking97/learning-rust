# Async Rust

Rust provides support for asynchronous programming along other third party crates that provide a runtime.

## Learning resources

- [x] [Rust's Journey to Async/Await](https://www.youtube.com/watch?v=lJ3NC-R3gSI&ab_channel=InfoQ)
- [ ] [Async Rust Basic Concepts](https://dev.to/rogertorres/asynchronous-rust-basic-concepts-44ed)
- [ ] [tldr; on how async works](https://dev.to/rogertorres/rust-tokio-stack-overview-runtime-9fh)
- [ ] [Cambridge. Executor and Reactor](https://www.youtube.com/watch?v=9_3krAQtD2k&t=3004s&ab_channel=JonGjengset)
- [ ] https://blog.logrocket.com/a-practical-guide-to-async-in-rust/
- [ ] https://smallcultfollowing.com/babysteps/blog/2019/10/26/async-fn-in-traits-are-hard/
- [ ] https://github.com/dtolnay/async-trait
- [Async Book (Incomplete)](https://rust-lang.github.io/async-book/01_getting_started/01_chapter.html)

# Summaries

## Rust's Journey to Async/Await

### May read more about:

- Threads
- Blocking I/O vs Non-blocking I/O
- Event Loop

### Content

**Non Rust Concepts**

- Parallelism: Do multiple tasks at the same time
- Concurrency: Do multiple tasks in a different lapse of time
- Asynchronous: Features that enable parallelism and/or concurrency
- Native thread => 1:1 threading, tasks are provided by the operating system
  - By default they are heavy and very OS dependant for how the scheduler works
- Green threads => N:M threading, tasks are provided by your programming language
  - Are lighter, which in turn means that you can create as much as you want. (Like Go)
  - Stack growth can cause issues

**History**

> A "systems programming language" that doesn't let you use the system threads?

Since Rust v1.0 aimed to be a system programming language, support for native OS threads was a **must**. In earlier versions of Rust, both support for green threads and system threads were achieved, but since they both implemented the _Runtime_ trait, the green threads were not light anymore.

Therefore green threads were removed by the release of Rust 1.0 and shipped with OS thread support only.

The problems started to raise when developers started doing many I/O operations, since native threads were the only tool for asynchronous programming and these threads are heavy by design. The Core Team still wanted to give other alternative than OS Threads and they recognized two main operations:

- CPU Bound Operations, are based on the speed the CPU takes to completing a task. Parallelism is better suited here. (OS Threads)
- I/O Bound Operations, are based on doing a lot of Input/Output operations which are usually very simple. Concurrency is better suited here.

They started to look on how other languages tackle this problem

- Go was working with Asynchronous blocking I/O using **Green Threads**
- Node.js was an Asynchronous non-blocking I/O via the Event Loop provided by the V8 Engine

Since **Green Threads** couldn't be implemented since they would have to share the _Runtime_ trait and wouldn't be any different to the native threads.

Node.js was having problems dealing with the [callback hell](http://callbackhell.com/), which was later replaced by **Promises** which were later replaced by **async/await**. This solution was the most interesting.

The concept of **Promises** spawn the concept of **Futures** in Rust, but they were both inspired by Twitter employee Marius Eriksen article ["Your Server as a function"](https://monkey.org/~marius/funsrv.pdf) in which was one of the first conceptualization of **Future**.

For this concepts to work they needed a runtime, luckily Nginx had already a concept of an "Event Loop"

As of right now, Rust still doesn't offer and wouln't offer a Runtime for Futures, but there are multiple crates that offer it:

- Tokio
- Actix-web (Tokio fork)
- Async_std

Support for I/O operations are nowadays more important than ever for building scalables web applications: https://www.techempower.com/benchmarks/

Notes:

- Sketchy JS Promise => Rust Future => JS Promise on WebAssembly: https://imgur.com/BvspSqH.png

## Basic Async Rust concepts

- async is used to create an asynchronous block or function, making it return a Future.
- .await will wait for the completion of the future and eventually give back the value (or an error, which is why it is common to use the question mark operator in .await?).
- Future is the representation of an asynchronous computation, a value that may or may not be ready, something that is represented by the variants of the Poll enum.
- Poll is the enum returned by a future, whose variants can be either Ready<T> or Pending.
- poll() is the function that works the future towards its completion. It receives a Context as a parameter and is called by the executor.
- Context is a wrapper for Waker.
- Waker is a type that contains a wake() function that will be called by the reactor, telling the executor that it may poll the future again.
- Executor is a scheduler that executes the futures by calling poll() repeatedly.
- Reactor is something like an event loop responsible for waking up the pending futures.

## Rust asynchronous support

Rust allows the creation of concurrent applications via the async/await syntax

```
async fn foo() -> i32 {
    11
}

fn bar() {
    let x = foo();

    // it is possible to .await only inside async fn or block
    async {
        let y = foo().await;
    };
}

```

Blocks and functions declared with async desugar into a block of function that returns an implementation of a trait called **Future**
