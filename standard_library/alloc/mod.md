- [`Box`](#box)
- [`Rc` and `Arc`](#rc-and-arc)
- [`alloc` module](#alloc-module)
- [`vec` module](#vec-module)
  - [`Vec` struct: a contiguous growable array type.](#vec-struct-a-contiguous-growable-array-type)
- [`str`](#str)
- [`String`](#string)
- [`Box<T>`](#boxt)
- [`Rc<T>`](#rct)
- [`Weak<T>`](#weakt)

The `alloc` library provides smart pointers and collections for managing heap-allocated values.

## `Box`

- 根据文档 `Box` Type 是一个 smart pointer type, 但将 `Box` 理解成一个 container 会更加形象。当使用 `Box::new` 来创建一个 pointer value 时，Rust 会在 heap 上分配内存存储 contained content，并在 stack 上创建一个 owning pointer，通常把这个 stack pointer 称为一个 `Box` value。和普通 value 一样 Box value 也只能有唯一的 owner。
- 由于 Rust 在编译阶段需要知道每个数据类型的内存大小，这给递归形式的数据结构的定义带来了很大困难。递归形式数据结构的典型例子就是链表的节点，节点类型的 fields 包含了节点类型自身。`Box` 可以很好地解决这个问题，通过为递归的 field 提供一层封装。
- 在概念理解上 Box 和普通的 value 差不多，只是存储机制不同；而在使用上 Box 与 Reference 基本一样，比如 `mut Box<String>` 和 `&mut String`。

## `Rc` and `Arc`

- `Rc/Arc` 在内部实现原理上和 `Box` 很像，也可以理解成一个 container。在使用 `Rc::new` 或 `Arc::new` 创建一个对应的 pointer value 时，Rust 都会在 Heap 上分配内存存储 contained content，在 stack 上创建一个 owning pointer。
  - 但和 `Box` 不同的是 `Rc/Arc` pointer 可以有多个 owner，共同 share ownership。当所有的 owner 对被 drop 之后，heap space 才会释放。
  - 在通过 clone 创建新的 shared owners 时，会在 stack 上创建一个新的 pointer，其指向同一个 heap container。
  - Rust 是内存和线程安全的，由于 `Rc/Arc` 有多个 owner，所以 `Rc/Arc` 是 immutable 的。
  - `Rc` 和 `Arc` 类型基本一致，唯一差别就是 `Arc` 可以在线程间共享，而 `Rc` 只能在一个线程内共享。
- 软件设计中 `Rc/Arc` 常用来定义公共数据，然后在多个主体（如线程）共同持有。`Rc/Arc` 类型也常常和具有 interior mutability 的类型（如 `Cell`, `RefCell`, `Mutex`, Atomics）一起使用，可以在多个主体之间进行同步数据。
  - Interior mutability 简单理解就是一个变量没有使用 `mut` 定义，但其内容却依然是 mutable 的。
- 在一般的使用上，Rc 与 reference 没有太大差别，比如一个 `Rc<String>` 和 `&String` 一样使用。

Inherited mutability

## `alloc` module

- 这个 module 包含了一些 memory allocation APIs, 比较 low-level, 不直接使用。
- `alloc` 在 heap 上分配一块指定 layout 的内存
- `dealloc` 在 heap 上指定位置上释放指定 layout 的内存
- `realloc` 在 heap 上已分配位置处，改变原有分配内存的大小 (shrink or grow)

    ```rust
    use std::alloc::{alloc, dealloc, Layout};

    unsafe {
        let layout = Layout::new::<u16>();
        let ptr = alloc(layout);

        *(ptr as *mut 16) = 42;
        assert_eq!(*(ptr as *mut 16), 42);

        dealloc(ptr, layout);
    }
    ```

## `vec` module

- 这个模块定义了一些与 Vector 的核心 struct `Vec` 和三个相关的 Iterator。
  - `Drain`: mutable borrow vector 的 slice 创建一个 Iterator
    ```rust
    let mut v = vec![1, 2, 3, 4];
    {
        let iter = v.drain(1..3);
        println!("{:?}", iter); // Drain([2, 3])
    }
    println!("{:?}", v); // [1, 4]
    ```
  - `IntoIter`: move vector 创建一个 Iterator
    ```rust
    let v = vec![1, 2, 3];
    let iter = v.into_iter();
    println!("{:?}", iter); // IntoIter([1, 2, 3])
    println!("{:?}", v); // Error: borrow after moved
    ```
  - `Splice`: 一个 iterator。`vec.splice(Range, Iter)` 方法使用传入的 Iterator 替换自身的片段，然后被替换的片段作为 `Splice` 返回。不太好理解，不知道典型使用场景。
    ```rust
    let mut v = vec![1, 2];
    {
        let new = [7, 8];
        let iter = v.splice(1.., new);
        println!("{:?}", iter.collect::<Vec<_>>()); // [2]
    }
    println!("{:?}", v); // [1, 7, 8]
    ```
- Vector 是特定类型 `T` 的数据集，这些 `T` values 连续地分布在一片 heap 内存上。Vector 的大小是动态的，可以不断插入或抛出 `T` values。
- Vector 的索引效率是 `O(1)`, 尾部 pushing/poping value 是 `O(1)`。

### `Vec` struct: a contiguous growable array type.

- 数据类型定义 `pub struct Vec<T, A: Allocator = Global> { /* private fields */ }`
- 一个 vector 在内存上的底层表示：
    ```
              pointer      len    capacity
            +------------+-------+-------+
    Stack   |  0x01234   |   2   |   4   |
            +------------+-------+-------+
                  |
                  V
              +------+------+--------+--------+
    Heap      |  'a' |  'b' | uninit | uninit |
              +------+------+--------+--------+
    ```
- `Vec` will never automatically shrink itself, even if completely empty.
- `push` and `insert` will (re)allocate when `len == capacity`.
- `Vec` 的相关方法：
  - `Vec::new()`, `Vec::with_capacity(usize)`，宏 `vec![]`
  - `vec.push(T)`, `vec.insert(usize, T)`, `vec.extend(iter: T)`
  - `vec.append(Vec)`, `vec.split_off(usize) -> Vec`
  - 索引：`vec[0]`,切片：`vec[..3]`, `&vec`
  - `vec.pop()`, `vec.swap_remove(usize) -> T`, `vec.remove(usize)`, `vec.drain(Range) -> Iterator`, `vec.clear()`
  - `vec.len()`, `vec.is_empty()`
  - `vec.sort()`, `vec.sort_unstable()`, `vec.sort_by()`, `vec.sort_by_key()`
  - `vec.concat()`, `vec.join(Separator)`

## `str`

- Unicode string slices. 典型的 `str` 是字符串字面值，如 `"hello"`, 生命周期为 `'static` 即这个程序的生命周期。
- alloc 为 `&str` 定义了一些相关的 iterators
  - `Bytes`: an iterator over bytes of a string slice, `str.bytes()`
  - `Chars`: an iterator over chars of a string slice, `str.chars()`
  - `Lines`: an iterator over the lines of a string, as string slices, `str.lines()`
  - `Matches`
  - `Split`

## `String`

- A `String` type has ownership over the contents of the string. 
- `String` 实现了 `Deref<Target = str>` trait，所有的 `str` 方法可以直接应用在 `String` 上。
- 创建一个 string 的方法：
    ```rust
    let s = "hello".to_string();
    let s: String = "hello".into();
    let s = String::from("hello");
    let s = format!("{}", 123);
    ```
- `str` 和 `String` 都不支持索引操作，因为索引操作是一个 `O(1)` 的操作，而 `String` 的每个字符长度可能不同，无法进行 `O(1)` 的索引。

## `Box<T>`

- `Box<T>` is a pointer type, providing the simplest form of heap allocation in Rust.
- Box pointer 具有 heap application 的 ownership，当 Box pointer 被 drop 时，heap allocation 也会自动释放。
- Fixed-sized type values 默认时分配在 stack 上的，Box 就提供了将其手动分配在 heap 上的方式。
    ```rust
    let val: u8 = 5;  // allocated on stack
    let boxed: Box<u8> = Box::new(5); // allocated on heap
    ```
- Box 也合适来创建 recursive data structure:
    ```rust
    #[derive(Debug)]
    enum List<T> {
    Cons(T, Box<List<T>>),
    Nil,
    }
    let list = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
    ```
- `Box<T>` 的常用方法
	- `Box::new(x: T)` allocates memory on the heap and then places `x` into it.
	- `Box::from_raw(raw: *mut T)` constructs a box from a raw pointer.
	- `Box::into_raw(b: Self) -> *mut T` consumes the `Box`, returning a wrapped raw pointer.
	- `Box::leak<'a>(b: Self) -> &'a mut T` consumes and leaks the `Box`, returning a mutable reference `&'a mut T`.
		- This function is mainly useful for data that lives for the remainder of the program's life.
		- Dropping the returned reference will cause a memory leak.

## `Rc<T>`

- `Rc<T>` is a single-threaded reference-counting pointer, providing shared ownership of a value of type `T`, allocated in the heap.
- `Rc` value 的 clone 方法会创建一个新的 pointer 指向相同的 heap allocation。
- `Rc` contained value 是 immutable 的。如果需要 mutability, `Rc` contain value 可以设置成 `Cell` 或 `RefCell` type.
- The `downgrade` method can be used to create a non-owning `Weak` pointer.
	- A `Weak` pointer can be `upgrade` to an `Rc`, but this will return `None` if the value stored has been dropped.
- `Rc<T>` 常用的方法有
	- `Rc::new(value: T)` constructs a new `Rc<T>`
	- `Rc::into_raw(this: Self) -> *const T` consumes the `Rc`, returning the wrapped pointer.
	- `Rc::as_ptr(this: &Self) -> *const T` provides a raw pointer to the data.
	- `Rc::from_raw(ptr: *const T) -> Self` constructs an `Rc<T>` from a raw pointer.
	- `Rc::downgrade(this: &Self) -> Weak<T>` creates a new `Weak` pointer to this allocation.
	- `Rc::weak_count(this: &Self) -> usize` gets the number of `Weak` pointers to this allocation.
	- `Rc::strong_count(this: &Self) -> usize` gets the number of strong (`Rc`) pointers to this allocation.
	- `Rc::ptr_eq(this: &Self, other: &Self) -> bool` returns `true` if the two `Rc`s point to the same allocation.

## `Weak<T>`

- `Weak` is a version of `Rc` that holds a non-owning reference to the managed allocation.
- A `Weak` pointer is useful for keeping a temporary reference to the allocation managed by `Rc` without preventing its inner value from being dropped.
- It's also used to prevent circular references between `Rc` pointers.
- `Weak<T>` 常用的方法有
    - `weak.upgrade() -> Option<Rc<T>>` attempts to upgrade the `Weak` pointer to an `Rc`, delaying dropping of the inner value if successful.
    - `weak.as_ptr() -> *const T` returns a raw pointer to the object `T` pointed to by this `Weak<T>`.
    - `weak.into_raw() -> *const T` consumes the `Weak<T>` and turns it into a raw pointer.
    - `Weak::from_raw(ptr: *const T)`
