Primitive Types

- [Bool type](#bool-type)
- [`u8` as Byte](#u8-as-byte)
- [Integer Types](#integer-types)
- [Float Types](#float-types)
- [`char` Type](#char-type)
- [`str` string slice](#str-string-slice)
- [slice `[T]`](#slice-t)

总结每种类型的常用运算/方法，以及类型间的转换。

## Bool type

- 一个 `bool` 类型变量只有两种可能取值 `true`, `false`。
- Bool 值之间支持 Bool 运算：与 `&`, 或 `|`, 非 `!`
- Bool 值可以转换成整数，但不能反向转换。
    ```rust
    assert_eq!(false as u8, 0);
    assert_eq!(i23::from(true), 1);
    let a: bool = 1u32 as bool; // cannot cast as `bool`- [Primitive Types](#primitive-types)
  - [Bool type](#bool-type)
  - [`u8` as Byte](#u8-as-byte)
  - [Signed Integer Types](#signed-integer-types)
    ```
- Bool值可以从 string 解析
    ```rust
    use std::str::FromStr;
    assert_eq!(FromStr::from_str("true"), Ok(true));
    ```

## `u8` as Byte

- 除了作为无符号整数，`u8` 也是字节类型。`let a: u8 = b'w';`
- 相关的方法有:
  - `is_ascii`, `is_ascii_digit`, `is_ascii_punctuation` 等，检查是否为特定类别的字符
  - `make_ascii_lowercase`, `make_ascii_uppercase` 原位的大小写转换
  - `to_ascii_lowercase`, `to_ascii_uppercase` 生成大小写转换的 copy

## Integer Types

- 根据占用的比特数，有符号整型包括了 `i8`, `i16`, `i32`, `i64`, `i128`；无符号整型包括了 `u8`, `u16`, `u32`, `u64`, `u128`。
- 支持加减乘除，取模，对数，指数等各种数学运算函数。多数的数学函数都有 `checked_*`, `overflowing_*`, `saturating_*`, `wrapping_*` 为前缀的四种版本。
  - `checked_*` 当计算过程发生溢出时会返回 `None`
  - `overflowing_*` 返回一个元组`(T, bool)`，包含计算结果和一个是否溢出的标识。当溢出发生时，计算结果为 wrapped value
  - `saturating_*` 当计算过程发生溢出，返回类型的最大值或最小值。
  - `wrapping_*` 当计算结果发生溢出，返回 wrapped value。`MAX + 1` 的 wrapped value 就是 `MIN`。
- 整数类型之间可以进行转换。
  - 当源类型的取值范围包含于目标类型是使用 `as` 可以正常转换，如 `100u8 as i32`；但当源类型的取值范围不包含于目标类型时就会发生 wrapping，比如 `256i32 as u8` 的转换结果为 `0`，这有时会导致 unexpected behavior。
  - 对于每种可能发生 wrapping 的转换，Rust 都为该整型实现了 `TryFrom` trait，比如 `u8::try_from(256i32)`，转换结果为 `Result` 类型，当转换过程发生 overflow 时，就会返回错误。
- 从 `&str` 可以解析整型值，如 `i32::from_str("23")`

## Float Types

- 根据占用的比特数，浮点类型有 `f32` 和 `f64` 两种。浮点类型的取值除了普通的数值，还有 `NAN`, `INFINITY`, `NEG_INFINITY`。
- 浮点数支持各种三角、指数、对数、根号、取整等运算。
- 值得注意的是，`f32` 和 `f64` 包含非正常取值，所以没有实现 `std::cmp::Ord` trait。没法像整数那么方便地进行排序。
    ```rust
    let mut vec_int = vec![1, 5, 10, 2, 15];
    vec_int.sort();
    let mut vec_flt = vec![1.1, 1.15, 5.5, 1.123, 2.0];
    vec_flt.sort_by(|a, b| a.partial_cmp(b).unwrap());
    ```
- 将整数转换为浮点数，可能会出现精度的严重损失，或者得到 `Inf`:
    ```rust
    let a = u64::MAX;
    let b = a as f32;
    println!("{}, {}", a, b); // 18446744073709551615, 18446744000000000000

    let a = u128::MAX;
    let b = a as f32;
    println!("{}", b); // inf
    ```
- 可以从 `&str` 解析出浮点数，`f32::from_str("3.14")`

## `char` Type

- `char` 是一个 Unicode 标量值，或 Unicode 码点。`char` 占四个字节，与 `u32` 一致，但 `char` 的取值范围比 `u32` 要小，有效的 `char` 取值于 `'\0'..='\u{D7FF}'` 和 `'\u{E000}'..='\u{10FFFF}'` 两个区间。
- `char` 的总是四个字节，而 `String` 中的字符是 UTF8 编码的，长度不固定。
    ```rust
    let v = vec!['h', 'e', 'l', 'l', 'o'];
    assert_eq!(20, v.len() * std::mem::size_of::<char>());
    let s = String::from("hello");
    assert_eq!(5, s.len() * std::mem::size_of::<u8>());
    ```
- `char` 的很多方法与 `u8` byte 相同，如 `is_ascii`, `is_alphanumeric`, `is_ascii_punctuation`, `to_lowercase` 等
- 与 `u32` 进行转换：`'@' as u32`, `char::from_u32(128175).unwrap()`

## `str` string slice

- 以UTF8编码的字符串切片，只能以借用的形式 `&str` 使用
- `&str` 引用是胖指针，包含了两部分 pointer 和 length of bytes。可以通过 `.as_ptr()` 和 `.len()` 分别获得。
  - `.len()` 是包含的字节数，而不是字符数
- 常用的方法：
  - `as_bytes`, `chars`, `lines`
  - `contains`, `end_with`, `starts_with`, `find`, `rfind`,
  - `into_string`, `parse`
  - `is_empty`, `is_ascii`, 
  - `matches`, `rmatches`, `replace`, `split`, `rsplit`, `trim`
  - `to_lowercase`, `to_uppercase`, `repeat`
  - `as_ptr`,

## slice `[T]`

- Slice is view into a block of memory represented as a pointer and a length.


Rust 中几乎每种运算符都对应着一个 trait。