## 作用域
Rust 中使用 `let` 来声明变量，可以在声明时说明类型，也可以不声明，让编译器从上下文推导出变量的类型。

变量绑定有一个作用域（scope），它被限定只在一个代码块（`{}` 包围的代码）中生效，变量只能在其作用域内访问，。

同时， Rust 允许变量 `遮蔽`，在内部作用域里可以覆盖外面的变量。

## 类型
> Rust 提供了多种机制，用于改变或定义原生类型和用户定义类型。

Rust 不提供原生类型之间的隐式类型转换（coercion），但可以使用 as 关键字进行显式类型转换（casting）。

```rust
fn main() {
    let num = 65.4321;

    // let integer: u8 = decimal; // 报错

    let int = num as u8;

    println!("int: {}", int); // int: 65
}
```

### 别名
可以用 `type` 关键词给已有的类型取个别名。但名字必须遵循驼峰命名法（CamelCase）。
```rust
type Integer32 = u32;

fn main() {
    let num: Integer32 = 65;

    println!("u32: {}", num);
}
```
### 类型转换
Rust 中会使用 `trait` 解决类型之间的转换问题。一般的转换会用到 `From` 和 `Into` 两个 `trait`。但是，涉及 String 的转换会比较特别。
#### `From` 
`From` 这个`trait`就是用来从其它类型转换过来。  
在标准库中有无数 `From` 的实现，规定原生类型及其他常见类型的转换功能。可以直接调用 `from` 来进行转换：
```rust
type Integer32 = u32;

fn main() {
    let str = "Rust";

    let string: String = String::from(str); // From

    println!("{}", string);
}
```
还可以自定义 `From` 转换逻辑：
```rust
use std::convert::From;

#[derive(Debug)]
struct Score {
    val: u32
}

impl From<u32> for Score {
    fn from(val: u32) -> Self{
        Score { val: val }
    }
}

fn main() {
    let math = Score::from(99);
    println!("math: {:?}, score: {}", math, math.val);
}
```

#### `Into`
同 `From` 相反，`Into` trait 就是转换成其它类型。令人兴奋的是，如果你为你的类型实现了 `From`，那么同时你也就实现了 `Into`。就如上例，我们实现了 `Score` 的 `From` ，那么我们就可以直接使用 `into`。

```rust
use std::convert::From;

#[derive(Debug)]
struct Score {
    val: u32
}

impl From<u32> for Score {
    fn from(val: u32) -> Self{
        Score { val:val }
    }
}

fn main() {
    let math_score = 99;
    // 必须指定 Score 类型
    let math: Score = math_score.into();
    println!("math: {:?}, score: {}", math, math.val);
}
=
```

#### `TryFrom and TryInto`
> 类似于 From 和 Into，TryFrom 和 TryInto 是类型转换的通用 trait。不同于 From/Into 的是，TryFrom 和 TryInto trait 用于易出错的转换，也正因如此，其返回值是 Result 型。

以下是使用示例（其中， `assert!(matches!(a, b))` 是断言语句，类似于 node 中的断言）

```rust
use std::convert::TryFrom;
use std::convert::TryInto;

#[derive(Debug)]
struct PerfectScore(u32);

impl TryFrom<u32> for PerfectScore {
    type Error = ();

    fn try_from(val: u32) -> Result<Self, Self::Error>{
        if val >= 90 && val < 100 {
            Ok(PerfectScore(val))
        } else {
            Err(())
        }
    }
}

fn main() {
    assert!(matches!(PerfectScore::try_from(98), Ok(PerfectScore(98))));
    assert!(matches!(PerfectScore::try_from(87), Ok(PerfectScore(87)))); // assertion failed

    let result: Result<PerfectScore, ()> = 98u32.try_into();
    assert!(matches!(result, Ok(PerfectScore(98))));
}
```

#### `ToString 和 FromStr`
> 要把任何类型转换成 String，只需要实现那个类型的 ToString trait。但是，通常可以实现fmt::Display trait，它会自动提供 ToString。

可以直接实现该类型的 `ToString`：
```rust
use std::string::ToString;

#[derive(Debug)]
struct PerfectScore(u32);

impl ToString for PerfectScore {
    fn to_string(&self) -> String{
        if self.0 >= 90 {
            format!("perfect")
        } else {
            format!("not good")
        }
    }
}

fn main() {
    let math = PerfectScore(98);

    println!("math: {:?}, to_string: {}", math, math.to_string()); // math: PerfectScore(98), to_string: perfect
}
```

