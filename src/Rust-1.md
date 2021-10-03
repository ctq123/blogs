# Rust编程系列1-入门介绍

> 主要介绍Rust系列的文章，本文主要介绍Rust的优劣势分析，安装，基本概念，包管理，编写猜谜游戏

## 一、背景
Rust是一门注重编译型、静态型的编程语言，它由Mozilla主导，它具有以下几个优点：安全、稳定、快速

> 1）安全：在编译期间，编译器会拒绝使用各种潜在的错误（包括并发的错误）来编译代码，保证运行时的错误尽可能少，语言机制上提供各种规范来约束空值和错误处理，以提高安全性
> 2）稳定：Rust编译器的检查通过添加和重构来确保稳定性
> 3）快速：它独特的编译时内存管理，没有令人头疼的指针，也没有run-time的GC造成的低效，它努力使安全代码成为快速代码，因此它非常快速

我们学习一门语言，往往只会看到它如何如何优秀，吹得天花乱坠，仿佛一门语言就能解决所有问题，然而事实并非如此，Rust也有它的缺点：灵活度受限、上手难

> 1）灵活度受限：为了做到编译时期捕获潜在bug，给予程序的灵活度有限，制定很多规范，有时会令人束手束脚
> 2）上手难：对传统语言不了解不深的童鞋往往很难理解rust的某些地方为什么要设计成这样，会显得特别隐晦

