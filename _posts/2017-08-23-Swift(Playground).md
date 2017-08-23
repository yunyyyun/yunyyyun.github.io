
### 什么是Playground？

它可以实时执行 swift 代码，并将结果显示出来，是苹果为了推广swift，让swift更易学而推出的工具

创建palyground非常简单，打开xcode就能看到“Get started with a playground”,创建之后，随意键入swift代码即可在右边实时看到显示结果

例如：
键入代码：

```
let individualScores = [75, 43, 103, 87, 12]
var teamScore = 0
for score in individualScores {
    if score > 50 {
        teamScore += 3
    } else {
        teamScore += 1
    }
}
```

右边则看到

```
[75, 43, 103, 87, 12]
0


(3 times)

(2 times)


```

不仅将结果实时展示，将循环命中数显示，甚至连行数也对上，太方便了，另外右边的数据也能点击查看详细信息

### Sources && Resources

在playground界面，使用 Cmd + 1 打开项目导航栏，这时可以看到Sources 和 Resources两个文件夹

其中Resources目录用于存放资源文件
Sources目录下的源文件则会被编译成模块(module)并自动导入到 Playground 中，并且这个编译只会进行一次(或者我们对该目录下的文件进行修改的时候)，而非每次你敲入一个字母的时候就编译一次（playground文件里的代码会实时编译出结果，当代码太多会影响执行效率）。 
另外，由于此目录下的文件都是被编译成模块导入的，只有被设置成 <font color=red>public</font> 的类型，属性或方法才能在 Playground 中使用。

例如：
在Sources新建func.swift,键入如下代码

```
public func makeIncrementer() -> ((Int) -> Int) {
    func addOne(number: Int) -> Int {
        return 1 + number
    }
    return addOne
}
```

则可以直接在playground文件里面调用：

```
makeIncrementer()(9)
```

未完待续～