或者实现其 `Display`：
```rust
use std::fmt;

#[derive(Debug)]
struct PerfectScore(u32);

impl fmt::Display for PerfectScore {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result{
        if self.0 >= 90 {
            write!(f, "perfect")
        } else {
            write!(f, "not good")
        }
    }
}

fn main() {
    let math = PerfectScore(98);

    // 打印类型：PerfectScore(98), Display: perfect, to_string: perfect
    println!("打印类型：{:?}, Display: {}, to_string: {}", math, math, math.to_string()); 
}
```

实现 `Display` 可以顺带实现 `ToString`。而实现 `ToString`并不会实现 `Display`，即打印时需要使用 `{:?}`，否则会报错，需要调用 `to_string`。所以，推荐直接实现 `Display` 。

如果要将 `String` 转成其它类型，只要对目标类型实现了 `FromStr` trait，就可以用 `parse` 把字符串转换成目标类型。 标准库中已经给无数种类型实现了 `FromStr`。如果要转换到用户定义类型，只要手动实现 `FromStr` 就行。
```rust
fn main() {
    let number: Result<i32, _> = "5".parse(); // parse 得到的是 Result 类型（有可能转换失败）
    println!{"{:?}", number.unwrap()}; // 5

    let number2 = "10".parse::<u32>().unwrap(); // 指定类型
    println!{"{}", number2}; // 10
}
```

## 表达式
> Rust 有多种语句。最普遍的语句类型有两种：一种是声明绑定变量，另一种是表达式带上英文分号(;)：
```rust
fn main() {
    // 变量绑定
    let x = 2;

    // 表达式;
    x * 5;
    ();
}
```
代码块也是表达式，所以它们可以用作赋值中的值。代码块中的最后一个表达式将赋给适当的表达式，不需要 return。但是，如果代码块的最后一个表达式结尾处有分号，则返回值为 ()
```rust
fn main() {
    let a = {
        // 未加分号，将4赋给 a
        4
    };

    let b = {
        // 加了分号，将 () 赋给 b
        5;
    };
    println!("{:?}, {:?}", a, b); // 4, ()
}
```

## 流程控制
### `if/else`
if-else 分支判断和其他语言基本一致。不同的是，Rust 中的判断条件不加小括号，而大括号是不可省略的（与js相反）。注意，所有分支都必须返回相同的类型。
```rust
if n == 0 {
    print!("n 是 0");
} else if n > 0 {
    print!("n大于0");
} else {
    print!("n小于0");
}
```
特殊的，rust 中 if-else 语句可以用于在赋值语句中（注意要在if-else语句结尾处加分号，表明语句结束）：
```rust
fn main() {
    let n = 2;
    let res = if n == 0 {
        0
    } else if n > 0 {
        1
    } else {
        -1
    };
    println!("{:?}", res); // 1
}
```

### `loop`
> Rust 提供了 loop 关键字来表示一个无限循环，等同于 while(true)。可以使用 break 语句在任何时候退出一个循环，还可以使用 continue 跳过循环体的剩余部分并开始下一轮循环。
```rust
fn main() {
    let mut count = 0u32;
    loop {
        count += 1;
        if count % 2 == 1 {
            continue;
        }
        if count > 5 {
            break;
        }
        println!("{}", count);
    }
}
```
同时，在多层 loop 循环里，可以用一些 label（标签）来注明，以便于 break 或 continue 来作用于指定的 loop。
```rust
fn main() {
    'out: loop {
        loop {
            break 'out;
        }
    }
}
```
在Rust中，loop也可以用于在赋值语句中,=，将值放在 break 之后，它就会被 loop 表达式返回：
```rust
fn main() {
    let mut n  = 2;
    let res = loop {
        n += 1;
        if n == 10 {
            break n*2
        }
    };
    println!("{:?}", res); // 1
}
```

### `while`
`while`和`if/else`一样，判断条件不加小括号，而大括号不可省略。
```rust
fn main() {
    let mut n = 1;
    while n < 101 {
        if n % 15 == 0 {
            println!("fizzbuzz");
        } else if n % 3 == 0 {
            println!("fizz");
        } else if n % 5 == 0 {
            println!("buzz");
        } else {
            println!("{}", n);
        }
        n += 1;
    }
}
```

