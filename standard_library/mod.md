# Rust 标准库

Rust 标准库是编写Rust软件的基础，提供了语言层面的基础共享抽象，如一些 Traits 和 Types。

> **Dev**
> 标准库的源代码位于 `rust-lang/rust` repo 的 `/library` 目录下。
> 标准库主要由三个 crates 组成：
> - `core` (Core Library)：是标准库的基础，no dependent upstream library。It is the portable glue between the language and its libraries, defining the intrinsic and primitive building blocks of all Rust code. The core library is minimal and platform-agnostic.
> - `alloc`: 依赖于 `core` lib, 提供了基于堆内存管理的 smart pointer 和 collections。
> - `std`: 依赖于 `core` 和 `alloc` libs, provides a set of minimal and battle-tested shared abstractions for the broader Rust ecosystem.
>   - 通常 programmer 仅使用 `std`，并且几乎所有 Rust program 默认 `use std;`，即依赖 `std`。`core` 和 `alloc` 的很多 APIs 会在 `std` re-exported。
>   - 但当一个 crate 应用了 `#![no_std]` attribute，则不依赖 `std` lib。
> - `core`, `alloc`, `std` 三个 crates 的依赖关系如下图：
> 
> ```mermaid
> graph TD;
>     std --> core
>     std --> alloc
>     alloc --> core
> ```



`std` APIs 可以分成四个部分：
- `std::*` modules
- primitive types
- standard macros
- the rust prelude

Rust Standard Library 分成很多个更聚焦的 modules，它们是构建 Rust 软件的基础部件。每个 module 的实现了相对原子化的特性，如 `std::cmp` module 实现了 ordering and comparing values 的一些工具（traits, functions, macros 等）。

虽然基础数据类型 (primitive types) 是由 compiler 实现的，但 standard library 内也为 primitives 添加了一些方法。



- Trait and Generic Types
- Function
- Enum
- Struct