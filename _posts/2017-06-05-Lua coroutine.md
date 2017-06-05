
# Lua coroutine

协程的函数都放在全局的coroutine表里，_G.coroutine里一共5个函数：

![](/img/_postRes/coroutine.jpg)

具体用法（代码出自云风大神）：

```
function foo(a)
    print("foo", a)
    return coroutine.yield(2 * a)
end
co = coroutine.create(function ( a, b )
    print("co-body", a, b)
    local r = foo(a + 1)
    print("co-body", r)
    local r, s = coroutine.yield(a + b, a - b)
    print("co-body", r, s)
    return b, "end"
end)
print("main", coroutine.resume(co, 1, 10))
print("main", coroutine.resume(co, "er"))
print("main", coroutine.resume(co, "x", "y"))
print("main", coroutine.resume(co, "x", "y"))
```
--运行结果：

```
>lua -e "io.stdout:setvbuf 'no'" "sp.lua" 
co-body    1	10
foo	2
main	true	4
co-body	er
main	true	11	-9
co-body	x	y
main	true	10	end
main	false	cannot resume dead coroutine
>Exit code: 0
```
稍微解释下print("main", coroutine.resume(co, "er"))的运行结果，local r = foo(a + 1)里r的值为coroutine.yield(2 * a)，而coroutine的返回为resume的入参，即"er"，故r="er"
