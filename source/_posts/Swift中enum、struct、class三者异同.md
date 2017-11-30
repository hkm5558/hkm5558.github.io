---
title: Swift中enum、struct、class三者异同
date: 2017-11-29 18:30:34
tags: [Swift, Class, Enum, Struct]
categories: Swift
---

## 写在前面 ##
由于在开发过程中常常需要用到系统提供的基础类型之外的的类型，所以`Swift`允许我们根据自己的需要构建属于自己的类型系统以便于更加灵活和方便的开发程序并将其称之为`named types`。
`Swift`主要为我们提供了以下四种`named types` 分别是：`enum`、`struct`、`class`和`protocol`，  相信熟悉`objective-c`开发的同学们对于`iOS`中枚举、结构体和类的概念一点都不陌生。相比于`objective-c`中的这三者，`Swift`将`enum`和`struct`变得更加灵活且强大，并且赋予了他们很多和`class`相同的属性实现更加丰富多彩的功能，以至于有时候我们很难分清他们到底有什么区别以及我该什么时候用哪种类型，接下来本文将介绍在`Swift`中`enum`和`struct`的定义和新特性以及两者与`class`之间的异同。

## 枚举（enum） ##

 - 枚举的定义
`Swift`中的枚举是为一组有限种可能性的相关值提供的通用类型（在`C`/`C++`/`C#`中，枚举是一个被命名的整型常数的集合）；使用枚举可以类型安全并且有提示性地操作这些值。与结构体、类相似，使用关键词`enum`来定义枚举，并在一对大括号内定义具体内容包括使用`case`关键字列举成员。就像下面一样：

```swift
//定义一个表示学生类型的全新枚举类型 StudentType，他有三个成员分别是pupil（小学生，）、middleSchoolStudent（中学生）、collegeStudents（大学生）
enum StudentType {
  case pupil
  case middleSchoolStudent
  case collegeStudent
}
```

上面的代码可以读作：如果存在一个`StudentType`的实例，他要么是`pupil` （小学生）、要么是`middleSchoolStudent`（中学生）、要么是`collegeStudent`（大学生）。注意，和`C`、o`bjective-c`中枚举的不同，`Swift` 中的枚举成员在被创建时不会分配一个默认的整数值。而且不需要给枚举中的每一个成员都提供值(如果你需要也是可以的)。如果一个值（所谓“原始值”）要被提供给每一个枚举成员，那么这个值可以是字符串、字符、任意的整数值，或者是浮点类型（引自文档翻译）。简单说`Swift`中定义的枚举只需要帮助我们表明不同的情况就够了，他的成员可以没有值，也可以有其他类型的值（不局限于整数类型）。
枚举中有两个很容易混淆的概念：原始值(`raw value`)、关联值(`associated value`)，两个词听起来比较模糊，下面简单介绍一下：

 - 枚举的原始值(`raw value`)
 枚举成员可以用相同类型的默认值预先填充，这样的值我们称为原始值(`raw value`)，下面的`StudentType`中三个成员分别被Int类型的10 、15、 20填充表示不同阶段学生的年龄。注意：`Int`修饰的是`StudentType`成员原始值的类型而不是`StudentType`的类型，`StudentType`类型从定义开始就是一个全新的枚举类型。

```swift
enum StudentType: Int{
    case pupil = 10
    case middleSchoolStudent = 15
    case collegeStudents = 20
}
```

定义好`StudentType`成员的原始值之后，我们可以使用枚举成员的`rawValue`属性来访问成员的原始值，或者是使用原始值初始化器来尝试创建一个枚举的新实例

```swift
//  常量student1值是 10
let student1 = StudentType.pupil.rawValue
//  变量student2值是 15
var student2 = StudentType.middleSchoolStudent.rawValue
//  使用成员rawValue属性创建一个`StudentType`枚举的新实例
let student3 = StudentType.init(rawValue: 15)
//  student3的值是 Optional<senson>.Type
type(of: student3)
//  student4的值是nil，因为并不能通过整数30得到一个StudentType实例的值
let student4 = StudentType.init(rawValue: 30)
```

