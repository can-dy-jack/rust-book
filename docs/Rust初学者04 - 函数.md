## 函数
> 函数（function）使用 `fn` 关键字来声明。函数的参数需要标注类型，就和变量一样，如果函数返回一个值，返回类型必须在箭头 `->` 之后指定。

函数最后的表达式将作为返回值。也可以在函数内使用 return 语句来提前返一个值。

以经典的[两数之和](https://leetcode.cn/problems/two-sum/)为例（这里不考虑使用哈希表方式）：
```rust
// 指定参数类型、返回值类型
pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {
    // 循环数组
    for i in 0..nums.len() {
        for j in i+1..nums.len() {
            if (nums[i] + nums[j] == target) {
                // 可以使用 return 提前返回
                return vec![i as i32, j as i32];
            }
        }
    }
    // 走到结尾了也没匹配上（虽然题目保证一定有匹配值），需要保证返回值的类型
    vec![0, 0]
}
```

## 方法
> 方法（method）是依附于对象的函数。这些方法通过关键字 `self` 来访问对象中的数据和其他。方法在 `impl` 代码块中定义。

方法即是放在对象里的函数，对象里的函数就是方法，方法使用 `.` 访问，下面是一个简单的使用示例：

```rust
#![allow(dead_code)]

struct Flower {
    color: String,
    smell: String,
}

impl Flower {
    fn new(color: &str, smell: &str) -> Flower { // 构造方法
        Flower { color: color.to_string(), smell: smell.to_string() }
    }
    // `&self` 是 `self: &Self` 的语法糖（sugar），self指向自己。
    fn say(&self) { // 方法
        println!(
            "{}",
            format!("I love {} Rose, it smell {}.", self.color, self.smell)
        );
    }
}

fn main() {
    let rose = Flower::new("red", "good"); // 使用构造函数

    rose.say(); // 调用 say 方法
}
```

## 闭包
Rust 中，函数和方法使用方式和其它语言类似，基本没什么特殊的地方。但是Rust中有一种特殊的函数，叫做闭包，如下一个将变量加一的闭包：
```rust
fn main() {
    let num = 1;

    // 闭包 等同于 fn add_one(i: i32) -> i32 { i + 1 }
    let add_one = |i| i + 1;  // |i| { i + 1 } 括号可以省略

    println!("{}", add_one(num)); // 2
}
```

> Rust 中的闭包（`closure`），也叫做 `lambda` 表达式或者 `lambda`，是一类能够捕获周围作用域中变量的函数。

闭包于普通函数而言，有几个特殊的地方：
1. 语法简单，只需要指定变量名，而输入和返回类型两者都可以自动推导（可以主动指定类型）。
2. 写法简单，声明时使用 || 替代 () 将输入参数括起来。
3. 可省略大括号，大括号在只有单个表达式时是可选的。
4. 捕获变量，有能力捕获外部环境的变量。

### 捕获变量
在普通函数里，传入的参数类型需要指定类型，如果只使用引用需要指定引用类型，而闭包可以灵活地判断使用引用还是变量本身。

1. 通过引用 `&T` 捕获变量

```rust
fn main() {
    let letter = "A";

    let print = || println!("letter is {}", letter);
    print();

    // `color` 可再次被不可变借用，因为闭包只持有一个指向 `color` 的不可变引用。
    let _reborrow = &letter;
    print();

    // 在最后使用 `print` 之后，移动或重新借用都是允许的。
    let _color_moved = letter;
}
```

2. 通过可变引用`&mut T` 捕获变量

```rust
fn main() {
    let mut count = 0;
    let mut add_one = || {
        count += 1;
        println!("`count`: {}", count);
    };
    
    // 使用可变借用调用闭包
    add_one();
    
    // 因为之后调用闭包，所以仍然可变借用 `count`
    // let _reborrow = &count; 试图重新借用将导致错误 
    add_one();
    
    // 闭包不再借用 `&mut count`，因此可以正确地重新借用
    let _count_reborrowed = &mut count;
}
```

3. 通过值 `T` 捕获变量（拿到了变量的所有权而非借用）
```rust
use std::mem;

fn main() {
    // 内置 Box 类型将数据存储在堆上
    let movable = Box::new(3);

    // `mem::drop` 释放资源
    let consume = || {
        println!("`movable`: {:?}", movable);
        mem::drop(movable);
    };

    // `consume` 消耗了该变量，所以该闭包只能调用一次。
    consume();
}
```

> **闭包优先通过引用来捕获变量，并且仅在需要时使用其他方式。**  
> 闭包本质上很灵活，能做功能要求的事情，使闭包在没有类型标注的情况下运行。这使得捕获（capture）能够灵活地适应用例，既可移动（move），又可借用（borrow）。

在竖线 | 之前使用 `move` 关键词会强制闭包取得被捕获变量的所有权：
```rust
fn main() {
    let nums = vec![1, 2, 3];

    let contains = move |num| nums.contains(num);

    println!("{}", contains(&1));
    println!("{}", contains(&4));

    // 报错， 借用检查不允许在变量被移动走之后继续使用它。
    println!("There're {} elements in vec", nums.len());
}
```

### 将闭包作为函数参数
可以将闭包作为函数的参数，需要使用 `where` 关键词指定闭包的类型。一共有三种闭包类型：

- `Fn`：表示捕获方式为通过引用（&T）的闭包。
- `FnMut`：表示捕获方式为通过可变引用（&mut T）的闭包。
- `FnOnce`：表示捕获方式为通过值（T）的闭包。

```rust
// 使用无参数和返回值的闭包作为 输入参数
fn read<F>(f: F) where F: FnOnce() {
    f();
}

// 有参数和返回值的闭包作为 输入参数
fn write<F>(f: F) -> i32 where F: Fn(i32) -> i32 {
    f(3)
}
```

**注意**，指定输入参数中闭包的类型时，闭包可能采取 &T，&mut T 或 T 中的一种捕获方式，编译器最终是根据所捕获变量在闭包里的使用情况决定捕获方式。所以，需要给到足够多的“权限”，即闭包捕获的是引用类型可以指定`Fn`，`FnMut` 或 `FnOnce`，但是如果捕获的是值，则需要指定 `FnOnce`。

```rust
fn read<F>(f: F) where F: Fn() {
    f();
}

fn main() {
    let name = "read";

    let diary = || {
        println!("{}.", name);
    };

    read(diary);
}
```

> 扩展：既然闭包可以作为函数的输入参数，那么普通函数也是可以的：
```rust
fn run_fn<F : Fn()>(f: F) { // 也是 Fn FnMut fnOnce 三种泛型类型
    f();
}

fn print_hello() {
    println!("Hello!");
}

fn main() {
    run_fn(print_hello);
}
```

### 将闭包作为返回值
> 目前 Rust 只支持返回具体（非泛型）的类型。按照定义，匿名的闭包的类型是未知的，所以只有使用impl Trait才能返回一个闭包。

除此之外，还必须使用 `move` 关键字，它表明所有的捕获都是通过值进行的。这是必须的，因为在函数退出时，任何通过引用的捕获都被丢弃，在闭包中留下无效的引用。

```rust
fn create_fn() -> impl Fn() {
    let text = "Fn".to_owned();

    move || println!("This is a: {}", text)
}

fn create_fnmut() -> impl FnMut() {
    let text = "FnMut".to_owned();

    move || println!("This is a: {}", text)
}

fn create_fnonce() -> impl FnOnce() {
    let text = "FnOnce".to_owned();
    
    move || println!("This is a: {}", text)
}

fn main() {
    let fn_plain = create_fn();
    let mut fn_mut = create_fnmut();
    let fn_once = create_fnonce();

    fn_plain();
    fn_mut();
    fn_once();
}
```

## 高阶函数
> 高阶函数（Higher Order Function, HOF），指那些输入一个或多个函数，并且/或者产生一个更有用的函数的函数。

Rust 同时支持链式调用，结合高阶函数，可实现函数式的代码：
```rust
fn is_odd(n: u32) -> bool {
    n % 2 == 1
}

fn main() {
    let upper = 1000;

    let sum_of_squared_odd_numbers: u32 =
        (0..).map(|n| n * n)             // 所有自然数取平方
             .take_while(|&n| n < upper) // 取小于上限的
             .filter(|&n| is_odd(n))     // 取奇数
             .fold(0, |sum, i| sum + i); // 最后加起来
}
```

## 发散函数
> 发散函数（diverging function）无返回值。 使用 `!` 标记，这是一个空类型。

> The ! type, also called “never”.   
> *This is a nightly-only experimental API.*

和所有其他类型相反，这个类型无法实例化，因为此类型可能具有的所有可能值的集合为空。 它与 `()` 类型不同，后者只有一个可能的值。

虽然这看起来像是一个抽象的概念，但实际上这非常有用且方便。这种类型的主要优点是它可以被转换为任何其他类型，从而可以在需要精确类型的地方使用，例如在 match 匹配分支。 这也是永远循环（如 loop {}）的函数（如网络服务器）或终止进程的函数（如 exit()）的返回类型。

## Tips
- 方法里，`&self` 是 `self: &Self` 的语法糖，`self` 指向自己，类似于 JS 中的 `this` 关键词。

## 参考资料
- [https://rustwiki.org/zh-CN/rust-by-example/fn.html](https://rustwiki.org/zh-CN/rust-by-example/fn.html)
- [https://doc.rust-lang.org/std/primitive.never.html#](https://doc.rust-lang.org/std/primitive.never.html#)
