# Swift 代码规范

本文尝试鼓励规范来完成以下目标(大致地先后顺序):

1. 提高严谨性，减少开发者出错的概率
2. 明确意图
3. 减少冗余
4. 减少关于美的讨论

如果你有什么建议，开个`pull request`。

## 留白

* 使用`Tab`键，而不是空格。
* 文件结束时留一空行。
* 充分利用空行将代码分离成逻辑块。
* 不要在一行结尾留下空格。
  * 甚至不要在空行开头留有缩进。
  

## 尽可能使用`let`绑定而不是`var`绑定

尽可能使用`let foo = …`而不是`var foo = …`(并且当困惑的时候)。只有当你完全必须使用的时候才用`var`(例如，你知道这个值可能改变。例如，当使用`weak`存储修饰符时)。

原因:这两个关键词的用处和含义都很清晰，但是*默认使用let*会使代码更安全清晰。

`let`绑定保证而且开发者很清楚它的值不会改变。随后，代码的意图更加明确。

写代码更容易啦。一旦你使用`var`，还要假设它的值不会改变，你不得不人工地校验。

于是，当你看到`var`被使用时，假设它的值会改变，并问问你自己为什么。


## 早早地Return和break 

当你遇到确切的条件来继续执行时，尝试早早地退出。因此，不要这样使用：

```swift
if n.isNumber {
    // Use n here
} else {
    return
}
```

这样使用:
```swift
guard n.isNumber else {
    return
}
// Use n here
```

你也可以使用`if`语句，但是推荐使用`guard`，因为`guard`语句不会像`return`, `break` 和 `continue`引起编译错误，因此退出是安全的。

## 避免可选类型的强制解析

如果你有一个`FooType?` 或者 `FooType!`类型的标识符`foo`，如果可以的话，不要强制解析它来获取基础类型`foo!`。

相反，推荐这样:

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```

或者，在某些场景下，你可能想要使用Swift的可选类型链，例如:

```swift
// Call the function if `foo` is not nil. If `foo` is nil, ignore we ever tried to make the call
foo?.callSomethingIfFooIsNotNil()
```

原因: 明确的`if let`可选类型绑定可以使代码更安全。强制解析更容易引起运行时崩溃(runtime crashes)。


## 避免使用隐式解析可选类型

无论哪里，如果`foo`可能是`nil`，使用`let foo: FooType?`，而不是`let foo: FooType!`(注意，一般来说，使用`?`，而不是`!`)。

原因: 明确的可选类型可以使代码更安全。隐式解析可选类型(Implicitly unwrapped optionals)有潜在地运行时崩溃的风险。


## 推荐在只读属性和下标脚本(subscripts)中使用隐式getters

如果可能的话，在只读计算属性和只读下标脚本(subscripts)上忽略`get`关键词。

因此，这样写:

```swift
var myGreatProperty: Int {
	return 4
}

subscript(index: Int) -> T {
    return objects[index]
}
```

而不是这样:

```swift
var myGreatProperty: Int {
	get {
		return 4
	}
}

subscript(index: Int) -> T {
    get {
        return objects[index]
    }
}
```

原因: 第一个版本代码的意图和含义都很清晰，并且只需要少量代码。


## 总是为顶级定义(top-level definitions)指定明确地访问控制作用域

总是应该为顶级函数、类型和变量指定明确的访问控制说明符。

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

然而，在它们内部的定义可以适当地保持默认地访问控制作用域：

```swift
internal struct TheFez {
	var owner: Person = Joshaber()
}
```

原因：为顶级定义明确地指定`internal`是罕有的情形，并且明确地确保是经过深思熟虑。在一个定义(definition)内部,重复使用相同的访问控制说明符略显有些重复，保持默认通常是合理的。

## 当定义一个类型，总是用冒号和标识符关联起来

当指定一个标识符的类型，总是把冒号直接放在标识符的后面，后面跟一个空格，然后是类型名称。

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

原因: 类型说明符表达一些关于标识符的信息，所以应该和它放在一块。

同样，当指定一个字典的类型，总是把冒号直接放在键类型的后面，紧跟一个空格，然后一个值类型。

```swift
let capitals: [Country: City] = [ Sweden: Stockholm ]
```

## 必需时才明确地使用`self`

当访问关于`self`的属性或方法时，默认保留`self`的隐式引用。

```swift
private class History {
	var events: [Event]

	func rewrite() {
		events = []
	}
}
```

在语言必需时，才包含这个明确的关键词-例如，在一个闭包中或参数名称冲突:

```swift
extension History {
	init(events: [Event]) {
		self.events = events
	}

	var whenVictorious: () -> () {
		return {
			self.rewrite()
		}
	}
}
```

原因: 这样的话，`self`在闭包中的语义更加明显，并且避免在其它地方冗余。


## 优先使用结构体(structs)，其次才是类(classes)

除非你需要只有类才能提供的功能(像identity或析构器(deinitializers))，否则使用结构体来替代。

注意，继承通常不是一个好的使用类的理由，因为多态可以由协议(protocols)来提供，并且可以通过协议合成(composition)来重复利用。

比如，这个类的层级：

```swift
class Vehicle {
    let numberOfWheels: Int

    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * Float(numberOfWheels)
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```

可以重构成这些定义：

```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
}

func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
    return pressurePerWheel * Float(vehicle.numberOfWheels)
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

原因: 值类型更简单、容易使用，而且通过`let`关键词达到需求。


## 默认使用`final`修饰类

类应该被`final`修饰，仅仅当确实有一个继承的需求被鉴定时，才允许继承。
即使在那种情形，遵循相同的原则，尽可能多的把类中的定义都用`final`修饰。

原因: 合成通常适合用于继承，并且如果希望加入继承需要多思考。


## 能省略类型参数就省略

参数化类型的方法可以省略关于接收类型的类型参数，当它们对接收者来说都一样时。

例如：

```swift
struct Composite<T> {
	…
	func compose(other: Composite<T>) -> Composite<T> {
		return Composite<T>(self, other)
	}
}
```

可以转换为:

```swift
struct Composite<T> {
	…
	func compose(other: Composite) -> Composite {
		return Composite(self, other)
	}
}
```

原因:省略多余的类型参数可以使目的更明确，并且当返回的类型有几种不同的类型参数时对比鲜明。


## 在操作符定义两边使用空格

当定义操作符时，在操作符两边使用空格。不要这样写：

```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

这样写:

```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

原因：由标点符号字符组成的操作符，当后面紧跟一个类型的标点符号或值参数列表时，不便于阅读。添加空格来分割他们更加清晰。























