使用原始值初始化器这种方式初始化创建得到`StudentType`的实例`student4`是一个`StudentType`的可选类型，因为并不是给定一个年龄就能找到对应的学生类型，比如在`StudentType`中给定年龄为30就找不到对应的学生类型（很可能30岁的人已经是博士了）。所以原始值初始化器是一个可失败初始化器。
总结一句：原始值是为枚举的成员们绑定了一组类型必须相同值不同的固定的值（可能是整型，浮点型，字符类型等等）。这样很好解释为什么提供原始值的时候用的是等号。

 - 枚举的关联值(`associated value`)
 关联值和原始值不同，关联值更像是为枚举的成员们绑定了一组类型，不同的成员可以是不同的类型(提供关联值时用的是括号)。例如下面的代码

```swift
//定义一个表示学生类型的枚举类型 StudentType，他有三个成员分别是pupil、middleSchoolStudent、collegeStudents
enum StudentType {
  case pupil(String)
  case middleSchoolStudent(Int, String)
  case collegeStudents(Int, String)
}
```

这里我们并没有为`StudentType`的成员提供具体的值，而是为他们绑定了不同的类型，分别是`pupil`绑定`String`类型、mid`dleSchoolStudent`和`collegeStudents`绑定（`Int`， `String`）元组类型。接下来就可以创建不同`StudentType`枚举实例并为对应的成员赋值了。
```swift
//student1 是一个StudentType类型的常量，其值为pupil（小学生），特征是"have fun"（总是在玩耍）
let student1 = StudentType.pupil("have fun")
  //student2 是一个StudentType类型的常量，其值为middleSchoolStudent（中学生），特征是 7, "always study"（一周7天总是在学习）
let student2 = StudentType.middleSchoolStudent(7, "always study")
  //student3 是一个StudentType类型的常量，其值为collegeStudent（大学生），特征是 7, "always LOL"（一周7天总是在撸啊撸）
let student3 = StudentType.middleSchoolStudent(7, "always LOL")
```

这个时候如果需要判断某个`StudentType`实例的具体的值就需要这样做了：

```swift
switch student3 {
    case .pupil(let things):
        print("is a pupil and \(things)")
    case .middleSchoolStudent(let day, let things):
        print("is a middleSchoolStudent and \(day) days \(things)")
    case .collegeStudent(let day, let things):
        print("is a collegeStudent and \(day) days \(things)")
  }
```

控制台输出：is a collegeStudent and 7 days always LOL，看到这你可能会想，是否可以为一个枚举成员提供原始值并且绑定类型呢，答案是不能的！因为首先给成员提供了固定的原始值，那他以后就不能改变了；而为成员提供关联值(绑定类型)就是为了创建枚举实例的时候赋值。这不是互相矛盾吗。

 - 递归枚举
 递归枚举是拥有另一个枚举作为枚举成员关联值的枚举（引自文档翻译）。
关于递归枚举我们可以拆封成两个概念来看：递归 + 枚举。递归是指在程序运行中函数（或方法）直接或间接调用自己的这样一种方式，其特点为重复有限个步骤、格式较为简单。下面是一个经典的通过递归算法求解n!（阶乘）的函数。

```swift
func factorial(n: Int)->Int {
    if n > 0 {
        return n * factorial(n: n - 1)
    } else {
        return 1
    }
}
//1 * 2 * 3 * 4 * 5 * 6 = 720
let sum = factorial(n: 6)
```

函数`factorial (n: int)-> Int`在执行过程中很明显的调用了自身。结合枚举的概念我们这里可以简单的理解为递归枚举类似上面将枚举值本身传入给成员去判断的情况。


## 结构体（struct） ##

 - 结构体的定义
 结构体是由一系列具有相同类型或不同类型的数据构成的数据集合。结构体是一种值类型的数据结构，在`Swift`中常常使用结构体封装一些属性甚至是方法来组成新的复杂类型，目的是简化运算。我们通过使用关键词`struct`来定义结构体。并在一对大括号内定义具体内容包括他的成员和自定义的方法（是的，`Swift`中的结构体有方法了），定义好的结构体存在一个自动生成的成员初始化器，使用它来初始化结构体实例的成员属性。废话不多说直接上代码:

