
### String


#### String初始化

```
var emptyString = "" 
var anotherEmptyString = String()
```
以上两个字符串都是空串"" :

```
if emptyString.isEmpty {    print("Nothing to see here") // 打印输出:"Nothing to see here"}
```
其他字符串初始化和赋值：

```
let characters = "321"
let str2 = String(characters)

let intValue = 4
let str3 = "123\(intValue)"  //字符串插值
```


#### String连接
String+String :

```
var foo = "foo"
foo += " bar"
```
String+Character :

```
foo.append("!")
```
注：只有var String才能可变


#### 字符串长度
字符长度：

```
let str:String="123abc啊哦咦！"
print(str.characters.count) //10
```
存储长度：

```
print(str.lengthOfBytes(using: .unicode))//20
print(str.lengthOfBytes(using: .utf8))   //18
```
存储长度因不同编码，汉字等字符所占字节不同而不同：

```
var word = "cafe"
print(word.characters.count)//4
print(word.lengthOfBytes(using: .unicode))//8
print(word.lengthOfBytes(using: .utf8))   //4
word += "\u{301}"
print(word.characters.count)//4
print(word.lengthOfBytes(using: .unicode))//10
print(word.lengthOfBytes(using: .utf8))   //6
```


#### 访问和修改字符串
每一个String值都有一个关联的索引(index)类型,它对应着字符串中的每一个字符的位置。使用index可获取每个字符：

```
let greeting = "Guten Tag!"
greeting[greeting.startIndex]// G
greeting[greeting.index(before: greeting.endIndex)]// !
greeting[greeting.index(after: greeting.startIndex)]// u
let index = greeting.index(greeting.startIndex, offsetBy: 7)
greeting[index]// a
```


#### 字符串遍历

```
let str:String="123qwe！"
for c in str.characters{
    print(c)
}

for index in str.characters.indices {
    print("\(str[index]) ", terminator: "")
}

for i in 0..<str.characters.count{
    let index = str.index(greeting.startIndex, offsetBy: i)
    print(str[index])
}
```
通过index方法遍历String不是一个好的方法


#### 插入和删除
插入:

```
var welcome = "hello"
welcome.insert("!", at: welcome.endIndex)
welcome.insert(contentsOf:" there".characters, at: welcome.index(before: welcome.endIndex))
```
删除：

```
welcome.remove(at: welcome.index(before: welcome.endIndex)) 
let range = welcome.index(welcome.endIndex, offsetBy: -6)..<welcome.endIndex
welcome.removeSubrange(range)
```


#### 字符串比较：

```
let caf = "caf"
let cafe = caf+"e"
let cafe1 = caf+"\u{65}"+"\u{301}"
let cafe2 = caf+"\u{E9}"
if cafe1 == cafe2 {
    print("cafe1 and cafe2 is equal")
}
if cafe == cafe2 {
    print("cafe and cafe2 is equal")
}
```
注： (U+00E9)和(U+0065)+(U+0301)都是é 的有效表现方式


#### 附上leetCode第三题(最大连续不重复子串)的解：

```
class Solution {
    func lengthOfLongestSubstring(_ s: String) -> Int {
        if s.isEmpty{
            return 0
        }
        var maxLen = 1
        var currentLen = 1
        var indexTable:[Character:Int]=[:]
		let chars = [Character](s.characters)
		let sCount = chars.count
        var startI = 0
        var startJ = 1
        while(startJ<sCount){
            indexTable[chars[startI]] = startI
            currentLen = startJ-startI
            for j in startJ..<sCount{
                let pos = indexTable[chars[j]]
                if (pos != nil)&&(pos! >= startI){
                    maxLen = maxLen>currentLen ? maxLen:currentLen
                    startJ = j+1
                    startI = indexTable[chars[j]]!+1
                    indexTable[chars[j]] = j
                    break
                }
                else{
                    indexTable[chars[j]] = j
                    currentLen+=1
                    if j+1 == sCount{
                        maxLen = maxLen>currentLen ? maxLen:currentLen
                        return maxLen
                    }
                }
            }
        }
        return maxLen
    }
}
```








