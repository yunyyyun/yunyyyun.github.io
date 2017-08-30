
### 闭包


#### 闭包表达式和语法
假设我们有如下姓名列表：

```
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
```
需要使用```sorted(by:)```方法对姓名排序，一般会这样写：

```
func backward(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}
var reversedNames = names.sorted(by: backward)
```
使用闭包，我们有更为简洁的写法：

```
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})
```
以上即为一个闭包的例子，闭包表达式语法有如下的一般形式:

```
{ (parameters) -> returnType in     statements}
```
闭包的函数体部分由关键字 in 引入。该关键字表示闭包的参数和返回值类型定义已经完成，闭包函数体即将开始。

闭包是支持类型推导的，上面的例子甚至可以简化为：

```
reversedNames = names.sorted(by: {s1, s2 in return s1 > s2 } )
```
单行表达式闭包可以通过省略 return 关键字来隐式返回单行表达式的结果，如上面例子可以改写为:

```
reversedNames = names.sorted(by: { s1, s2 in s1 > s2 } )
```
Swift 自动为内联闭包提供了参数名称缩写功能，你可以直接通过 $0 ，$1 ，$2 来顺序调用闭包的参数，以此类推。上面例子可以改写为:

```
reversedNames = names.sorted(by: { $0 > $1 } )
```
最简单的写法：

```
reversedNames = names.sorted(by: >)
```
因为Swift 的 String 类型定义了关于大于 号(>)的字符串实现，其作为一个函数接受两个 String 类型的参数并返回 Bool 类型的值。而这正好与```sorted(by:)``` 方法的参数需要的函数类型相符合。因此，你可以简单地传递一个大于号，Swift 可以自动推断出 你想使用大于号的字符串函数实现。

如果你需要将一个很长的闭包表达式作为最后一个参数传递给函数，可以使用尾随闭包来增强函数的可读性。尾随闭包是一个书写在函数括号之后的闭包表达式，函数支持将其作为最后一个参数调用。在使用尾随闭包时，你不用写出它的参数标签,如此，上面例子可以改写为:

```
reversedNames = names.sorted() { $0 > $1 }
```
如果闭包表达式是函数或方法的唯一参数，则当你使用尾随闭包时，你甚至可以把 () 省略掉:

```
reversedNames = names.sorted { $0 > $1 }
```
一个例子，使用```map(_:)``` 方法，修改数组：

```
let digitNames = [
    0: "零", 1: "壹", 2: "贰",   3: "叁", 4: "肆",
    5: "伍", 6: "陆", 7: "柒", 8: "捌", 9: "玖"
]
let numbers = [16, 58, 510]
let strings = numbers.map {  //["壹陆", "伍捌", "伍壹零"]
    (number) -> String in
    var number = number
    var output = ""
    repeat {
        output = digitNames[number % 10]! + output
        number /= 10
    } while number > 0
    return output
}
```

#### 值捕获

```
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}
makeIncrementer(forIncrement: 2)() //2
makeIncrementer(forIncrement: 2)() //2
let incrementByTwo = makeIncrementer(forIncrement: 2)
incrementByTwo() //2
incrementByTwo() //4
incrementByTwo() //6
```
```incrementer()```函数并没有任何参数，但是在函数体内访问了```runningTotal```和 amount 变量。这是因为它从外围函数捕获了```runningTotal```和```amount```变量的引用。捕获引用保证了```runningTotal```和```amount```变量在调用完```makeIncrementer```后不会消失，并且保证了在下一次执行 ```incrementer```函数时，```runningTotal```依旧存在。

```
注意: 如果你将闭包赋值给一个类实例的属性，并且该闭包通过访问该实例或其成员而捕获了该实例，你将在闭包和该 实例间创建一个循环强引用。
```
与```String```等不同，闭包是引用类型，无论你将函数或闭包赋值给一个常量还是变量，你实际上都是将常量或变量的值设置为对应函数或闭包的引用。例如上面的```incrementByTwo```，只是一个指向闭包的引用常量，而非闭包本身。
这也意味着如果你将闭包赋值给了两个不同的常量或变量，两个值都会指向同一个闭包:

```
//（依赖上面例子的代码）
let alsoIncrementByTwo = incrementByTwo
alsoIncrementByTwo() //8
```


#### 逃逸闭包
当一个闭包作为参数传到一个函数中，但是这个闭包在函数返回之后才被执行，我们称该闭包从函数中逃逸。当 你定义接受闭包作为参数的函数时，你可以在参数名之前标注 @escaping ，用来指明这个闭包是允许“逃逸”出 这个函数的。
例如，上一篇的例子：

```
func mathFunc(f: @escaping ((Int)->Int), type: Int = 0)->(Int)->Int{
    func defalutAddOne(a:Int)->Int{
        return (a+1)
    }
    if type==0{
        return f
    }
    else{
        return defalutAddOne
    }
}
func addOne(a:Int)->Int{
    return (a+100)
}
print(mathFunc(f: addOne, type: 0)(1))
print(mathFunc(f: addOne, type: 1)(1))
```


#### 自动闭包与延迟求值

自动闭包是一种自动创建的闭包，用于包装传递给函数作为参数的表达式。这种闭包不接受任何参数，当它被调用的时候，会返回被包装在其中的表达式的值。这种便利语法让你能够省略闭包的花括号，用一个普通的表达式来代替显式的闭包。
自动闭包让你能够延迟求值，因为直到你调用这个闭包，代码段才会被执行。延迟求值对于那些有副作用(Side Effect)和高计算成本的代码来说是很有益处的，因为它使得你能控制代码的执行时机。下面的代码展示了闭包如何延时求值：

```
var arr = [1,2,3,4,5]
print(arr.count) //5
let doSthInArr = { arr.remove(at: 0) } //(()) -> Int
print(arr.count) //5
print("Now do it:\(doSthInArr())!")
print(arr.count) //4
```
将闭包作为参数传递给函数时，你能获得同样的延时求值效果：

```
arr = [1,2,3,4,5]
func serve(customer doSthInArr: () -> Int) {
    print("Now do it: \(doSthInArr())!")
}
serve(customer: { arr.remove(at: 0) } )
serve(customer: { arr.remove(at: 0) } )
```

```
注意 过度使用 autoclosures 会让你的代码变得难以理解。上下文和函数名应该能够清晰地表明求值是被延迟 执行的。
```