```swift
//定义一个 Student（学生）类型的结构体用于表示一个学生，Student的成员分别是语、数、外三科`Int`类型的成绩
struct Student {
  var chinese: Int
  var math: Int
  var english: Int
}
```

看到这里熟悉`Swift`的同学可能已经发现了一点结构体和类的区别了：定义结构体类型时其成员可以没有初始值。如果使用这种格式定义一个类，编译器是会报错的，他会提醒你这个类没有被初始化。

 - 结构体实例的创建
 创建结构体和类的实例的语法非常相似，结构体和类两者都能使用初始化器语法来生成新的实例。最简单的语法是在类或结构体名字后面接一个空的圆括号，例如 `let student1 = Student()`。这样就创建了一个新的类或者结构体的实例，任何成员都被初始化为它们的默认值（前提是成员均有默认值）。但是结合上面的代码，由于在定义Student结构体时我们并没有为他的成员赋初值，所以 `let student1 = Student()`在编译器中报错了，此处报错并不是因为不能这样创建实例而是因为`student1`成员没有默认值，所以我们可以使用下面的方式创建实例:

```swift
//使用Student类型的结构体创建Student类型的实例（变量或常量）并初始化三个成员（这个学生的成绩会不会太好了点）
let student2 = Student(chinese: 90, math: 80, english: 70)
```

所有的结构体都有一个自动生成的成员初始化器，你可以使用它来初始化新结构体实例的成员就像上面一样（前提是没有自定义的初始化器）。如果我们在定义`Student`时为他的成员赋上初值，那么下面的代码是编译通过的：

```swift
struct Student {
  var chinese: Int = 50
  var math: Int = 50
  var english: Int = 50
}
let student2 = Student(chinese: 90, math: 80, english: 70)
let student4 = Student()
```

总结一句：定义结构体类型时其成员可以没有初始值，但是创建结构体实例时该实例的成员必须有初值。

 - 自定义的初始化器
 当我们想要使用自己的方式去初始化创建一个Student类型的实例时，系统提供的成员初始化器可能就不够用了。例如，我们希望通过形如 `let student5 = Student(stringScore: "70,80,90")` 的方式创建实例时，就需要自定义初始化方法了：

```swift
struct Student {
  var chinese: Int = 50
  var math: Int = 50
  var english: Int = 50
      init() {}
      init(chinese: Int, math: Int, english: Int) {
            self.chinese = chinese
            self.math = math
           self.english = english
      }
      init(stringScore: String) {
           let cme = stringScore.characters.split(separator: ",")
           chinese = Int(atoi(String(cme.first!)))
           math = Int(atoi(String(cme[1])))
           english = Int(atoi(String(cme.last!)))
      }
  }
  let student6 = Student()
  let student7 = Student(chinese: 90, math: 80, english: 70)
  let student8 = Student(stringScore: "70,80,90")
```

一旦我们自定义了初始化器，系统自动的初始化器就不起作用了，如果还需要使用到系统提供的初始化器，在我们自定义初始化器后就必须显式的定义出来。

 - 定义其他方法
如果此时需要修改某个学生某科的成绩，该如何实现呢？当然，我们可以定义下面的方法：

```swift
//更改某个学生某门学科的成绩
func changeChinese(num: Int, student: inout Student){
  student.chinese += num
}
changeChinese(num: 20, student: &student7)
```

此时`student7`的语文成绩就由原来的`90`被修改到了`110`，但是此方法有两个明显的弊端：1，学生的语文成绩`chinese`是`Student`结构体的内部成员，一个学生的某科成绩无需被`Student`的使用者了解。即我们只关心学生的语文成绩更改了多少，而不是关心学生语文成绩本身是多少。2，更改一个学生的语文成绩本身就是和`Student`结构体内部成员计算相关的事情，我们更希望达到形如：`student7.changeChinese(num: 10)` 的效果，因为只有学生本身清楚自己需要将语文成绩更改多少（更像是面向对象封装的思想）。很明显此时`changeChinese(num:)`方法是`Student`结构体内部的方法而不是外部的方法，所以我定义了一个修改某个学生数学成绩的内部方法用于和之前修改语文成绩的外部方法对比：

