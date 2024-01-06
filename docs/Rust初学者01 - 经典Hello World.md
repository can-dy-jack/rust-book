> 开坑：Rust学习。  
> 前言：本人是一位刚入门的前端开发，技术实力本就有限，加上近9个月由于基本将大部分的精力都放在工作上了，剩余的一点时间也只想躺着休息，也没有之前入门时的技术热情了，导致自身技术基本以来工作被动去提升。    
> 今天是元旦，2024年的第一天，今年的目标就是提升自己的实力（前端和后端）。后端暂定为学习Rust，并记录学习过程，有所不足请大佬指教。

## Rust环境安装
安装方法可去官网查看官方的教程： [安装 Rust - Rust 程序设计语言 (rust-lang.org)](https://www.rust-lang.org/zh-CN/tools/install)；

但大概率会安装失败：

```bash
error: could not download file from 'https://static.rust-lang.org/dist/channel-rust-stable.toml.sha256'
```

这里推荐去官网下载msi文件手动下载：[Other Installation Methods - Rust Forge (rust-lang.org)](https://forge.rust-lang.org/infra/other-installation-methods.html)


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/672bdd23bccb4ae2927434cd90fd1b45~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1980&h=1239&s=242057&e=png&b=fcfcfc)

选择对应的版本下载安装即可,这里我选择windows的版本。


安装后可检测是否安装成功：
```bash
rustc -- version
cargo --version
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c3db605ef0a43f18df55a44a41eb4a6~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=715&h=241&s=23952&e=png&b=0d0d0d)


## Hello World
> 安装好环境之后，即可着手编写rust程序，我们第一个程序自然是`Hello World` 。

新建 `index.rs` 文件，写入我们的第一个 `rust` 代码：
```rust
// 主函数
fn main() {
    // 将文本打印到控制台
    println!("Hello World!");
}
```
接下来就可以编译运行，使用 `rustc` 可将 `rs` 文件编译成可执行文件：
```bash
// 编译
rustc index.js
```
运行后会，`rs`同级目录下会生成 `.exe` 文件，运行该文件即可看到输出`Hello World`。
```bash
// 运行 index.exe
./index
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a39aa2d3d2c84c919e90833121c385bb~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=846&h=192&s=30901&e=png&b=181818)

看到输出内容则证明你已经成功运行了 Rust 项目！~~恭喜你已经成功掌握了Rust！🤣~~
### cargo
类似于前端，程序不仅可以直接用 node.js 运行单个 js 文件，也可以使用包管理工具搭建项目来运行程序。类似于 `npm` ， `cargo` 是 rust 的管理工具。使用 `cargo` 可快速创建项目：
```bash
cargo new [name]
```
新建项目后，目录下会出现几个文件：
```bash
. 
├── Cargo.toml 
└── src 
    └── main.rs
```
简单的 Cargo 项目有以下文件：
- `src/main.rs` rust程序入口
- `Cargo.toml` Rust 项目的配置文件，类似于 `package.json` 。
- `Cargo.lock` 依赖文件。类似于 `npm` 的 `package-lock.json`,在安装依赖后会生成。 

cargo创建的项目可以使用cargo命令运行：`cargo run` 。

相信有过前端开发经验的开发的同学应该会感觉到很熟悉，Rust就像是为前端开发的后端语言！

## 参考资料
- [Rust 程序设计语言 (rust-lang.org)](https://www.rust-lang.org/zh-CN/)
- [Overview - Rust Forge (rust-lang.org)](https://forge.rust-lang.org/index.html)
