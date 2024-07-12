# rust

在rust中,变量默认是immutable不可改变的,当变量不可变时,一旦值被绑定到一个名称上,这个值就不能被改变,这样的设计有助于防止意外的数据修改和并发问题.若要使变量可变,需要在声明时使用mut关键字.

```rust
let x =5;
let mut x =5;
```

rust中常量总是不可变化,使用const关键字,且只能被声明为常量表达式(必须是在编译期确定的值)


### 复合类型

+ array: [1,2,3]
+ tuple: (1,true)

#### tuple元组

元组是一个将多个其他类型的值组合进一个复合类型的主要方式.元组长度固定:一旦声明,其长度不会增长或缩小

```rust
fn main{
  let tup:(i32,f64,u8)=(500,12,12)
}

```

### 表达式和语句

>表达式会产生值,语句不产生值.
>
>表达式的末尾都没有分号,语句的末尾有分号.

 





    





## 所有权
所有权（系统）是 Rust 最为与众不同的特性，它让 Rust 无需垃圾回收器（garbage collector）即可保证内存安全。因此，理解 Rust 中所有权的运作方式非常重要。在本章中，我们将讨论所有权以及相关功能：借用、slice 以及 Rust 如何在内存中存放数据。

Rust 的核心功能（之一）是 所有权（ownership）。虽然该功能很容易解释，但它对语言的其他部分有着深刻的影响。

所有运行的程序都必须管理其使用计算机内存的方式。一些语言中具有垃圾回收机制，在程序运行时不断地寻找不再使用的内存；在另一些语言中，开发者必须亲自分配和释放内存。Rust 则选择了第三种方式：通过所有权系统管理内存，编译器在编译时会根据一系列的规则进行检查。在运行时，所有权系统的任何功能都不会减慢程序。

因为所有权对很多开发者来说都是一个新概念，需要一些时间来适应。好消息是随着你对 Rust 和所有权系统的规则越来越有经验，你就越能自然地编写出安全和高效的代码。持之以恒！

当你理解了所有权，你将有一个坚实的基础来理解那些使 Rust 独特的功能。在本章中，你将通过完成一些示例来学习所有权，这些示例基于一个常用的数据结构：字符串。

(rust尽量将安全问题解决在编译期,并且编译器对于用户非常友好)

string 类型为了支持可变可增长的文本片段,需要在堆上分配一块在编译时未知大小的内存

+ 必须再运行时向内存分配器请求内存
+ 需要一个当我们处理完string时将内存返回给分配器的方法


然而，第二部分实现起来就各有区别了。在有 垃圾回收（garbage collector，GC）的语言中， GC 记录并清除不再使用的内存，而我们并不需要关心它。没有 GC 的话，识别出不再使用的内存并调用代码显式释放就是我们的责任了，跟请求内存的时候一样。从历史的角度上说正确处理内存回收曾经是一个困难的编程问题。如果忘记回收了会浪费内存。如果过早回收了，将会出现无效变量。如果重复回收，这也是个 bug。我们需要精确地为一个 allocate 配对一个 free。

Rust 采取了一个不同的策略：内存在拥有它的变量离开作用域后就被自动释放。下面是示例 4-1 中作用域例子的一个使用 String 而不是字符串字面量的版本：

``` rust
fn main() {
    {
        let s = String::from("hello"); // 从此处起，s 开始有效

        // 使用 s
    }                                  // 此作用域已结束，
                                       // s 不再有效
}
```
当变量离开作用域时，Rust 为我们调用一个特殊的函数。这个函数叫做 `drop`，它的功能是释放我们的内存。Rust 在结尾的 `}` 处调用 `drop`。这里，Rust 在 `}` 处调用 `drop`，这样就可以在这个作用域中不再需要这个变量的时候释放这个内存。

可变引用和不可变引用的区别在于是否可以修改引用的值

