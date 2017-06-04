
# lua入门学习（猜数字游戏）
## 代码如下：
```
print('--------------------------by yun------------------------------')
print('-------注意：本游戏全程只接受纯数字输入，其它皆不合法--------')
i=1
repeat
	print(i,'请输入最大的数:')
	max=io.read('*number')
	min=1
	assert(type(max)=='number')
		print('游戏开始...')
		math.randomseed(tostring(os.time()):reverse():sub(1, 6))
		rnumber=math.random(min,max)
		repeat
			print('输入您猜的数字:')
			currentnumber=io.read('*number')
			assert(type(currentnumber)=='number')
			if currentnumber>rnumber then
				max=math.min(currentnumber,max+1)-1
				print('您猜的太大了，请在如下闭区间再猜：')
				print ("[",min,",",max,"]" )
			elseif currentnumber<rnumber then
				min=math.max(currentnumber,min-1)+1
				print('您猜的太小了，请在如下闭区间再猜：')
				print ("[",min,",",max,"]" )
			end
		until currentnumber==rnumber
		print('恭喜您猜对了!')
	print('要退出吗？(输入0退出否则继续)')
	q=io.read('*number')
	assert(type(max)=='number')
	i=i+1
until q==0
print('---------------------游戏结束，谢谢！----------------------')
```
## 简单描述：
1.  lua没有continue，具体原因后面研究
2.  io.read('*number')，从控制台读入纯数字，必须注意的是，如果用户输入带了非数字字符串，那么将会从头截取数字读入，恶心的是后面的字符串会遗留，并可能在下次io.read()时读入
例如：
a=io.read('*number')
s=io.read()
print(a)
print(s)
运行后，控制台输入123a123能看到
123
a123
的输出结果。