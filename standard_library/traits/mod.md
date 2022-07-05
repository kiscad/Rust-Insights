## `AsRef`, `AsMut`

- 一般地，我们可以使用 `&` 和 `&mut` 运算符来生成一个 type `T` value 的 reference, 这是因为 Rust 为所有的类型都实现了 `AsRef` 和 `AsMut` 两个 traits。
    ```rust
    trait AsRef<T: ?Sized> { fn as_ref(&self) -> &T; }
    trait AsMut<T: ?Sized> { fn as_mut(&mut self) -> &mut T; }
    ```

- 这两个 traits 包含了 generic type，所以也可以一个类型 `T1` 实现 `AsRef<T2>` 和 `AsMut<T2>`，当两种类型的底层存储一致时。
    - Rust 为 `Vec<T>` 实现了 `AsRef<[T]>`
    - Rust 为 `String` 实现了 `AsRef<str>` 和 `AsRef<[u8]>`
    - Rust 为 `String`, `str`, `OsString`, `OsStr`, `PathBuf`, `Path` 都实现了 `AsRef<Path>`

- 这种机制为代码编写提供了引用的多态，比如一个参数类型为 `&T` 的函数可以接受所有实现了 `AsRef<T>` 的类型引用。
    - `AsRef` is typically used to make functions more flexible in the argument types they accept.

## `Borrow` and `BorrowMut`

- `Borrow` trait 的定义和 `AsRef` 作用相似，都是提供 reference type conversion。但 `Borrow` trait 多了一个约束: a type should implement `Borrow<T>` only when a `&T` hashes and compares the same way as the value it's borrowed from.
- `Borrow` is designed to address a specific situation with generic hash tables and other associative collection types.
- 比如有一个 `HashMap<String, i32>`，每次调用其 `get` 方法时，我们不希望在 heap 创建一个 String，然后传给 `get` 再 drop。这样非常低效。
- `HashMap` 的 `get` 方法定义为：
  ```rust
  impl<K, V> HashMap<K, V> where K: Eq + Hash {
    fn get<Q: ?Sized>(&self, key: &Q) -> Option<&V>
    	where K: Borrow<Q>, Q: Eq + Hash
    { ... }
  }
  ```
- 从定义 `get` 可以接收任何 `K` type 实现了的 `Borrow` 类型。`String` 实现了 `Borrow<str>` 和 `Borrow<String>`。那么 `HashMap<String, i32>` 的 `get` 方法就接收 `&str` 或 `&String` 类型参数。

## `Deref` and  `DerefMut`

- 通过为一个类型实现 `Deref` 和 `DerefMut` traits, 可以定义 `*` 和 `.` 运算符的行为。
    - 比如一个 smart pointers `Box<T>` 本身是一个 stack value，而 `Box` 通过实现 `Deref` trait，所以 `Box<T>` 可以像 `&T` 一样来使用，如调用 `T` 的方法。
- 另外 `Deref` 和 `DerefMut` trait 还可以实现自动类型转换的效果，即 **deref coercions**。
    - 比如 `String` 实现了 `Deref<Target=str>` trait，从而一个 string value 不仅可以调用自身的方法，还可以调用 `str` 的所有方法。
    - `Vec<T>` 实现了 `Deref<Target=[T]>` trait，从而所有接收 `&[T]` 的函数，也可以传入 `&Vec<T>`。
- The `Deref` and `DerefMut` traits are designed for implementing
    - smart pointer types, like `Box`, `Rc` and `Arc`,
    - and types that serve as owning version of something you would also frequently use by reference, the way `Vec<T>` and `String` serve as owning versions of `[T]` and `str`.