## String slice
字符串 slice 是 String 中一部分值的引用。String slice 的类型声明写作 `&str`。更具体地说，字符串 slice 是一个指向字符串开头的位置的指针，以及 slice 的长度。这也就是为什么字符串 slice 是一个引用：它没有所有权。
slice依然是左闭右开,符合自然的习惯(好算长度,另外也符合逻辑,不会出现定义不能覆盖的情形)


## rust 没有NUll
在很多语言中，null 是一个值，代表没有值。在 Rust 中，我们使用 Option<T> 来表达一个值可能为空的情况。Option<T> 是一个枚举。Option<T> 有两个变量：Some(T) 和 None。Some(T) 保存一个 T 类型的值，而 None 代表没有值。
```
enum Option<T>{
    Some(T),
    None,
}
```
这个枚举和泛型结合的方式是 Rust 的一个强大功能。Option<T> 使得 Rust 不需要 null 检查。Option<T> 有一个 expect 方法，如果 Option<T> 是 None，expect 会导致程序崩溃并显示 expect 中的信息。这个方法在调试时非常有用，但在生产环境中，你可能会选择其他处理 None 的方法。


Option 枚举和其相对于空值的优势
这一部分会分析一个 Option 的案例，Option 是标准库定义的另一个枚举。Option 类型应用广泛因为它编码了一个非常普遍的场景，即一个值要么有值要么没值。

例如，如果请求一个非空列表的第一项，会得到一个值，如果请求一个空的列表，就什么也不会得到。从类型系统的角度来表达这个概念就意味着编译器需要检查是否处理了所有应该处理的情况，这样就可以避免在其他编程语言中非常常见的 bug。

编程语言的设计经常要考虑包含哪些功能，但考虑排除哪些功能也很重要。Rust 并没有很多其他语言中有的空值功能。空值（Null ）是一个值，它代表没有值。在有空值的语言中，变量总是这两种状态之一：空值和非空值。

Tony Hoare，null 的发明者，在他 2009 年的演讲 “Null References: The Billion Dollar Mistake” 中曾经说到：

> I call it my billion-dollar mistake. At that time, I was designing the first comprehensive type system for references in an object-oriented language. My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.
>
> 我称之为我十亿美元的错误。当时，我在为一个面向对象语言设计第一个综合性的面向引用的类型系统。我的目标是通过编译器的自动检查来保证所有引用的使用都应该是绝对安全的。不过我未能抵抗住引入一个空引用的诱惑，仅仅是因为它是这么的容易实现。这引发了无数错误、漏洞和系统崩溃，在之后的四十多年中造成了数十亿美元的苦痛和伤害。

``` rust
enum Option<T>{
    None,
    Some(T),
}

let some_number=Some(5);
let some_char=Some('a');
let absent_number:Option<i32>=None;
```
Option<T> 和 T 是不同的类型，因为 Option<T> 和 T 是不同的枚举成员。这意味着编译器可以在编译时检查是否正确地处理了所有情况，而不会出现空值错误。

当有一个Some(T)值时，T 就是我们的值，当有一个 None 时，没有值。Option<T> 是一个非常通用的枚举，它在标准库中有很多用途。

option的定义让我们在编译时就能够知道是否处理了所有的情况,这样就不会出现空值错误(例如,Option<i8>和i8,i8编译器可以确保总是有一个有效的值)

换句话说，在对 Option<T> 进行运算之前必须将其转换为 T。通常这能帮助我们捕获到空值最常见的问题之一：假设某值不为空但实际上为空的情况。

消除了错误地假设一个非空值的风险，会让你对代码更加有信心。为了拥有一个可能为空的值，你必须要显式的将其放入对应类型的 Option<T> 中。接着，当使用这个值时，必须明确的处理值为空的情况。只要一个值不是 Option<T> 类型，你就 可以 安全的认定它的值不为空。这是 Rust 的一个经过深思熟虑的设计决策，来限制空值的泛滥以增加 Rust 代码的安全性。

