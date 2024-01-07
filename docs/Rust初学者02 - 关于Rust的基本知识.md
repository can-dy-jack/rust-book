## 变量声明
Rust 中使用 `let` 来声明变量（~~这不是 JS 吗？🤣~~），默认变量是不可修改的，如果要声明可变化的变量可加 `mut` 参数。
```rust
let word = "hello";
word = "Hello"; // cannot mutate immutable variable `word`

let mut word2 = "world";
word2 = "world!"; // 修改变量word2
```
声明变量时可以声明变量类型，不指定变量类型的话 Rust 会自动推导其类型或者使用默认类型。整型默认为 `i32` 类型，浮点型默认为 `f64`类型。  
**注意**，使用 `mut` 指定变量可以变化时，指的是在该变量范围内变化，并不允许改变其类型！


## 注释
类似于 js， Rust的注释有单行注释和块注释。

```rs
// 单行注释
fn main() {
    /* 块注释 */
}
```

除此之外，Rust还支持特殊的注释：**文档注释**，其内容将被解析成 HTML 帮助文档。
```rust
/// 为接下来的项生成帮助文档
//! 为注释所属于的项（crate、模块或函数等）生成帮助文档
```
文档注释对于需要文档的大型项目来说非常重要。当运行 `rustdoc`，文档注释就会编译成文档。它们使用 `///` 标记，并支持 Markdown！


## 输出
在上一节中，我们第一个Rust程序就是输出 hello world，现在我们来了解一些输出相关的知识。

