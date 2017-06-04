unpack函数：

```
function sum(...)               --> 函数实现求和，(..)表示参数个数不确定
	local s=0                    -->local修饰局部变量，效率比全局高
	for i,v in ipairs{...}do
		s=s+v
	end
	return s
end
arr2={1,2,3,4,5,6,7,8,9,10}             -->table 
print(sum(12,21),sum(unpack(arr2)))     -->33,55
```
总结：
unpack函数,接受数组作为参数，返回下标1开始的所有元素，可实现泛型调用

```
f=string.find
a={'hello lua','lua'}
sstart,eend=f(unpack(a))
print(sstart,eend)-->7,9
```

另一段有意思的代码：

```
function copy(...)
	return ...
end
a,b,c,d=copy(1,2,3)
print(a,b,c,d)                          -->1,2,3,nil
```