那么当有一个 Option<T> 的值时，如何从 Some 成员中取出 T 的值来使用它呢？Option<T> 枚举拥有大量用于各种情况的方法：你可以查看它的文档。熟悉 Option<T> 的方法将对你的 Rust 之旅非常有用。

总的来说，为了使用 Option<T> 值，需要编写处理每个成员的代码。你想要一些代码只当拥有 Some(T) 值时运行，允许这些代码使用其中的 T。也希望一些代码只在值为 None 时运行，这些代码并没有一个可用的 T 值。match 表达式就是这么一个处理枚举的控制流结构：它会根据枚举的成员运行不同的代码，这些代码可以使用匹配到的值中的数据。

### 匹配 Option<T>
我们可以使用 match 表达式来处理 Option<T>。这里是一个示例，它只会在 Option<T> 是 Some(3) 时执行代码：

``` rust
fn main() {
    let some_number = Some(5);
    let some_string = Some("a string");

    let absent_number: Option<i32> = None;

    match some_number {
        Some(i) => println!("This is a number: {}", i),
        None => println!("This is not a number"),
    }

    match some_string {
        Some(s) => println!("This is a string: {}", s),
        None => println!("This is not a string"),
    }

    match absent_number {
        Some(i) => println!("This is a number: {}", i),
        None => println!("This is not a number"),
    }
}
```
Rust中的match是穷尽的(exhaustive):必须覆盖所有可能的情况,这样可以确保不会有任何的空值错误.

## 使用包,Crate,模块组织代码

### 封装(encapsulation)

我们也会讨论封装来实现细节，这可以使你更高级地重用代码：你实现了一个操作后，其他的代码可以通过该代码的公共接口来进行调用，而不需要知道它是如何实现的。你在编写代码时可以定义哪些部分是其他代码可以使用的公共部分，以及哪些部分是你有权更改实现细节的私有部分。这是另一种减少你在脑海中记住项目内容数量的方法。

“Encapsulate” 是一个英文单词，其词源和语义可以通过分析其拉丁词根和前缀来理解。以下是对这个单词的详细介绍。

### 词源

“Encapsulate” 来源于拉丁语，其组成部分如下：

1. **前缀 “en-”**：这个前缀表示“进入”或“使成…”，来源于拉丁语“in-”。
2. **词根 “capsula”**：这是拉丁语中的一个小盒子或容器，源自“capsa”，意思是盒子。
3. **后缀 “-ate”**：这是一个动词后缀，表示“使成为”或“进行…的动作”。

因此，“encapsulate” 的原始意思是“放入小盒子中”或“使其成为一个容器”。

### 语义

现代英语中，“encapsulate” 的意思有以下几个层面：

1. **物理上的封装**：将某物封闭在一个容器或保护壳中。
   - 例如，药品被封装在胶囊中。

2. **概念上的封装**：将信息、功能或数据隐藏或封闭起来，只暴露必要的部分。
   - 例如，在计算机科学中，封装是面向对象编程的重要概念，它将对象的状态和行为封装在一个对象内，只允许通过定义好的接口进行访问。

### 示例

#### 物理上的封装

- **句子**: The medicine is encapsulated in a gelatin capsule.
  - **翻译**: 这种药物被封装在一个明胶胶囊中。

#### 概念上的封装

- **句子**: In software engineering, encapsulation is used to hide the internal state and implementation details of an object.
  - **翻译**: 在软件工程中，封装用于隐藏对象的内部状态和实现细节。

### 相关概念

#### 封装（Encapsulation）在计算机科学中的应用

封装是面向对象编程（OOP）的四大基本原则之一（其他三个是继承、多态和抽象）。它主要涉及以下方面：

1. **数据隐藏**：通过限制访问权限来隐藏对象的内部数据，仅允许通过公开的方法访问和修改数据。这有助于保护对象的完整性和防止意外的干扰。
   - 例如，在一个类中，使用 `private` 关键字将属性私有化，然后提供 `public` 方法进行访问和修改。