## 二、介绍
### 1）安装
不同的系统会有不同的安装方式，这里只介绍Linux和macOS的安装方式，更多[Rust官网](https://doc.rust-lang.org/book/ch01-01-installation.html)，开始安装rustup工具

```
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

选择默认版本并回车，成功后会提示

> Rust is installed now. Great!

### 2）编写hello world
创建一个名为rust-demo的工程，如下

![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1606745658522-cc365c53-059c-4380-8bbd-a677b5206b4a.png#align=left&display=inline&height=170&margin=%5Bobject%20Object%5D&originHeight=170&originWidth=644&size=0&status=done&style=none&width=644)

```
// main.ts
fn main() {
  println!("hello world!");
}
```

在终端窗口分别执行如下命令：

```
$ cd src
$ rustc main.rs
$ ./main
```

结果输出如下：

```
hello world!
```

rustc main.rs执行成功编译后，Rust输出二进制可执行文件main，如下：

![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1606745658425-e13b0b2f-a856-4886-b035-da4df90b8afa.png#align=left&display=inline&height=190&margin=%5Bobject%20Object%5D&originHeight=190&originWidth=656&size=0&status=done&style=none&width=656)

然后执行./main，运行main文件将得到我们期望的结果

### 3）包管理

Cargo是Rust的构建系统和包管理器，有点类似于npm+webpack的集成功能
查看cargo的版本：

```
$ cargo -V
```

创建一个名为test_cargo的项目：

```
$ cargo new test_cargo
```

里面会自动生成两个文件，src/main.rs和Cargo.toml，如图：
![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1606745658609-cd0ec63d-94e1-450a-8779-d21bf0e08d07.png#align=left&display=inline&height=314&margin=%5Bobject%20Object%5D&originHeight=314&originWidth=642&size=0&status=done&style=none&width=642)

其中src/main.rs的内容与上面我们自己创建的第一个程序hello world一样，而Cargo.toml的内容如下：

```
[package]
name = "test_cargo"
version = "0.1.0"
authors = ["your name <you@example.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```
有没有发现它跟我们的package.json很相似？

我们构建和运行的常用命令有：
```
// 构建项目
$ cargo build  
// 运行项目
$ cargo run
// 构建项目并检查错误（与cargo build相似，但不会生成二进制文件）
$ cargo check
```
通常情况下我们在本地运行的是cargo build，它的可运行文件在target/debug目录下

而在测试环境，运行的是cargo build --release，它的可运行文件在target/release目录下，如图

![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1606745658612-7a5014cf-79b4-4cef-b645-8b3932fb7eea.png#align=left&display=inline&height=828&margin=%5Bobject%20Object%5D&originHeight=828&originWidth=668&size=0&status=done&style=none&width=668)

```
$ cargo build  // 类似于npm build NODE_ENV=development
$ cargo build --release // 类似于npm build NODE_ENV=production
```

更多的命令可查看

```
$ cargo --help
```
同时Rust推荐使用下划线命名法，对驼峰命名法会给出警告，编译期间就会给出警告（编译通过不会报错）
```
use std::io;

fn main() {
    println!("Let's guess the number!");
    println!("Please input your number:");

    let mut numStr = String::new();

    io::stdin()
        .read_line(&mut numStr)
        .expect("Failed to read line");
    
    println!("you guessed: {}", numStr);

}
```


然后运行cargo build


![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1606745658606-649d2785-aea3-4651-9d8f-3454f0af767c.png#align=left&display=inline&height=748&margin=%5Bobject%20Object%5D&originHeight=748&originWidth=1762&size=0&status=done&style=none&width=1762)


我们按照它的规范，将numStr改成num_str，解析注释如下：


```
use std::io;// 表示输入/输出，与C++的语法类似
use std::cmp::Ordering;
use rand::Rng;// 需要在Cargo.toml的dependencies添加rand = "0.5.5"

fn main() {// fn函数声明
    println!("Let's guess the number!");
    let target_number = rand::thread_rng().gen_range(1, 101);// 表示[1-101),包含下限，不包括上限，即1-100之间的数字
    
    loop {// while循环
        println!("Please input your number:");

        let mut num_str = String::new();// mut表示可变的，mutable

        io::stdin()
            .read_line(&mut num_str)
            .expect("Failed to read line");// stdin表示输入，read_line读取一行数据，&表示引用，类似C++

        let num_str: u32 = match num_str.trim().parse() {// 对输入的数字进一步处理，忽略无效输入
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("you guessed: {}", num_str);

        match num_str.cmp(&target_number) {// 比较
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;// 跳出循环
            }
        }
    }
}
```

运行结果如下：
![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1606745658565-86827103-85b2-4406-a85e-ebfef875c47f.png#align=left&display=inline&height=1358&margin=%5Bobject%20Object%5D&originHeight=1358&originWidth=1442&size=0&status=done&style=none&width=1442)
### 4）基本概念

#### 1）变量let、可变性mut、常量const
| 内容 | 说明 |
| --- | --- |
| let | 变量，默认是不可变的变量（类似于常量） |
| mut | 可变，表示可变的变量 |
| const | 常量 |

至于let和const有什么区别，请看例子

```
let x = 5;
x = 6;
// 报错，x是不可变的变量，不允许赋值
```

```
let mut x = 5;
x = 6;
// 正常
```

```
let x = 5;
let x = 6;
// 正常，跟js有区别，属于重新声明和赋值
```
```
let mut x = 5;
x = “6”;
// 报错，不允许修改类型
```
```
let x = 5;
const MAX: u32 = 6000;
// 正常，const常量不允许搭配mut，且需要类型注释
```
#### 2）返回值
```
let x = 5;

let y = {
    let x = 3;
    x + 1
};

println!("{},{}", x, y);
  
  // 5, 4
```
请注意，该x + 1行的末尾没有分号，才会返回值；若有分号，表示一条语句，语句不会返回值，如下：
```
let x = 5;

let y = {
    let x = 3;
    x + 1;// 异常
};

println!("{},{}", x, y);
  
// error
```
同样的道理，在函数的返回值中也是一样
```
fn main() {
    let x = one(5);// 正常
    println!("{}", x);
    
    let y = ones(5);// 异常
    println!("{}", y);
}

fn one(x: i32) -> i32 {// 正常
    x + 1
}

fn ones(x: i32) -> i32 {// 异常
    x + 1;
}
// error
```
需要注意的是Rust的箭头函数为 -> 而不是 =>

同时值的类型不会自动转换，需要明确制定类型

```
fn main() {
  let number = 3;

  // 正常
  if number < 5 {
      println!("condition was true");
  } else {
      println!("condition was false");
  }

  // 异常
  if number {// 会抛出异常，数值不会自动转换为bool
    println!("number is 3");
  }

  // 异常
  let x = if true { 5 } else { "six" };// 抛出异常，因为x无法获得准确的类型（数值/字符串）

}
```


#### 3）循环


Rust控制循环有三种方式：loop, while和for


```
fn main() {
  // loop循环
  let mut x = 3;
  let result = loop {
    println!("x {}", x);
    x -= 1;
    if x == 0 {
      break x + 2;// 这里的返回值是带分号的
    }
  };
  println!("result: {}", result);// 2

  // while循环
  let mut y = 3;
  while y != 0 {
    println!("y {}", y);
    y -= 1;
  }

  // for循环
  let z = [1, 2, 3];
  for e in z.iter() {
    println!("z {}", e);
  }
}
```

下一章节将介绍Rust独特的垃圾管理机制

### 5）参考
官网：[https://doc.rust-lang.org/book/title-page.html](https://doc.rust-lang.org/book/title-page.html)
