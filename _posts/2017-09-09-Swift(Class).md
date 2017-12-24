
## 类

#### 简介
swift的类和c++的类基本相似，这里整理不同的地方，以突显其独有特性

#### 1.属性
swift class的属性有三种：存储属性，计算属性和类型属性。存储属性存储常量或变量作为实例的一部分，而计算属性计算(不是存储)一个值。类型属性则直接作用于类型本身，而不是类的实例。

一个典型例子：

```
struct Point {
    var x = 0.0, y = 0.0//存储属性
}
struct Size {
    var width = 0.0, height = 0.0
}
struct Rect {
    static let noUseVar =  0//类属性
    var origin = Point()
    var size = Size()
    var center: Point {//计算属性
        get {
            let centerX = origin.x + (size.width / 2)
            let centerY = origin.y + (size.height / 2)
            return Point(x: centerX, y: centerY)
        }
        set(newCenter) {
            origin.x = newCenter.x - (size.width / 2)
            origin.y = newCenter.y - (size.height / 2)
        }
    }
}
```

计算属性并不存储真实的值，而只是一种计算载体，提供获取 Rect 实例的 center 的方法。只有 getter 没有 setter 的计算属性就是只读计算属性。


```必须使用var关键字定义计算属性，包括只读计算属性，因为它们的值不是固定的。 关键字只用来声明 常量属性，表示初始化后再也无法修改的值。
```

#### 2.属性观察器
属性观察器监控和响应属性值的变化，每次属性被设置值的时候都会调用属性观察器，即使新值和当前值相同的时候也不例外。

可以为属性添加如下的一个或全部观察器:

 1. willSet：在新的值被设置之前调用
 2. didSet：在新的值被设置之后立即调用

```
class StepCounter {
    var totalSteps: Int = 0 {
        willSet(newTotalSteps) {
            print("About to set totalSteps to \(newTotalSteps)")
        }
        didSet {
            if totalSteps > oldValue  {
                print("Added \(totalSteps - oldValue) steps")
            }
        }
    }
}
```
oldValue表示旧值的参数名

#### 3.延迟属性、防止重写属性
延迟属性（存储属性）是指当第一次被调用的时候才会计算其初始值的属性,在属性声明前使用lazy来标示一个延迟存储属性:```lazy var v = 0```

```
必须将延迟存储属性声明成变量(使用 var 关键字)
var类属性默认是延迟属性(?)
```
通过final关键字表示一个属性、方法或类是不可重写（或继承）的
例如：```final var，final func， final class```

#### 4.swift class的引用性质

```
struct Resolution {
    var width = 0
    var height = 0
}

class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}

let hd = Resolution(width:640, height: 480)
//hd.width = 2
let v1 = VideoMode()
v1.frameRate=0.3

let v2 = v1
v2.frameRate=0.8
print(v1.frameRate)//0.8
```
以上v2和v1都是一个VideoMode实例的引用，因此虽然v2和v1都是let的，但是依然可以修改```v2.frameRate=0.8```

#### 5.构造函数
swift class通过关键字init定义构造函数：

```
struct Color {     let red, green, blue: Double     init(red: Double, green: Double, blue: Double) {         self.red   = red         self.green = green         self.blue  = blue     }     init(white: Double) {         red   = white         green = white         blue  = white} }
```
