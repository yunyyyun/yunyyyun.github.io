
### 可选类型 Optional

Swift中的Optional作为一种类型，既可以存储一个值，也可以为空（nil）,通常在类型后面加一个？表示它是Optional类型的：

```
var number： Int? = 32
```
其中？是语法糖，上面写法等价于：

```
var number: Optional<Int> = 32
```
打开Optional定义可以看到其枚举定义：

```
public enum Optional<Wrapped> : ExpressibleByNilLiteral {
    case none
    case some(Wrapped)
    ...
}
```
一个Optional对象只存在两种状态：包含一个值，或者为空，我们都可以通过解包（unwrap）来获取:

```
var myString: String? = "Hello"
if myString != nil{
    //print("\(myString)")
    print("\(myString!)")
}
```

### 安全使用Optional

我们不能再把Optional当作Boolean值处理:

```
var myString: String? = "Hello"
if myString {
    print(myString)
}
```
可喜的是编译器会辅助我们修正错误：

```
if myString != nil{
    print("myString contain a string value of \(myString!)")
}
```
实际上，Swift提供了一种更加方便的形式来完成这一过程：

```
if let actualString = myString {
    print("myString contain a string value of \(actualString )")
} else {
    print("myString is nil")
}
```
注：也可用```if var actualString = myString ``` ，此时则actualString为可变量

### 隐式解包Optional、！

Swift中我们还有一种特殊的Optional，在对它的成员或者方法进行访问时，编译器会自动进行解包，被称为隐式解包：

```
var possisbleString: String!
```
在访问时，编译器会自动帮我们完成在变量后插入!的行为

```
let possibleString: String! = "An implicity unwrapped optional string."
let implicitString: String = possibleString //此处我们不需要!来对possibleString 进行显示解包
```
使用隐式解包（！）有个风险：尝试访问一个为空的隐式解包Optional, 就会遇到一个runtime error，因此保证隐式解包的类型不为空是必要的。

### Optional Chaining

可以通过一个链来安全的访问一个Optional的属性或者方法：

```
if (leftViewController?.view.superview) != nil){
	...
}
```
当leftViewController为空时，直接返回nil，Optional Chaining帮助我们简化了代码，使我们不必一层层判断非nil

### ??的使用

```
var s1: String?
let s2 = s1 ?? "default"
```
同样语法糖，等价于：

```
if s1 != nil{
    s2=s1
}
else{
	s2="default"
}
```

### 其它（思考）

```
let s1: String?
let s2: String?=nil
var s3: String?
var s4: String?=nil
```
以上4种写法区别？

### 附上leetCode第二题答案

```
class Solution {
    func addTwoNumbers(_ l1: ListNode?, _ l2: ListNode?) -> ListNode? {
        if l1 == nil {
            return l2
        }
        if l2 == nil {
            return l1
        }
        var p1 = l1;
        var p2 = l2;
        var l3:ListNode?;
        var p3:ListNode?;
        var flag = 0;
        var n1 = 0;
        var n2 = 0;
        var tmp = 0;
        while (p1 != nil || p2 != nil){
            n1 = (p1==nil) ?0: p1!.val
            n2 = (p2==nil) ?0: p2!.val
            tmp = n1+n2+flag
            if tmp>=10 {
                flag=1
                tmp = tmp-10
            }
            else{
                flag=0
            }
            if p3==nil {
                l3 = ListNode(tmp)
                p3 = l3
            }
            else{
                p3!.next = ListNode(tmp)
                p3 = p3!.next!
            }
            print(3)
            p1 = (p1==nil) ?p1:p1?.next
            p2 = (p2==nil) ?p2:p2?.next
        }
        if flag==1 {
            p3!.next = ListNode(1)
        }
        return l3
    }
}
```