### `for`
> `for in` 结构可以遍历一个 `Iterator`（迭代器）。创建迭代器的一个最简单的方法是使用**区间标记** a..b。这会生成从 a（包含此值） 到 b（不含此值）的(可以使用a..=b表示两端都包含在内的范围。)，步长为 1 的一系列值。
```rust
fn main() {
    for n in 1..101 {
        if n % 15 == 0 {
            println!("fizzbuzz");
        } else if n % 3 == 0 {
            println!("fizz");
        } else if n % 5 == 0 {
            println!("buzz");
        } else {
            println!("{}", n);
        }
    }
}
```
> 如果没有特别指定，for 循环会对给出的集合应用 `into_iter` 函数，把它转换成一个迭代器。这并不是把集合变成迭代器的唯一方法，其他的方法有 `iter` 和`iter_mut` 函数。

这三个函数会以不同的方式迭代集合中的数据。
- `iter` - 在每次迭代中借用集合中的一个元素。这样集合本身不会被改变，循环之后仍可以使用。拿到的每一个元素是引用。
```rust
fn main() {
    let words = vec!["a", "b", "c", "d"];

    for word in words.iter() {
        match word {
            &"a" => print!("It's begin： a "),
            _ => print!("{} ", word),
        }
    }
}
```

- `into_iter` - 会消耗集合。在每次迭代中，集合中的数据本身会被提供，即将集合变成了一次性数据。因此拿到的每一个元素都是其本身，而不是引用。
```rust
fn main() {
    let words = vec!["a", "b", "c", "d"];

    for word in words.into_iter() {
        match word {
            "a" => print!("It's begin： a "),
            _ => print!("{} ", word),
        }
    }
    // println!("words: {:?} ", words); // value borrowed here after move
}
```

- `iter_mut` - 可变地（mutably）借用集合中的每个元素，允许集合被就地修改，所以需要返回修改过的值。
```rust
fn main() {
    let mut words = vec!["a", "b", "c", "d"];

    for word in words.iter_mut() {
        *word = match word {
            &mut "a" => "It's begin： a ",
            _ => " ",
        };
    }
}
```

### `match`
Rust 通过 `match` 关键字来提供模式匹配，和其它语言的 `switch` 用法类似。**第一个**匹配分支会被比对，并且**所有可能的值都必须被覆盖**。
```rust
fn main() {
    let number = 13;

    match number {
        // 匹配单个值
        1 => println!("One!"),
        // 匹配多个值
        2 | 3 | 5 | 7 | 11 => println!("This is a prime"),
        // 匹配一个闭区间范围
        13..=19 => println!("A teen"),
        // 处理其他情况
        _ => println!("Ain't special")
    }

    let boolean = true;
    // match 也是一个表达式
    let binary = match boolean {
        // match 分支必须覆盖所有可能的值
        false => 0,
        true => 1,
    };
}
```
`match` 不仅可以用来匹配，还可用来解构类型（本质上也是匹配所有可能的值）。
1. 解构元组
```rust
fn main() {
    let triple = (0, 1, 3);
    match triple {
        (0, y, z) => (), // 匹配第一个是0的情况，并将第二个值赋给y，第三个值赋给z
        (1, ..)  => (), // 匹配第一个是1的情况
        _  => (),
    }
}
```
2. 结构枚举类型
```rust
enum Language {
    C,
    Rust,
    JavaScript,
    CSS(u32),
    HTML(u32)
}

fn main() {
    let color = Language::CSS(3);

    match color {
        Language::C             => println!("C Language"),
        Language::Rust          => println!("Rust Language"),
        Language::JavaScript    => println!("JavaScript Language"),
        Language::CSS(version)  => println!("CSS{}", version),
        Language::HTML(version) => println!("HTML{}", version),
    }
}
```
3. 指针和引用
对指针来说，解构（destructure）和解引用（dereference）要区分开。
- 解引用使用 `*`
```rust
// 获得一个 `i32` 类型的引用。`&` 表示取引用。
let reference = &4;

match reference {
    &val => println!("Got a value via destructuring: {:?}", val),
}

// 如果不想用 `&`，需要在匹配前 解引用
match *reference {
    val => println!("Got a value via dereferencing: {:?}", val),
}
```
- 解构使用 `&`、`ref`、和 `ref mut`
```rust
fn main() {
    // 不用引用
    let _not_a_reference = 3;
    // 使用 `ref` 更改赋值行为，从而可以对具体值创建引用
    let ref _is_a_reference = 3;


    let value = 5;
    let mut mut_value = 6;
    match value {
        ref r => println!("Got a reference to a value: {:?}", r),
    }
    match mut_value {
        ref mut m => {
            // 已经获得了 `mut_value` 的引用，先要解引用，才能改变它的值。
            *m += 10;
        },
    }
}
```

