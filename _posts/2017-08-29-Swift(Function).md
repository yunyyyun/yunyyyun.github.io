
### 函数


#### 函数定义

```
func someFunction(n1: Int, argumentLabel parameterName: Int, _ n3:Int, n4:Int = 4000)-> (Int,Int,Int){
    print(n1,parameterName,n3)
    return (n1,parameterName,n3)
}
someFunction(n1: 1, argumentLabel: 2, 3)
someFunction(n1: 1, argumentLabel: 2, 3, n4: 44)
```
以上为一个函数定义，其返回类型为三元组（Int, Int, Int），其中argumentLabel为参数标签，parameterName为参数名称，参数名称在函数实现里面使用，参数标签则在外部调用时使用，参数标签是可选的，其缺省值是参数名称。如果希望函数调用时能省去参数标签，可将参数标签用下划线（_）代替，参数也可以指定默认值,指定默认值的参数（上面的n4）是可选参数

注：参数名称和参数标签能够使你的代码更具可读性


#### 可变参数函数
可变参数函数是允许的：

```
func sum(nums:Int...)-> Int{ //可变参数
    var sum=0
    for n in nums
    {
        sum+=n
    }
    return sum
}
sum(nums: 1,3,2)

func sumOfArray(nums:[Int])-> Int{
    var sum=0
    for n in nums
    {
        sum+=n
    }
    return sum
}
sumOfArray(nums: [1,3,2])
```
注：一个函数最多只能拥有一个可变参数。


#### 输入输出参数
例如要写个swap函数：
```
func mySwap(a: inout Int, b: inout Int){
    a = a + b
    b = a - b
    a = a - b
}
var a=1
var b=2
mySwap(a: &a, b: &b)
```
如果你想要一个函数可以修改参数的值，并且想要在这些修改在函数调用结束后仍然存在，那 么就应该把这个参数定义为输入输出参数(In-Out Parameters)


#### 函数类型
在swift中函数类型就像其他一样，也是有类型的,例如：```func f1(){}```是 ```() -> Void```类型的函数，上例mySwap是```(inout Int, inout Int) -> Void```类型函数，someFunction是```(Int,Int,Int,Int)->(Int,Int,Int)```类型函数

函数是有类型的，可以定义一个类型为函数的常量或变量，并将适当的函数赋值给它:

```
func addTwoInts(_ a: Int, _ b: Int) -> Int {
    return a + b
}
var mathFunction: (Int, Int) -> Int = addTwoInts
mathFunction(2,10)
```
当然也能把函数当作函数的参数和返回值：

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
注：@escaping指逃逸闭包（如果一个闭包被作为一个参数传递给一个函数，并且在函数return之后才被唤起执行，那么这个闭包是逃逸闭包。）


#### 嵌套函数
定义在函数里的函数就是嵌套函数：

```
func chooseStepFunction(backward: Bool) -> (Int) -> Int {    func stepForward(input: Int) -> Int { return input + 1 }    func stepBackward(input: Int) -> Int { return input - 1 }
	return backward ? stepBackward : stepForward}
```
默认情况下，嵌套函数是对外界不可见的，但是可以被它们的外围函数(enclosing function)调用。一个外围 函数也可以返回它的某一个嵌套函数，使得这个函数可以在其他域中被使用。