打印操作由[`std::fmt`](https://rustwiki.org/zh-CN/std/fmt/)里面所定义的一系列宏处理，包括：
-   `format!`：将格式化文本写到字符串。
-   `print!`：与 `format!` 类似，但将文本输出到控制台（io::stdout）。
-   `println!`: 在`print!`输出结果上追加一个换行符。
-   `eprint!`：与 `print!` 类似，但将文本输出到标准错误（io::stderr）。
-   `eprintln!`：在`eprint!`的基础上追加一个换行符。

以上函数（准确来说是[宏](https://rustwiki.org/zh-CN/rust-by-example/macros.html)，宏和函数的简单区别就是宏的名字都带`!`）都以相同的方式格式化文本，而且格式化的正确性会在编译时检查，而不是运行时报错。

这里列出一些常见的格式化文本的使用方式：

1. 正常输出
```rust
println!("println")
```
2. 插入变量
使用`{}`来占位，`{}`会被任意变量内容所替换，变量内容会转化成字符串。
```rust
let word = "hello";
println!("{}, world", word); // hello, world
```
用变量替换字符串可以使用位置参数（*不指定的话会按顺序来，且数量对应不上会报错*）。
```rust
println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob"); 
// Alice, this is Bob. Bob, this is Alice
```
除此之外，还可以在占位符内命名参数：
```rust
println!(
    "{who} {does} {what}",
    does = "are",
    who = "We",
    what = "frends!"
);
// We are frends!
```
3. 可以使用 `:` 指定特殊的格式，比如指定变量为二进制：
```rust
println!("{}用二进制表示为{0:b}", 2);
// 2用二进制表示为10
```
4. 还可以按指定宽度来右对齐文本：
```rust
println!("{number:>width$}", number="right", width=10); // "     right"
```
或者这样写：
```rust
println!("{0:>1$}", "right", 10); // "     right"
```

还可以为数字补0：
```rust
println!("{number:>0width$}", number=1, width=9); // "000000001"
// or
println!("{0:>01$}", 1, 9); // "000000001"
```

5. 指定显示浮点数的位数
```rust
println!("Pi is roughly {PI:.3}", PI = 3.141592); // Pi is roughly 3.142
```

以上使用的都是简单的基本类型变量的格式化输出，即 `fmt::Display` [`trait`](https://rustwiki.org/zh-CN/rust-by-example/trait.html)。  
`std::fmt`包含多种 `trait`来控制文字显示，其中重要的两种 trait 的基本形式如下：
-  `fmt::Debug`：使用 `{:?}` 标记。格式化文本以供调试使用。
-  `fmt::Display`：使用 `{}` 标记。以更优雅和友好的风格来格式化文本。

## `fmt::Debug`
> 所有的类型，若想用 `std::fmt` 的格式化打印，都要求实现至少一个可打印的 `traits`。仅有一些类型提供了自动实现，比如 `std` 库中的类型。所有其他类型都**必须手动实现**。  
> `fmt::Debug` 这个 `trait` 使这项工作变得相当简单。所有类型都能推导（`derive`，即自动创建）`fmt::Debug` 的实现。  
> 但是 `fmt::Display` 需要手动实现。

格式化输出自定义类型变量：
```rust
struct Number(i8);
println!("{}", Number(2)); // 报错
```
需要加上 `#[derive(Debug)]` 声明，并使用`{:?}`占位：
```rust
#[derive(Debug)]
struct Number(i8);
println!("{:?}", Number(2)); // Number(2)
```

**所有 `std` 库类型都天生可以使用 `{:?}` 来打印，但使用 `derive` 的一个问题是不能控制输出的形式。**  
同时 Rust 也通过 `{:#?}` 提供了 “美化打印” 的功能，或者可以通过手动实现 `fmt::Display` 来控制显示效果。

### `fmt::Display`
> `fmt::Display` 采用 `{}` 标记

其实现方式需要自己实现，比较灵活，下面是一个例子：
```rust
use std::fmt;

struct Strawberry(i32);

impl fmt::Display for Strawberry {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        return write!(f, "I have eat {} 🍓.", self.0);
    }
}

fn main() {
    println!("{}", Strawberry(12)); // I have eat 12 🍓.
}
```

实现复合结构体的自定义显示：
```rust
use std::fmt;

#[derive(Debug)]
struct Complex{
    real: f32,
    imag: f32
}

impl fmt::Display for Complex {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        return write!(f, "{} + {}i", self.real, self.imag);
    }
}

fn main() {
    let c = Complex{
        real: 3.3,
        imag: 7.2
    };
    println!("Display: {}", c); // Display: 3.3 + 7.2i
    println!("Display: {:?}", c); // Debug: Complex { real: 3.3, imag: 7.2 }
}
```

通过以上的例子，可以简单反应 `fmt::Display` 的基本使用方式。  
注意：对于一些**泛型容器**（generic container），`fmt::Display` 都没有实现。所以，在这些泛型的情况下要用 `fmt::Debug`。

对一个结构体实现 `fmt::Display`，其中的元素需要一个接一个地处理到，这可能会很麻烦。问题在于每个 `write!` 都要生成一个 `fmt::Result`。正确的实现需要处理所有的 `Result`。`Rust` 专门为解决这个问题提供了 `?` 操作符，使用方法如下：
```rust
use std::fmt;

struct List(Vec<i32>);

impl fmt::Display for List {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let vec = &self.0;

        write!(f, "[")?;

        for (index, value) in vec.iter().enumerate() {
            if index != 0 { 
                write!(f, ", ")?;
            }
            write!(f, "{}: {}", index, value)?;
        }

        return write!(f, "]");
    }
}

fn main() {
    let arr = List(vec![2, 7]);
    println!("{}", arr); // [0: 2, 1: 7]
}
```
结合 Display 的格式字符串，还可以做到更加丰富的自定义输出，如下面的RGB转16进制：
```rust
use std::fmt::{Formatter, Display, Result};

struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

impl Display for Color {
    fn fmt(&self, f: &mut Formatter) -> Result {
        return write!(
            f, 
            "RGB ({r}, {g}, {b}) 0x{r:0w$X}{g:0w$X}{b:0w$X}", // 0w$是补位0，X是转16进制
            r = self.red,
            g = self.green,
            b = self.blue,
            w = 2
        );
    }
}

fn main() {
    for color in [
        Color { red: 128, green: 255, blue: 90 },
        Color { red: 0, green: 3, blue: 254 },
        Color { red: 0, green: 0, blue: 0 },
    ].iter() {
        println!("{}", *color)
    }
    /* 
    RGB (128, 255, 90) 0x80FF5A
    RGB (0, 3, 254) 0x0003FE
    RGB (0, 0, 0) 0x000000
    */
}
```

## 类型
### 原生类型
Rust的原生类型分为标量类型（scalar type）和复合类型（compound type）。类似于Js的基本类型和引用类型。

标量类型分为几大类：
-   有符号整数（signed integers）：`i8`、`i16`、`i32`、`i64`、`i128` 和 `isize`（指针宽度）
-   无符号整数（unsigned integers）： `u8`、`u16`、`u32`、`u64`、`u128` 和 `usize`（指针宽度）
-   浮点数（floating point）： `f32`、`f64`
-   `char`（字符）：单个 Unicode 字符，如 `'a'`，`'α'` 和 `'∞'`（每个都是 4 字节）
-   `bool`（布尔型）：只能是 `true` 或 `false`
-   单元类型（unit type）：`()`。其唯一可能的值就是 `()` 这个空元组

> 尽管单元类型的值是个元组，但它并不包含多个值，被认为是标量类型。

复合类型主要有两种：
-   数组（array）：如 `[34, 56, 78]`
-   元组（tuple）：如 `(false, 89)`

变量都能够显式地给出**类型说明**，如果不指定的话，Rust会给定其默认类型：整型默认为 `i32` 类型，浮点型默认为 `f64`类型。注意 **Rust 还可以根据上下文来推断（infer）类型**。

```rust
fn main() {
     // 给出类型说明
    let isOpen: bool = true;
    let let num_float: f64 = 1.0;
    
    // 默认方式决定类型。
    let default_float   = 3.0; // `f64`
    let default_integer = 7;   // `i32`
    
    // 类型也可根据上下文自动推断。
    let mut inferred_type = 12; // 根据下一行的赋值推断为 i64 类型
    inferred_type = 4294967296i64;
}
```

可变的（mutable）变量，其值可以改变，但变量的类型并不能改变！  
但可以用遮蔽（shadow）来覆盖前面的变量。

```rust
fn main() {
    // 
    let mut mutable = 12; // Mutable `i32`
    mutable = 21;
    // 报错！
    mutable = true;
    // 但可以用遮蔽（shadow）来覆盖前面的变量。
    let mutable = true;
}
```

特殊地，数字还可以通过**后缀**（suffix）或**默认方式**来声明类型。
```rust
let an_integer = 5i32; // 后缀说明
```

### 字面量
整数、浮点数、字符、字符串、布尔值和单元类型可以用数字、文字或符号之类的 “字面量”（literal）来表示，如整数 `100` 、浮点数 `3.14`、字符`'1'`、字符串`"ab"`、布尔值`true`和单元类型`()`。

另外，通过加前缀 `0x`、`0o`、`0b`，数字可以用十六进制、八进制或二进制记法表示。   
为了改善可读性，可以在数值字面量中插入下划线，比如：`1_000` 等同于 `1000`，`0.000_001` 等同于 `0.000001`。

~感觉更像 JS 了🤣~

同时，Rust的变量运算符和其它语言（尤其是 JS 😂）都差不多，运算符的的优先级和类 C 语言类似。

### 元组
> 元组是一个可以包含各种类型值的组合。

元组使用括号 `()` 来构造，而每个元组自身又是一个类型标记为 `(T1, T2, ...)` 的值，其中 `T1`、`T2` 是每个元素的类型。

函数可以使用元组来返回多个值，因为元组可以拥有任意多个值（元组也可以嵌套元组）。

注意，元组使用 `.[下标位置]` 来访问每个具体的值：
```rust
let tuple_a = (2,3,4,'a');
println!("tuple_a的第二个值为：{}", tuple_a.1); // tuple_a的第二个值为：3
```
**注意：** 创建单元素元组需要一个额外的逗号，用于和括号包含的字面量作区分: `(1i32, )`

### 类型解构
类似于 JS 的类型解构，Rust里的元组也可以被解构（deconstruct），将值解构给其它不同的变量。
```rust
let tuple = (2u32, "hello", true);
let (a, b, c) = tuple;
```
### 数组
> 数组（array）是一组拥有**相同类型**的对象的集合，在内存中是连续存储的。数组使用中括号 `[]` 来创建，且它们的大小在编译时会被确定。数组的类型标记为 `[T; length]`

切片（slice）和数组类似，但其大小在编译时是不确定的。

> 切片是一个双字对象（two-word object），第一个字是一个指向数据的指针，第二个字是切片的长度。这个 “字” 的宽度和 usize 相同，由处理器架构决定，比如在 x86-64 平台上就是 64 位。  
> slice 可以用来借用数组的一部分。  
> slice 的类型标记为 &[T]。  

数组和切片使用方式举例：
```rust
fn main() {
    // 数组
    let arr = [1, 2, 3, 4, 5];

    // len() 获取数组长度
    println!("数组第一个值为：{}，数组的长度为：{}", arr[0], arr.len()); // 数组第一个值为：1，数组的长度为：5

    // slice 可以指向数组
    println!("{:?}", &arr[1 .. 3]); // [2, 3]
}
```

### 自定义类型
Rust 自定义数据类型主要是通过下面这两个关键字来创建：

1. `struct`： 定义一个结构体（structure）
2. `enum`： 定义一个枚举类型（enumeration）

### struct 结构体
> 结构体中有 3 种类型，使用 `struct` 关键字来创建：
> 
> 1. 元组结构体（tuple struct），事实上就是具名元组而已
> 2. C 语言风格结构体（C struct）
> 3. 单元结构体（unit struct），不带字段，在泛型中很有用
> 
> ```rust
> // 单元结构体
> struct Unit;
> 
> // 元组结构体
> struct Pair(i32, f32);
> 
> // 带有两个字段的结构体
> struct Point {
>     x: f32,
>     y: f32,
> }
> ```

结构体可以将若干个类型不一定相同的数据捆绑在一起形成整体，每个成员和其本身都有一个名字，这样访问它成员的时候就不用记住下标了，可以用于规范常用的数据结构。例如下面实现矩形，并计算其面积：
```rust
struct Point {
    x: f32,
    y: f32,
}

#[allow(dead_code)]
struct Rectangle {
    // 给定左上角和右下角在空间中的位置来指定矩形
    top_left: Point,
    bottom_right: Point,
}

fn rect_area(rect: Rectangle) -> f32 {
    let Rectangle {
        top_left: tl,
        bottom_right: br
    } = rect;
    let Point { x: x1, y: y1} = tl;
    let Point { x: x2, y: y2} = br;
    let width = x2 - x1;
    let height = y2 - y1;
    let area = width * height;

    if area > 0f32 { // Rust 没有提供三元表达式
        area // return 可以不用写
    } else {
        -area
    }
}

fn main() {
    let point: Point = Point { x: 2.0, y: 1.0 };
    // let bottom_right = Point { x: 5.2, ..point }; // { x: 5.2, y: 0.4 }

    let _rectangle = Rectangle {
        // 结构体的实例化也是一个表达式
        top_left: Point { x: 5.0, y: 6.0 },
        bottom_right: point,
    };
    let res = rect_area(_rectangle);
    println!("矩形的面积为：{}", res); // 矩形的面积为：矩形的面积为：15
}
```
### enum 枚举
> `enum` 关键字允许创建一个从数个不同取值中选其一的枚举类型（enumeration）。任何一个在 `struct` 中合法的取值在 `enum` 中也合法。

```rust
enum WebEvent {
    PageLoad,
    KeyPress(char),
    Click { x: i64, y: i64 }
}

fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("page loaded"),
        WebEvent::KeyPress(c) => println!("pressed '{}'.", c),
        WebEvent::Click { x, y } => {
            println!("clicked at x={}, y={}.", x, y);
        },
    }
}

fn main() {
    let pressed = WebEvent::KeyPress('x');
    let click   = WebEvent::Click { x: 20, y: 80 };
    let load    = WebEvent::PageLoad;

    inspect(pressed); // pressed 'x'.
    inspect(click); // clicked at x=20, y=80.
    inspect(load); // page loaded
}
```

还可以创建类型别名：

```rust
type event = WebEvent;
```

最常见的情况就是在 `impl` 块中使用 `self` 别名。

```rust
use std::fmt;

struct Complex {
    real: f32,
    imag: f32
}

impl fmt::Display for Complex {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        return write!(f, "{} + {}i", self.real, self.imag);
    }
}
```
Rust可以隐式辨别值：
```rust
// 隐式辨别值（从 0 开始）的 enum
enum Number {
    Zero, // 0
    One, // 1
    Two, // 2
}
// 显式辨别值
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}
```

enum 的一个常见用法就是创建链表（linked-list）：[测试实例：链表](https://rustwiki.org/zh-CN/rust-by-example/custom_types/enum/testcase_linked_list.html)

### 常量
Rust 有两种常量，可以在任意作用域声明，包括全局作用域。它们都需要显式的类型声明：

- `const`：不可改变的值（通常使用这种）。
- `static`：具有 `'static` 生命周期的，可以是可变的变量（`static mut`）。


## 一些技巧
1. `#[allow(dead_code)]` 用于隐藏对未使用代码的警告。
    - `#[allow(dead_code)]`
    - `#![allow(dead_code)]` 用于本文件范围
2. 使用 `use` 声明可以导入特定的模块，就不需要再写完整的路径了，类似于 lua 中的 `local`。


## 参考
- [std::fmt - Rust (rustwiki.org)](https://rustwiki.org/zh-CN/std/fmt/)