4. 解构 struct
类似于 js 解构 Object，Rust可以解构结构体的成员，结构的成员顺序不重要，但名字一定要一样：
```rust
struct Foo { x: (u32, u32), y: u32 }

fn main() {
    let foo = Foo { x: (1, 2), y: 3 };
    let Foo { x: (a, b), y } = foo;

    println!("a = {}, b = {},  y = {} ", a, b, y);
}
```
也可以忽略某些变量：
```rust
let Foo { y, .. } = foo;
println!("y = {}", y);
```

#### 卫语句
match 匹配时，可以加上一些条件(卫语句)来过滤匹配：
```rust
fn main() {
    let pair = (1, 2);
    match pair {
        (x, y) if x == y => println!("These are twins"),
        (x, y) if x + y == 0 => println!("Antimatter, kaboom!"),
        (x, _) if x % 2 == 1 => println!("The first one is odd"),
        _ => println!("No correlation..."),
    }
}
```
#### 绑定
在 `match` 中，若间接地访问一个变量，则不经过重新绑定就无法在分支中再使用它。`match` 提供了 `@` 符号来绑定变量到名称。
```rust
fn age() -> u32 {
    15
}

fn main() {
    match age() { // n用来绑定age函数的返回值
        0             => println!("I haven't celebrated my first birthday yet"),
        n @ 1  ..= 12 => println!("I'm a child of age {:?}", n),
        n @ 13 ..= 19 => println!("I'm a teen of age {:?}", n),
        n             => println!("I'm an old person of age {:?}", n),
    }
}
```
还可以用于枚举类型：
```rust
fn some_number() -> Option<u32> {
    Some(5)
}

fn main() {
    match some_number() {
        Some(3) => println!("The Answer: 3!"),
        Some(n @ 5..=12)      => println!("It's {} !", n),
        _            => (),
    }
}
```

### `if let`
了解了这么多Rust中 `match` 的用法，你可能一开始会不解，后面会发现其类似于其它语言的 `switch` 语句，到最后会觉得在某些情况下有点多此一举的感觉。  
`if let` 就是用来优化 `match` 的语法糖，它更接近于人类的思维，类似于 “如果这两个值是一样的，就执行这段代码”，可以省去要匹配所有可能的值的代码。而且 `if let` 可以匹配任何枚举值，即使未注明 `#[derive(PartialEq)]`，也没有为其实现 `PartialEq`，也可以使用！
```rust
fn main() {
    let number = Some(7);
    let letter: Option<i32> = None;

    if let Some(i) = number { // 如果 number 是 Some 类型的变量
        println!("Matched {:?}!", i);
    }

    // 可以在else里指定失败情形
    if let Some(i) = letter {
        println!("Matched {:?}!", i);
    } else {
        println!("Didn't match a number. ");
    };
}
```

**注意**：`if let Some(i) = letter` 不要写成 `if Some(i) == letter`

### `while let`
和 `if let` 类似，`while let` 也可以把别扭的 `match` 改写得好看一些。考虑下面这段使 `i` 不断增加的代码：
```rust
fn main() {
    let mut optional = Some(0);

    loop {
        match optional {
            Some(i) => {
                if i > 9 {
                    println!("Greater than 9, quit!");
                    optional = None;
                } else {
                    println!("`i` is `{:?}`. Try again.", i);
                    optional = Some(i + 1);
                }
            },
            // 当解构失败时退出循环：
            _ => { break; }
        }
    }
}
```
首先使用 `if let` 可以将 `match` 优化一些,看起来更方便理解一点：
```rust
fn main() {
    let mut optional = Some(0);

    loop {
        if let Some(i) = optional {
            if i > 9 {
                println!("Greater than 9, quit!");
                optional = None;
            } else {
                println!("`i` is `{:?}`. Try again.", i);
                optional = Some(i + 1);
            }
        } else {
            break;
        }
    }
}
```
而使用 `while let` 可以将循环加进去，进一步优化写法：
```rust
fn main() {
    let mut optional = Some(0);

    while let Some(i) = optional {
        if i > 9 {
            println!("Greater than 9, quit!");
            optional = None;
        } else {
            println!("`i` is `{:?}`. Try again.", i);
            optional = Some(i + 1);
        }
    }
}
```

## 一些小细节

- `..` 可用来忽略匹配中的其余部分，可用来占位。
- `_` 表示不将值绑定到变量，可用来占位。