2. **接口定义**：定义对象与外部交互的接口，使对象的内部实现可以独立于外部使用方式进行修改。
   - 例如，通过 getter 和 setter 方法访问私有属性。

#### 示例代码

以下是一个简单的示例，展示了封装在面向对象编程中的应用：

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // 构造函数
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }

    // 获取宽度
    fn width(&self) -> u32 {
        self.width
    }

    // 获取高度
    fn height(&self) -> u32 {
        self.height
    }

    // 计算面积
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect = Rectangle::new(10, 20);
    println!("The width of the rectangle is: {}", rect.width());
    println!("The height of the rectangle is: {}", rect.height());
    println!("The area of the rectangle is: {}", rect.area());
}
```

在这个示例中，`Rectangle` 结构体的宽度和高度通过构造函数进行初始化，并且提供了方法来访问这些属性和计算面积。通过这种方式，我们封装了数据，并定义了对数据的访问接口。

### 总结

“Encapsulate” 这个词源自拉丁语，表示将某物封闭在一个小容器中。现代语义上，它既可以指物理上的封装，也可以指概念上的封装，如在计算机科学中的数据隐藏和接口定义。通过理解其词源和语义，我们可以更好地理解和应用这个概念。

### 作用域(scope)

这里有一个需要说明的概念 “作用域（scope）”：代码所在的嵌套上下文有一组定义为 “in scope” 的名称。当阅读、编写和编译代码时，程序员和编译器需要知道特定位置的特定名称是否引用了变量、函数、结构体、枚举、模块、常量或者其他有意义的项。你可以创建作用域，以及改变哪些名称在作用域内还是作用域外。同一个作用域内不能拥有两个相同名称的项；可以使用一些工具来解决名称冲突。

“Crates” 是一个英语单词，通常指大木箱或包装箱。它的词源和历史可以追溯到拉丁语和古希腊语。以下是对这个词源的详细介绍：

### 词源

1. **拉丁语**：这个词最初来自拉丁语的 “cratis”，意思是“编织的筐”或“格子”。“Cratis” 指的是一种用来装载或运输物品的容器，通常由木条或藤条编织而成。

2. **古希腊语**：这个词根源自古希腊语 “kratos”（κράτος），意思是“力量”或“支撑”。虽然直接联系不太明显，但它可能影响了“cratis” 的用法，因为木条编织的容器需要具有一定的强度和支撑力。

3. **古法语**：在进入英语之前，这个词通过古法语 “crate” 或 “crat” 被引入，保持了类似的含义，即用来运输货物的容器或筐子。

### 英语中的演变

在英语中，“crate” 一词逐渐演变成指用于运输和储存大件物品的木箱或大箱子。现代英语中，“crate” 主要用于描述坚固的容器，通常由木头或塑料制成，用于包装、运输和储存各种物品。

### 用法示例

- **普通用法**：
  - **句子**：We used a wooden crate to ship the furniture overseas. 
  - **翻译**：我们用一个木箱把家具运到海外。

- **比喻用法**：
  - **句子**：The old car was such a crate, it barely ran.
  - **翻译**：那辆旧车简直就是个破箱子，几乎都不能开。

### 相关术语

- **Crating**：指的是使用箱子包装或运输物品的过程。
  - **句子**：The company specializes in crating and shipping fine art.
  - **翻译**：这家公司专门从事美术品的包装和运输。

- **Milk Crate**：一种用于运输和储存牛奶瓶的坚固塑料箱子。
  - **句子**：He used a milk crate as a makeshift chair.
  - **翻译**：他用一个牛奶箱临时当椅子。

### 总结

“Crate” 一词源自拉丁语 “cratis”，指编织的筐或格子，经过古法语进入英语后演变成描述运输和储存容器的常用术语。在现代英语中，“crate” 主要指用于包装和运输的坚固箱子。这种词源的演变展示了语言如何随着时间和文化交流不断发展。

binary crate 起始点一般是main.rs,library crate 起始点一般是lib.rs,

              