```swift
struct Student {
    var chinese: Int = 50
    var math: Int = 50
    var english: Int = 50
   //修改数学成绩
    mutating func changeMath(num: Int) {
        self.math += num
    }
  }
  var student7 = Student(chinese: 20, math: 30, english: 40)
  //更改分数中语文学科的成绩
  func changeChinese(num: Int, student: inout Student){
      student.chinese += num
    }
  changeChinese(num: 20, student: &student7)
  student7.changeMath(num: 10)
```

尽管两者都能达到同样的效果，但是把修改结构体成员的方法定义在结构体内部显得更加合理同时满足面向对象封装的特点。以上两点就是我们为`Student`结构体内部添加`changeMath(num:)`的原因，他让我们把类型相关的计算表现的更加自然和统一，即自己的事情应该用自己的方法实现不应该被别人关心。值得一提的是在结构体内部方法中如果修改了结构体的成员，那么该方法之前应该加入：`mutating`关键字。由于结构体是值类型，`Swift`规定不能直接在结构体的方法（初始化器除外）中修改成员。原因很简单，结构体作为值的一种表现类型怎么能提供改变自己值的方法呢，但是使用`mutating`我们便可以办到这点，当然这也是和类的不同点。

- 常见的结构体
`Swift`中很多的基础数据类型都是结构体类型，下面列举的是一些常用的结构体类型：

```swift
//表示数值类型的结构体：
  Int，Float，Double，CGFloat...
//表示字符和字符串类型的结构体
  Character，String...
//位置和尺寸的结构体
  CGPoint，CGSize...
//集合类型结构体
  Array，Set，Dictionary...
```

很多时候你不细心观察的话可能不会想到自己信手拈来的代码中居然藏了这么多结构体。另外有时候在使用类和结构体的时候会出现下面的情况

```swift
// Person 类
class Person {
    var name: String = "jack"
    let life: Int = 1
}
    var s1 = Person()
    var s2 = s1
     s2.name = "mike"
```

```swift
// People 结构体数据结构
struct People {
    var name: String = "jack"
    let life: Int = 1
}
    var p1 = People()
    var p2 = p1
      p2.name = "mike"
```

细心的同学可能已经发现了其中的诡异。变量s1、s2是Person类的实例，修改了s2的name属性，s1的name也会改变；而p1、p2作为People结构体的实例，修改了p1的name属性，p2的name并不会发生改变。这是因为 `struct` 是值引用，`class` 是类型引用。

## 总结 ##

> 枚举、结构体、类的共同点：
>
 - 定义属性和方法；
 - 下标语法访问值；
 - 初始化器；
 - 支持扩展增加功能；
 - 可以遵循协议；
> 
> 类特有的功能；
>
 - 继承；
 - 允许类型转换；
 - 析构方法释放资源；
 - 引用计数；
 
 
 
 类是引用类型
> 引用类型(`reference types`，通常是类)被复制的时候其实复制的是一份引用，两份引用指向同一个对象。所以在修改一个实例的数据时副本的数据也被修改了(s1、s2)。

枚举，结构体是值类型
> 值类型(`value types`)的每一个实例都有一份属于自己的数据，在复制时修改一个实例的数据并不影响副本的数据(p1、p2)。值类型和引用类型是这三兄弟最本质的区别。

该如何选择

> 关于在新建一个类型时如何选择到底是使用值类型还是引用类型的问题其实在理解了两者之间的区别后是非常简单的，在这苹果官方已经做出了非常明确的指示（以下内容引自苹果官方文档）：
>> 当你使用Cocoa框架的时候，很多API都要通过NSObject的子类使用，所以这时候必须要用到引用类型class。在其他情况下，有下面几个准则：
1.什么时候该用值类型：
要用==运算符来比较实例的数据时
你希望那个实例的拷贝能保持独立的状态时
数据会被多个线程使用时  
2.什么时候该用引用类型（class）：   
要用==运算符来比较实例身份的时候
你希望有创建一个共享的、可变对象的时候