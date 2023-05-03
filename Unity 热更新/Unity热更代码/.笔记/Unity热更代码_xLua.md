[TOC]



# Lua 技术点梳理

## table

是特殊的容器，既可以像数组一样用索引去存储，也可以像字典一样用Key和Value去存储

变量可以无中生有

### 基本原理

```lua
--table 的索引存储

local mytable = {1,2,3}
--索引从1开始
print(mytable[1])
print(mytable[2])
print(mytable[3])

--lua赋值方式2
--第1个索引位值是1...
local mytable = {[1]=1, [2]=2, [3]=3}
print(mytable[1])
print(mytable[2])
print(mytable[3])

--lua赋值方式3
local mytable = {1,2,3, [3]=4}
print(mytable[1])
print(mytable[2])
print(mytable[3])
--输出结果 1 2 3
--table 的机制
--1、先找 指定索引位并赋值的 所以索引一开始[3]=4，即索引3=值4
--2、再找隐式指定索引的 所以 索引3被改写为值=3


--table 的键值对存储 -- 字典
local mytable = {1,2,5, a=9, b="bo"}
print(mytable[1])
print(mytable[2])
print(mytable[3])
print(mytable[4])
print(mytable[5])
print(mytable.a)
print(mytable.b)
--输出结果 1 2 5 nil nil 9 bo

--变量无中生有
local mytable = {1,2,5, a=9, b="bo"}
mytable.id = "1001"
mytable.userdata = { atk = 15, def = 20 }
print(mytable.id)
print(mytable.userdata.atk)
--输出结果 1001 15

--遍历打印table中所有的值
local mytable = {11,12,15, age=9, name="bo"}
mytable.id = "1001"
mytable.userdata = { atk = 15, def = 20 }
function mytable:test(p)
	print(p) 
    print(mytable[1])
end

--遍历table
for i,v in pairs(mytable) do
    --i=key v=value
    print(i)
    print(v)
end
--输出结果
[[
1
11
2
12
3
15
name
bo
age
9
id
1001
userdata
table: 0x560300c18ea0
test
function: 0x560300c18ee0

]]

--nil 也会占位
local mytable = {11,12,15, nil, 7}
print(mytable[5])
--打印结果 第5位=7

```



### 非空判断

```lua
local mytable
print(mytable)
if(mytable == nil) then
    print(" 为空")
end

--打印结果 
--[[
nil
为空
]]

local mytable={}
print(mytable)
if(mytable == nil) then
    print(" 为空")
end
--打印结果
--table:0x560068532c330

local mytable={}
print(mytable)
if(mytable == {}) then
    --不会进来 {} 在这里实时的生成了一个匿名table
    --table的地址肯定不一样 进不来
    print(" 为空")
end
--打印结果
--table:0x560068532c330


local mytable={}
print(mytable)
if( next(mytable) == nil) then
    --使用next 会获取表中的下一个内容 
    --判断会进来
    --next(table,索引值) 检查是否有值 默认索引值不传=1
    print(" 为空")
end
--打印结果
--[[
table:0x560068532c330
 为空
]]
```



### 获取元素长度

#号 和 getn 需要确定 table里 没有 nil 并且 都是按照数组索引存储的方式

```lua
local mytable = {1,2,3,4,5}
--#
print(#mytable)
--table.getn
print(table.getn(mytable))
--输出结果 5

local mytable = {1,2,3,4,5, a=5, 5}
--#和getn 只能找到按照数组方式存储的个数 无法找到keyValue
print(#mytable)
--输出结果 5

local mytable = {1,nil, 2, nil}
--#和getn 遇到nil后就不会往后继续检查数量
print(#mytable)
--输出结果 1

```



### 遍历

#### pairs

是最好的遍历

#### ipairs

功能有限 只能遍历数组的方式，不能遍历空值（会结果有误），也不能遍历键值对（结果值也会有误）

```lua
local mytable = {1,nil, 2, nil, 3}
--#和getn 甚至遇到多个nil后 长度的结果会混乱
print(#mytable)
--输出结果 3

--如果结果里有很多nil和键值对 需要用 pairs 遍历统计长度 不能用ipairs
local mytable = {1,nil, 2, nil, 3}
print("pairs遍历")
for i,v in pairs(mytable) do
	print(mytable[i])
end
print("ipairs遍历")
for i,v in ipairs(mytable) do
	print(mytable[i])
end
--pairs 和 ipairs 的区别在于 ipairs 遇到 nil 后 就不会遍历
--输出结果
--[[
pairs遍历
1
2
3
ipairs遍历
1

]]


local mytable = {1, 2, a=1, b="b", 3, 4}
print("pairs遍历")
for i,v in pairs(mytable) do
	print(mytable[i])
end
print("ipairs遍历")
for i,v in ipairs(mytable) do
	print(mytable[i])
end
--输出结果
--[[
pairs遍历
1
2
3
4
b
1
ipairs遍历
1
2
3
4

]]


--ipairs遍历到第2项得到为空就停止了
local mytable = { [1]=1, [3]=3, [4]=4}
print("pairs遍历")
for i,v in pairs(mytable) do
	print(mytable[i])
end
print("ipairs遍历")
for i,v in ipairs(mytable) do
	print(mytable[i])
end
--输出结果
--[[
pairs遍历
1
3
4
ipairs遍历
1
]]


```





## 元表

元表和面向对象

```lua
local mytable = { 1, 2, 3 }
local mymetatable = {}

--生成具有元表的新表 继承父类的子类
mytable = setmetatable(mytable, mymetatable)
--得到元表 即 mymetatable
getmetatable(mytable)

--直接生成
mytable = setmetatable({1,2,3}, {})

```



#### 设置与获取 

setmetatable getmetatable



## 元方法

### __index 

可以实现面向对象

当访问设置过元表的表的属性的时候，如果属性存在于当前表里面，就会调用这个属性，如果不存在这个属性，并且有元表实现了 __index 的方法，并且 这个 index 方法有两种形式，

一种是以 table 方式存在，

​	如果是以 table 的方式存在，就会去 table 中找属性存在，找不到就为nil



```lua
local mytable = { 1, 2, 3 }
local mymetatable = {
    __index = { b = 6, c = "c" }
}

--这里还没有设置元表 输出结果为 nil
print(mytable.b)
mytable = setmetatable(mytable, mymetatable)
getmetatable(mytable)
--设置了元表后 虽然表没有 但是表的元表方法__index 有
print(mytable.b)

--输出结果
--[[
nil
6

]]

```



另一种是以方法的方式存在，

​	如果是方法方式存在，如果没有这个方法，就会调用元表里的方法



```lua
local mytable = { 1, 2, 3 }
local mymetatable = {
    __index = function(table, key)
        	if key == "b" then
            	return "hello"
            else 
            	return nil
            end
        end
}

mytable = setmetatable(mytable, mymetatable)
print(mytable.b)

--输出结果
--[[
hello

]]

```



### __newindex

当一个新值被找到的时候

当给不存在的元素赋值的时候（表和元表都没有），如果有这个 __newindex 方法 就会调用，否则会赋值

可以用来做约束，比如不允许添加某个字段赋值

```lua
local mytable = { 1, 2, 3 }
local mymetatable = {
    __index = function(table, key)
        	if key == "b" then
            	return "hello"
            else 
            	return nil
            end
    end
    ,
    __newindex = function(table, key)
        print("__newindex 被调用" .. key)
    end
}

mytable = setmetatable(mytable, mymetatable)
--注意是赋值的时候调用 __newindex
mytable.c = 5

--输出结果
--[[
__newindex 被调用 c

]]

```



### __add

用在两个表相加，如果没有指定 __add 的情况下 两个表相加会报错

```lua
-- 计算表中最大值
function table_maxn(t)
    local mn = 0
    for k, v in pairs(t) do
        if mn < k then
            mn = k
        end
    end
    return mn
end

-- 设置元表
-- 两表相加操作 __add 两个表相加时候触发
mytable = setmetatable({ 1, 2, 3 }, {
  __add = function(mytable, newtable)
    for i = 1, 4 do
        print("table_maxn(mytable)",table_maxn(mytable),newtable[i])
      table.insert(mytable, table_maxn(mytable)+1,newtable[i])
    end
    return mytable
  end
})

secondtable = {3,4,5,6}

mytable = mytable + secondtable
for k,v in ipairs(mytable) do
    print(k,v)
end

-- 输出结果
--[[

table_maxn(mytable)	3	3
table_maxn(mytable)	4	4
table_maxn(mytable)	5	5
table_maxn(mytable)	6	6
1	1
2	2
3	3
4	3
5	4
6	5
7	6


]]
```



### __call

在lua调用一个值的时候会被调用

如果对当前的表作为一个方法，传入一个参数的时候就会调用

（注意要和__call的类型一致 -- 这里传入的是表）

mytable(newtable)

```lua
local mytable = { 1, 2, 3 }
local mymetatable = {
    __index = function(table, key)
        	if key == "b" then
            	return "hello"
            else 
            	return nil
            end
    end
    ,
    __newindex = function(table, key)
        print("__newindex 被调用" .. key)
    end
    ,
    __call = function(mytable, newtable)
        --会把两个表的所有的数值加到一起
        sum = 0
        for i= 1, #mytable do
            sum = sum + mytable[i]
       	end
        for i= 1, #newtable do
            sum = sum + newtable[i]
       	end
        
        return  sum
        
    end
}

mytable = setmetatable(mytable, mymetatable)
newtable = {10, 20, 30}
print(mytable(newtable))

--输出结果
--[[
66

]]

```



### __tostring

用在 打印表的时候调用，可以用于展示表的内部

```lua
local mytable = { 1, 2, 3 }
local mymetatable = {
    __index = function(table, key)
        	if key == "b" then
            	return "hello"
            else 
            	return nil
            end
    end
    ,
    __newindex = function(table, key)
        print("__newindex 被调用" .. key)
    end
    ,
    __call = function(mytable, newtable)
        --会把两个表的所有的数值加到一起
        sum = 0
        for i= 1, #mytable do
            sum = sum + mytable[i]
       	end
        for i= 1, #newtable do
            sum = sum + newtable[i]
       	end
        
        return  sum
        
    end
    ,
    __tostring = function(mytable)
    	--打印表的时候调用 这里打印长度
        print(#mytable)
        return #mytable
    end
}

mytable = setmetatable(mytable, mymetatable)
--当作变量打印
print(mytable)

--输出结果
--[[
3
3

]]
```



## 点调用和冒号调用

冒号的方式其实是一种语法糖，默认包含了mytable

点调用需要手动把自己传进去



当在Lua中调用一个静态成员的静态方法的时候，不需要把自身传入进去(self) 

​		如： Input.GetButtonDown()

当在Lua中调用一个成员函数的普通方法的时候，需要把自身传入进去

​		如： bullectAudio.Player(self)   bullectAudio:Player()



针对lua本身而言，需要传递self调用的时候使用冒号。

上面这句话对题目意义不大，那是标准lua下的情况。

而对于题中的Unity里xlua和C#互调用的情况，萌新只需要记住：

1. lua在访问**C#对象**中的**非静态***方法*时，使用冒号
2. lua在访问lua class 对象中的方法时，使用冒号

其他情况下全部用点号来访问就行。



```lua
mytable = setmetatable({1,2},{})

function mytable:test(p)
    --冒号的方式其实是一种语法糖，默认包含了mytable
    print(p)
    print(mytable[1])
end

mytable:test(5)
--点调用 需要显示的将 table 传进去
mytable.test(mytable, 5)
--mytable.test(5) 这样调用会报错

--输出结果
--[[
5
1
5
1

]]

```



## 闭包 - 重要

function 方法函数

```lua
function test1(n)
   	if n == 0 then 
        return 1
    else
        return n*test1(n - 1)
    end
end

--3*2*1
print(test1(3))
--打印结果 6

--方法赋值
test2 = test1
--打印结果 6
print(test2(3))
```

```lua
function test(tab, fun)
    for k,v in pairs(tab) do
        print(fun(k,v))
    end
end

--匿名方法
tab = {key1 = "v1", key2 = "v2"}
test(tab, function(key, val)
    			return key .. "=" .. val
    		end
)
 
--输出结果 
--[[
key2=v2
key1=v1

]]

```

function 可以视为变量，可以作为参数，允许匿名定义

function  执行的时候才会创建，没有执行的时候不会创建：允许运行期创建 第一类值

闭包：lua在编译一个函数的时候，会生成一个原型 prototype，其中包含了函数体对应的虚拟机指令，函数体中的常量，和一些调试信息。运行的时候，当lua执行 function<body> end 的时候，会创建一个新的数据对象，对象中包含了响应函数原型的引用以及一个数组（包含了所有upValue的引用），这个数据对象，叫做闭包。

```lua
function foo(x)
    print(x)
end

foo2 = function(x) 
    		print(x)
    	end
foo(2)
foo2(2)
--[[
2
2
]]

--方法的构造式
b = function(param)<body> end
a  = {}

--语法域 词法定界
--一个函数可以嵌套在另一个函数中，内部函数可以访问外部函数的局部变量
function f1(n)
    --n的作用范围
    --n 是闭包g1 g2 的upvalue，而是非全局变量，既不是g1 g2的临时变量，也不是全局变量
    local  function f2()
        --可以访问到f1的变量
        print(n)
    end
    --为了做闭包 需要return 出去
    return f2
end
--通过f1制作出了g1(闭包) 这个闭包是一个嵌套方法 嵌套方法里面的内部方法可以访问到外部方法的传递的变量
--同时变量在生成g1闭包的时候，变量会随着整个方法定义里的指令和局部参数都会作为变量缓存起来（g1不包含任何参数就可以执行）
g1 = f1(2021)
g1()
--输出 2021
g2 = f1(2022)
g2()
--输出 2022


function create(n)
    --同样的方法体内两个局部方法产生的闭包实际上是共享了同一个upValue（即 n） -- 传了引用地址而不是值
    local function foo1()
        print(n)
    end
    local function foo2()
		n = n + 10
    end
    --lua支持多返回值
    return foo1, foo2
end

--做出两个闭包方法 f1,f2共享了闭包里的n（即 2021）
f1,f2 = create(2021)
--f3,f4共享了闭包里的n（即 1990）
f3,f4 = create(1990)
--注意共享了upValue
f1()
f2()
f1()
--[[
2021
2031
]]
f3()
f4()
f3()
--[[
1990
2000
]]



function create(n)
    local function foo()
        local function foo1()
            print(n)
        end
        local function foo2()
            n = n + 10
        end
        return foo1, foo2
    end
    return foo
end

--生成foo闭包
f0 = create(2021)
--生成了4个闭包，4个闭包共享了upValue(即 n值)
f1,f2 = f0()
g1,g2 = f0()

f1()
f2()
f1()
g1()
g2()
g1()
--[[
2021
2031
2031
2041

]]
```



#### 闭包的作用

1. 高阶函数的参数 table.sort(t, function(t1, t2) return t1.param > t2.param end )

2. 重写（类似于 C# 面向对象的重写）

   io.open 可以打开一个文件，希望每次打开的时候有个验证，不去使用原生的 io 的 open 的方法

   希望加入自己的逻辑不去改动源代码

   使用闭包就可以重写解决

   ```lua
   local oldOpen = op.open
   
   local accessOk = functio(filename, mode)
   	--自己逻辑的 检查文件 校验文件方法
   end
   
   --闭包方法 
   io.open = function(filename, mode)
   	if accessOk(filename, mode) then 
   		return oldOpen(filename, mode)
   	else
   		return null, "校验失败"
   	end
   end
   ```

3. 实现迭代器

   协程的本质是迭代器

   迭代器每次执行的时候都会有变量控制执行到哪里，执行过的部分不执行，会去执行后面没有执行过的

   高阶的迭代器 - 帧延迟，时间延迟，挂起足够时间再去往下执行

   闭包可以实现迭代器

```lua
function values(t)
		local i = 0
		--每次自增
		return function() i = i + 1 return t[i] end
	end

	--使用迭代器
	t = {10, 20, 30, 40, 50, 60}

	--闭包 共享 upValue	
	iter = values(t)

	print(iter())
	print(iter())
	print(iter())
	print(iter())
	--[[
		10
		20
		30
		40
	]]
```





## lua的数据类型

1. table
2. function
3. nil
4. boolean
5. number 既可以表示整数也可以表示浮点数
6. string
7. userdata C语言有些类型在Lua中不存在
8. thread 线程



Lua中的变量可以定义任意类型

```lua
local a = 5
-- 打印number
print(type(a))
a = “1”
-- 打印 string
print(type(a))

```



## C# 和lua的相互调用

1. Lua 语言本身是用 C 语言开发的

2. Lua 语言 和 C语言本身可以相互直接调用，提供了一系列的调用接口，让C与Lua相互调用

3. C# 和 Lua的调用 = C# 调用C，C语言调用Lua

   xLua 插件，封装了一层 C# 调用 C 语言的接口

   （用C语言作为桥梁实现语言间相互调用）

   

   lua与C交互：基于栈操作，

   lua调用C函数时，需要写个封装函数，从栈上取出调用参数，调用C函数后把结果放到栈上；

   C要调用lua函数，也把参数放到栈上，用luaAPI完成调用后，从栈上取出结果。

   

   Lua 的虚拟栈 是让 C 语言 和 Lua 相互调用 

   栈：先进后出

   Lua的虚拟栈去 索引栈的元素（两套索引机制找值）

   ​	从栈的底 往上索引 最底下是1，往上是2、3、4...n

   ​	从栈的顶 往下索引 最顶是-1，往下是-2，-3，-4

   

   Lua虚拟机一次运行期间创建一个lua_state。每个lua_state 都包含自己的 Lua虚拟栈

   每个lua_state 是相互独立的，有自己的虚拟栈

   

   Lua 的数据类型都可以压栈出栈，Lua 给每一个数据类型的入栈提供了单独的C语言的方法（闭包类型的压栈 和 table类型的压栈 都有自己的方法调用）



### Lua 与 C 的通信

Lua 语言本身是C语言开发，自带 C/C++ 通信机制

**Lua和C/C++的数据交互：通过栈**

​	栈 相当于数据在 Lua 和 C/C++ 之间的中转地

​	每种数据类型都有对应的存取接口 

操作数据：
 	1. 将数据拷贝到“栈”上
 	2. 栈中的每个数据通过索引值定位，获取数据
     	1. 索引值为正 = 相对于栈底的偏移索引
     	2. 索引值为负 = 相对于栈顶的偏移索引
     	3. 1 或 -1 作为起始值。栈顶索引值=-1，栈底索引值=1



### C# 与 C 的通信

P/Invoke 方式 调用 Lua 的 dll

Lua 的 dll 执行 Lua 的 C 的 API

即：C# 借助 C/C++ 实现与 Lua 进行数据通信（xLua 的 LuaDLL.cs 文件）

**xLua 的 LuaDLL.cs** 文件 的 DllImport 修饰的数据入栈 与 获取 的接口

```c#
// LuaDLL.cs
[DllImport(LUADLL,CallingConvention=CallingConvention.Cdecl)]
public static extern void lua_pushnumber(IntPtr L, double number);

[DllImport(LUADLL,CallingConvention=CallingConvention.Cdecl)]
public static extern void lua_pushboolean(IntPtr L, bool value);

[DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]
public static extern void xlua_pushinteger(IntPtr L, int value);

[DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]
public static extern double lua_tonumber(IntPtr L, int index);

[DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]
public static extern int xlua_tointeger(IntPtr L, int index);

[DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]
public static extern uint xlua_touint(IntPtr L, int index);

[DllImport(LUADLL,CallingConvention=CallingConvention.Cdecl)]
public static extern bool lua_toboolean(IntPtr L, int index);
```



### 1. Lua与C#数据通信机制（Lua 调用 C#）



#### 传递C# 对象到 Lua

1. bool、int 简单的值类型 直接通过 C 的API 传递
2. Lua 没有 C# 对象 对应类型，传递到 Lua 的只能是 C# 对象的一个索引
   1. ObjectTranslator.cs 中的 Push 方法
   2. Push 方法中第一次传递对象会调用 addObject 将对象缓存到 对象池 并返回一个索引（索引值可以获取到这个对象）
   3. 得到索引后，Lua 通过 xlua_pushcsobj 将代表对象的索引传递到 Lua，在 xlua_pushcsobj  中执行了下面的逻辑：
      1. Lua 为代表对象的索引创建一个 userdata，并指向对象索引
      2. 如果需要缓存 则将 userdata（即 对象索引） 保存到缓存表中
      3. 为 userdata 设置元表

**C# 对象 在 Lua 中对应的是一个 userdata，利用对象索引保持了与 C# 对象的联系**



##### 注册 C# 类型信息 到 Lua 

复杂类型转换为userdata（索引）后，为 userdata 设置的元表（元表是通过getTypeId函数生成的），表示的是对象的类型信息（比如对象类型有哪些成员方法，属性或是静态方法等）



ObjectTranslator.cs 中 getTypeId 函数做的是：

1. 以类的名称作为 key 通过 LuaAPI.luaL_getmetatable 获取类对应的元表
2. 如果获取不到，则通过 TryDelayWrapLoader 生成元表
3. 调用 LuaAPI.luaL_ref 将获取到的元表添加到Lua注册表中，并返回 type_id（type_id 是 元表 在 Lua 注册表中的索引）



ObjectTranslator.cs 中 TryDelayWrapLoader 元表信息的生成：

LuaAPI.luaL_newmetatable 新建 metatable 

1. 通过 delayWrap判断，是否有为该类生成预先注册的**类型元表生成器代码**，
2. 如果有，直接使用生成函数进行填充元表（loader方法）。在xLua的生成代码中有一个XLuaGenAutoRegister.cs文件，在这个文件中会为对应的类注册初始化器，而这个初始化器负责将类对应的元表 生成函数 __Register 添加到delayWrap中。
3. 如果没有生成代码，则通过 反射填充元表（ReflectionWrap方法）

​		

###### 使用 生成函数__Register 填充元表 的框架流程

生成代码会为类的非静态值都生成对应的包裹方法，并将包裹方法以 key = func 的形式注册到不同的表中。userdata元表的____index和____newindex负责从这不同的表中找到对应key的包裹方法，最终通过调用包裹方法实现对C#对象的控制

LuaCallCSharp修饰

```c#
// TestXLua.cs
[LuaCallCSharp]
public class TestXLua
{
    public string Name;
    public void Test1(int a){
    }
    public static void Test2(int a, bool b, string c)
    {
    }
}
```

Generate Code 之后 生成的TestXLuaWrap.cs

```c#
public class TestXLuaWrap 
{
    public static void __Register(RealStatePtr L)
    {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        System.Type type = typeof(TestXLua);
        
        //1.准备需要的表 
        //在对类的非静态值（例如成员变量，成员方法等）进行注册前做一些准备工作
        //为元表添加__gc和__tostring元方法，以及准备好method表、getter表、setter表
        Utils.BeginObjectRegister(type, L, translator, 0, 1, 1, 1);
        
        //2.多个Utils.RegisterFunc
        //Generate Code时动态生成包裹方法
        //在这里将类的每个非静态值对应的包裹方法注册到不同的Lua表中
        //将idx指向的表中添加键值对 name = func
        Utils.RegisterFunc(L, Utils.METHOD_IDX, "Test1", _m_Test1);
        Utils.RegisterFunc(L, Utils.GETTER_IDX, "Name", _g_get_Name);
        Utils.RegisterFunc(L, Utils.SETTER_IDX, "Name", _s_set_Name);
        
        //3.结束对类的非静态值的注册
        //主要逻辑是为元表生成__index元方法和__newindex元方法(Lua调用C#的核心所在)
        //压入methods表 
        //压入getters表
        //压入csindexer
        //压入base
        //压入indexfuncs
        //压入arrayindexer
        //参数会作为upvalue关联到闭包obj_indexer 生成__index元方法 LuaAPI.gen_obj_indexer(L);
        // 注册表[LuaIndexs][type] = __index函数
        //生成__newindex元方法
        //注册表[LuaNewIndexs][type] = __newindex函数 LuaAPI.gen_obj_newindexer(L);
        Utils.EndObjectRegister(type, L, translator, null, null,
            null, null, null);
        
        //4.在对类的静态值（例如静态变量，静态方法等）进行注册前做一些准备工作
        //为类生成对应的cls_table表（通过SetCSTable 根据类的命名空间名逐层添加到注册表中），
        //提前创建好static_getter表与static_setter表，后续用来存放静态字段对应的get和set包裹方法。
        //注意这里还会为cls_table设置元表meta_table
        Utils.BeginClassRegister(type, L, __CreateInstance, 2, 0, 0);
        
        //将类的每个静态值对应的包裹方法注册到对应的Lua表中
        //静态变量对应的get和set包裹方法会被分别注册到static_getter表和static_setter表
        //（只读的静态变量除外）
        Utils.RegisterFunc(L, Utils.CLS_IDX, "Test2", _m_Test2_xlua_st_);
        
        //结束对类的静态值的注册
        //为cls_table的元表meta_tabl设置__index元方法和__newindex元方法
        Utils.EndClassRegister(type, L, translator);
    }
    
    [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    static int __CreateInstance(RealStatePtr L)
    {
        try {
            ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
            if(LuaAPI.lua_gettop(L) == 1)
            {
                TestXLua gen_ret = new TestXLua();
                translator.Push(L, gen_ret);
                return 1;
            }
        }
        catch(System.Exception gen_e) {
            return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
        }
        return LuaAPI.luaL_error(L, "invalid arguments to TestXLua constructor!");
        
    }

    [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    static int _m_Test1(RealStatePtr L)
    {
        try {
            ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
            TestXLua gen_to_be_invoked = (TestXLua)translator.FastGetCSObj(L, 1);
            {
                int _a = LuaAPI.xlua_tointeger(L, 2);
                gen_to_be_invoked.Test1( _a );
                return 0;
            }
        } catch(System.Exception gen_e) {
            return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
        }
    }
    
    [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    static int _m_Test2_xlua_st_(RealStatePtr L)
    {
        try {
            {
                int _a = LuaAPI.xlua_tointeger(L, 1);
                bool _b = LuaAPI.lua_toboolean(L, 2);
                string _c = LuaAPI.lua_tostring(L, 3);
                TestXLua.Test2( _a, _b, _c );
                return 0;
            }
        } catch(System.Exception gen_e) {
            return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
        }
    }
    
    [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    static int _g_get_Name(RealStatePtr L)
    {
        try {
            ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        
            TestXLua gen_to_be_invoked = (TestXLua)translator.FastGetCSObj(L, 1);
            LuaAPI.lua_pushstring(L, gen_to_be_invoked.Name);
        } catch(System.Exception gen_e) {
            return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
        }
        return 1;
    }
    
    [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    static int _s_set_Name(RealStatePtr L)
    {
        try {
            ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        
            TestXLua gen_to_be_invoked = (TestXLua)translator.FastGetCSObj(L, 1);
            gen_to_be_invoked.Name = LuaAPI.lua_tostring(L, 2);
        
        } catch(System.Exception gen_e) {
            return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
        }
        return 0;
    }
}
```

lua 调用 C#

```lua
-- lua测试代码
local obj = CS.TestXLua()
obj.Name = "test"  
-- 赋值操作将触发obj元表的__newindex，
--__newindex在setter表中找到Name对应的set包裹方法_s_set_Name，
--然后通过调用_s_set_Name方法设置了TestXLua对象的Name属性为"test"

-- lua测试代码
CS.TestXLua.Test2()  
-- CS.TestXLua获取到TestXLua类对应的cls_table，
--由于Test2是静态方法，在cls_table中可以直接拿到其对应的包裹方法_m_Test2_xlua_st_，
--然后通过调用_m_Test2_xlua_st_而间接调用了TestXLua类的Test2方法
```

生成代码还会为每个类以命名空间为层次结构生成cls_table表。

与类的非静态值相同，生成代码也会为类的静态值都生成对应的包裹方法并注册到不同的表中（注意这里有些区别，类的静态方法会被直接注册到cls_table表中）。

而cls_table元表的__index和__newindex负责从这不同的表中找到对应key的包裹方法，最终通过调用包裹方法实现对C#类的控制



###### 使用反射填充元表（没有生成代码情况下）

当没有生成代码时，会使用反射进行注册，与生成代码进行注册的逻辑基本相同。

通过反射获取到类的各个静态值和非静态值，然后分别注册到不同的表中，以及填充 ____index和 ____newindex元方法

```c#
// Utils.cs
public static void ReflectionWrap(RealStatePtr L, Type type, bool privateAccessible)
{
    //...
    // 为元表添加xlua_tag标志
    LuaAPI.lua_pushlightuserdata(L, LuaAPI.xlua_tag());
	LuaAPI.lua_pushnumber(L, 1);
    LuaAPI.lua_rawset(L, -3);  // 元表[xlua_tag] = 1
    //...
    SetCSTable(L, type, cls_field);
    //...
    
    LuaCSFunction item_getter;
    LuaCSFunction item_setter;
    makeReflectionWrap(L, type, cls_field, cls_getter, cls_setter, obj_field, obj_getter, obj_setter, obj_meta,
        out item_getter, out item_setter, privateAccessible ? (BindingFlags.Public | BindingFlags.NonPublic) : BindingFlags.Public);
	
    //...
    LuaAPI.xlua_pushasciistring(L, "__index");
    LuaAPI.lua_pushvalue(L, obj_field);  // 1.upvalue methods = obj_field
    LuaAPI.lua_pushvalue(L, obj_getter);  // 2.upvalue getters = obj_getter
    translator.PushFixCSFunction(L, item_getter);  // 3.upvalue csindexer = item_getter
    translator.PushAny(L, type.BaseType());  // 压入BaseType，4.upvalue base
    LuaAPI.xlua_pushasciistring(L, LuaIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);  // 5.upvalue indexfuncs = 注册表[LuaIndexs]
    LuaAPI.lua_pushnil(L);  // 6.upvalue arrayindexer = nil
    LuaAPI.gen_obj_indexer(L);  // 生成__index函数
    //store in lua indexs function tables
    LuaAPI.xlua_pushasciistring(L, LuaIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);  
    translator.Push(L, type);  // 压入type
    LuaAPI.lua_pushvalue(L, -3);
    LuaAPI.lua_rawset(L, -3);  // 注册表[LuaIndexs][type] = __index函数
    LuaAPI.lua_pop(L, 1);
    LuaAPI.lua_rawset(L, obj_meta); // set __index  即 obj_meta["__index"] = 生成的__index函数

    LuaAPI.xlua_pushasciistring(L, "__newindex");
    LuaAPI.lua_pushvalue(L, obj_setter);
    translator.PushFixCSFunction(L, item_setter);
    translator.Push(L, type.BaseType());
    LuaAPI.xlua_pushasciistring(L, LuaNewIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
    LuaAPI.lua_pushnil(L);
    LuaAPI.gen_obj_newindexer(L);
    //store in lua newindexs function tables
    LuaAPI.xlua_pushasciistring(L, LuaNewIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
    translator.Push(L, type);
    LuaAPI.lua_pushvalue(L, -3);
    LuaAPI.lua_rawset(L, -3);  // 注册表[LuaNewIndexs][type] = __newindex函数
    LuaAPI.lua_pop(L, 1);
    LuaAPI.lua_rawset(L, obj_meta); // set __newindex
                                    //finish init obj metatable

    LuaAPI.xlua_pushasciistring(L, "UnderlyingSystemType");
    translator.PushAny(L, type);
    LuaAPI.lua_rawset(L, cls_field);  // cls_field["UnderlyingSystemType"] = type  ， 记录类的基础类型

    if (type != null && type.IsEnum())
    {
        LuaAPI.xlua_pushasciistring(L, "__CastFrom");
        translator.PushFixCSFunction(L, genEnumCastFrom(type));
        LuaAPI.lua_rawset(L, cls_field);
    }

    //init class meta
    LuaAPI.xlua_pushasciistring(L, "__index");
    LuaAPI.lua_pushvalue(L, cls_getter);
    LuaAPI.lua_pushvalue(L, cls_field);
    translator.Push(L, type.BaseType());
    LuaAPI.xlua_pushasciistring(L, LuaClassIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
    LuaAPI.gen_cls_indexer(L);
    //store in lua indexs function tables
    LuaAPI.xlua_pushasciistring(L, LuaClassIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
    translator.Push(L, type);
    LuaAPI.lua_pushvalue(L, -3);
    LuaAPI.lua_rawset(L, -3);  // 注册表[LuaClassIndexs][type] = __index函数
    LuaAPI.lua_pop(L, 1);
    LuaAPI.lua_rawset(L, cls_meta); // set __index 

    LuaAPI.xlua_pushasciistring(L, "__newindex");
    LuaAPI.lua_pushvalue(L, cls_setter);
    translator.Push(L, type.BaseType());
    LuaAPI.xlua_pushasciistring(L, LuaClassNewIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
    LuaAPI.gen_cls_newindexer(L);
    //store in lua newindexs function tables
    LuaAPI.xlua_pushasciistring(L, LuaClassNewIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
    translator.Push(L, type);
    LuaAPI.lua_pushvalue(L, -3);
    LuaAPI.lua_rawset(L, -3);  // // 注册表[LuaClassNewIndexs][type] = __newindex函数
    LuaAPI.lua_pop(L, 1);
    LuaAPI.lua_rawset(L, cls_meta); // set __newindex
    // ...
}
```



##### 调用 C# 方法时参数的传递

类的静态值或是非静态值之所以都需要生成对应的包裹方法，是因为包裹方法就是用来处理参数传递问题

C函数已经定义好了协议（协议定义了参数以及返回值传递方法），可以正确的和Lua通讯



**C函数通过Lua中的栈来接受参数，参数以正序入栈（第一个参数首先入栈）。**

因此，当函数开始的时候，lua_gettop(L)可以返回函数收到的参数个数。

第一个参数（如果有的话）在索引1的地方，而最后一个参数在索引lua_gettop(L)处。

当需要向Lua返回值的时候，C函数只需要把它们以正序压到堆栈上（第一个返回值最先压入），然后返回这些返回值的个数。在这些返回值之下的，堆栈上的东西都会被Lua丢掉。

和Lua函数一样，从Lua中调用C函数可以有很多返回值。

也就是说，Lua内部实现是调用C函数时的参数会被自动的压栈。



包裹方法中封装了C#需要通过C的API获取到Lua传递过来的参数，从而与Lua进行数据通信

```c#
//以TestXLua的Test1方法为例，它需要一个int参数。
//所以它的包裹方法需要通过C的API获取到一个int参数，然后再使用这个参数去调用真正的方法
[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
static int _m_Test1(RealStatePtr L)
{
    try {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        TestXLua gen_to_be_invoked = (TestXLua)translator.FastGetCSObj(L, 1);
        {
            int _a = LuaAPI.xlua_tointeger(L, 2);  // 获取到int参数
            gen_to_be_invoked.Test1( _a );  // 调用真正的Test1方法
            return 0;
        }
    } catch(System.Exception gen_e) {
        return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
    }
}
```

只有将Lua的访问或赋值操作转换成函数调用形式时（为类的属性生成对应的get和set方法），参数才能利用函数调用机制被自动的压栈，从而传递给C#。

```lua
-- lua测试代码
obj.Name = "test"  -- 赋值操作
setter["Name"]("test")  -- 函数调用形式
```



**函数重载**：同名函数的参数不同的情况，只能通过同名函数被调用时传递的参数情况来判断到底应该调用哪个函数

```c#
[LuaCallCSharp]
public class TestXLua
{
    // 函数重载Test1
    public void Test1(int a){
    }
    // 函数重载Test1
    public void Test1(bool b){
    }
}

// 为Test1生成的包裹方法
[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
static int _m_Test1(RealStatePtr L)
{
    try {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        TestXLua gen_to_be_invoked = (TestXLua)translator.FastGetCSObj(L, 1);
        int gen_param_count = LuaAPI.lua_gettop(L);
        if(gen_param_count == 2&& LuaTypes.LUA_TNUMBER == LuaAPI.lua_type(L, 2))  
            // 根据参数数量与类型判断调用哪个方法
        {
            int _a = LuaAPI.xlua_tointeger(L, 2);
            gen_to_be_invoked.Test1( _a );
            return 0;
        }
        if(gen_param_count == 2&& LuaTypes.LUA_TBOOLEAN == LuaAPI.lua_type(L, 2))  
            // 根据参数数量与类型判断调用哪个方法
        { 
            bool _b = LuaAPI.lua_toboolean(L, 2);
            gen_to_be_invoked.Test1( _b );
            return 0;
        }
    } catch(System.Exception gen_e) {
        return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
    }
    return LuaAPI.luaL_error(L, "invalid arguments to TestXLua.Test1!");
}
```



#### GC 

基本原则是传递到Lua的C#对象，C#不能自动回收，**只能Lua在确定不再使用后通知C#进行回收**

为了保证C#不会自动回收对象，所有传递给Lua的对象都会被objects保持引用。

真实传递给Lua的对象索引就是对象在objects中的索引

Lua这边为对象索引建立的userdata会被保存在缓存表中，而缓存表的引用模式被设置为弱引用

```c#
// ObjectTranslator.cs
LuaAPI.lua_newtable(L);  // 创建缓存表
LuaAPI.lua_newtable(L);  // 创建元表
LuaAPI.xlua_pushasciistring(L, "__mode");
LuaAPI.xlua_pushasciistring(L, "v");
LuaAPI.lua_rawset(L, -3);  // 元表[__mode] = v，表示这张表的所有值皆为弱引用
LuaAPI.lua_setmetatable(L, -2);  // 为缓存表设置元表
cacheRef = LuaAPI.luaL_ref(L, LuaIndexes.LUA_REGISTRYINDEX);
```

当Lua这边不再引用这个userdata时，userdata会被从缓存表中移除，Lua GC时会回收这个userdata，回收之前又会调用userdata元表的**__gc方法**，以此来通知C#可以回收。

在BeginObjectRegister方法内部，会为userdata的元表添加__gc方法

```c#
// Utils.cs BeginObjectRegister方法
if ((type == null || !translator.HasCustomOp(type)) && type != typeof(decimal))
{
    LuaAPI.xlua_pushasciistring(L, "__gc");
    //translator.metaFunctions.GcMeta实际上就是 StaticLuaCallbacks的LuaGC方法
    LuaAPI.lua_pushstdcallcfunction(L, translator.metaFunctions.GcMeta);
    LuaAPI.lua_rawset(L, -3);  // 为元表设置__gc方法
}
```

StaticLuaCallbacks的LuaGC：

```c#
// StaticLuaCallbacks.cs
[MonoPInvokeCallback(typeof(LuaCSFunction))]
public static int LuaGC(RealStatePtr L)
{
    try
    {
        int udata = LuaAPI.xlua_tocsobj_safe(L, 1);
        if (udata != -1)
        {
            ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
            if ( translator != null )
            {
                //调用collectObject方法：
                //内部会将对象从objects移除，从而使对象不再被固定引用，能够被C# GC正常回收
                translator.collectObject(udata);
            }
        }
        return 0;
    }
    catch (Exception e)
    {
        return LuaAPI.luaL_error(L, "c# exception in LuaGC:" + e);
    }
}

// ObjectTranslator.cs
internal void collectObject(int obj_index_to_collect)
{
    object o;
    
    if (objects.TryGetValue(obj_index_to_collect, out o))
    {
        objects.Remove(obj_index_to_collect);
        
        if (o != null)
        {
            int obj_index;
            //弱引用的表
            //lua gc是先把weak table移除后再调用__gc，
            //这期间同一个对象可能再次push到lua，关联到新的index
            bool is_enum = o.GetType().IsEnum();
            if ((is_enum ? enumMap.TryGetValue(o, out obj_index) : reverseMap.TryGetValue(o, out obj_index))
                && obj_index == obj_index_to_collect)
            {
                if (is_enum)
                {
                    enumMap.Remove(o);
                }
                else
                {
                    reverseMap.Remove(o);
                }
            }
        }
    }
}
```



### 2. C#与Lua数据通信机制（C# 调用 Lua）

#### 传递Lua table 到 C#

```c#
// 注意，这里添加的LuaCallCSharp特性只是为了使xLua为其生成代码，不添加并不影响功能
[LuaCallCSharp]
public class TestXLua
{
    //LuaTable类型的静态变量 C#这边定义的一个类，封装了一些对Lua table的操作
    public static LuaTable tab;
}
```

点击 Generate Code 之后，为tab变量生成了对应的set和get包裹方法

```c#
// TestXLuaWrap.cs
[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
static int _g_get_tab(RealStatePtr L)
{
    try {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        translator.Push(L, TestXLua.tab);
    } catch(System.Exception gen_e) {
        return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
    }
    return 1;
}

[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
static int _s_set_tab(RealStatePtr L)
{
    try {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        TestXLua.tab = (XLua.LuaTable)translator.GetObject(L, 1, typeof(XLua.LuaTable));
    
    } catch(System.Exception gen_e) {
        return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
    }
    return 0;
}
```

```lua
--为tab静态变量赋值一个Lua table，table中包含一个 num = 1 键值对
-- Lua测试代码
local t = {
    num = 1
}
--Lua这边调用_s_set_tab前，会先将参数table t压入到栈中，
--因此_s_set_tab内部需要通过translator.GetObject拿到这个table，并将其赋值给tab静态变量
CS.TestXLua.tab = t
```

```c#
// ObjectTranslator.cs
// GetObject 从栈中获取指定类型的对象
public object GetObject(RealStatePtr L, int index, Type type)
{
    int udata = LuaAPI.xlua_tocsobj_safe(L, index);

    if (udata != -1)
    {
        // 对C#对象的处理
        object obj = objects.Get(udata);
        RawObject rawObject = obj as RawObject;
        return rawObject == null ? obj : rawObject.Target;
    }
    else
    {
        if (LuaAPI.lua_type(L, index) == LuaTypes.LUA_TUSERDATA)
        {
            GetCSObject get;
            int type_id = LuaAPI.xlua_gettypeid(L, index);
            if (type_id != -1 && type_id == decimal_type_id)
            {
                decimal d;
                Get(L, index, out d);
                return d;
            }
            Type type_of_struct;
            if (type_id != -1 && typeMap.TryGetValue(type_id, out type_of_struct) && type.IsAssignableFrom(type_of_struct) && custom_get_funcs.TryGetValue(type, out get))
            {
                return get(L, index);
            }
        }
        return (objectCasters.GetCaster(type)(L, index, null));
    }
}

// ObjectTranslator.cs
// 对于LuaTable类型是通过objectCasters.GetCaster获取转换器后，通过转换器函数转换得到
public ObjectCast GetCaster(Type type)
{
    if (type.IsByRef) type = type.GetElementType();  // 如果是按引用传递的，则使用引用的对象的type

    Type underlyingType = Nullable.GetUnderlyingType(type);
    if (underlyingType != null)
    {
        return genNullableCaster(GetCaster(underlyingType)); 
    }
    ObjectCast oc;
    if (!castersMap.TryGetValue(type, out oc))
    {
        oc = genCaster(type);
        castersMap.Add(type, oc);
    }
    return oc;
}
```

```c#
// ObjectCasters.cs
// 在castersMap中为一些类型定义好了转换函数，其中就包括我们新增的指定的类型 LuaTable类型
public ObjectCasters(ObjectTranslator translator)
{
    this.translator = translator;
    castersMap[typeof(char)] = charCaster;
    castersMap[typeof(sbyte)] = sbyteCaster;
    castersMap[typeof(byte)] = byteCaster;
    castersMap[typeof(short)] = shortCaster;
    castersMap[typeof(ushort)] = ushortCaster;
    castersMap[typeof(int)] = intCaster;
    castersMap[typeof(uint)] = uintCaster;
    castersMap[typeof(long)] = longCaster;
    castersMap[typeof(ulong)] = ulongCaster;
    castersMap[typeof(double)] = getDouble;
    castersMap[typeof(float)] = floatCaster;
    castersMap[typeof(decimal)] = decimalCaster;
    castersMap[typeof(bool)] = getBoolean;
    castersMap[typeof(string)] =  getString;
    castersMap[typeof(object)] = getObject;
    castersMap[typeof(byte[])] = getBytes;
    castersMap[typeof(IntPtr)] = getIntptr;
    //special type
    castersMap[typeof(LuaTable)] = getLuaTable;
    castersMap[typeof(LuaFunction)] = getLuaFunction;
}

//LuaTable对应的转换函数是getLuaTable
// ObjectCasters.cs
private object getLuaTable(RealStatePtr L, int idx, object target)
{
    //将idx处的table通过luaL_ref添加到Lua注册表中并得到指向该table的索引，
    //然后创建LuaTable对象保存该索引
    //Lua table在C#这边对应的是LuaTable对象，它们之间通过一个索引关联起来
    //这个索引表示Lua table在Lua注册表中的引用，利用这个索引可以获取到Lua table。
    if (LuaAPI.lua_type(L, idx) == LuaTypes.LUA_TUSERDATA)
    {
    	//拿到Lua table后，就可以继续访问Lua table的内容了。
        object obj = translator.SafeGetCSObj(L, idx);
        return (obj != null && obj is LuaTable) ? obj : null;
    }
    if (!LuaAPI.lua_istable(L, idx))
    {
        return null;
    }
    // 处理普通table类型
    LuaAPI.lua_pushvalue(L, idx);
    return new LuaTable(LuaAPI.luaL_ref(L), translator.luaEnv);
}

```

C# 中获取 Lua 的数据

```c#
// CS测试代码
int num = TestXLua.tab.Get<int>("num");
```

对Lua table的访问操作都被封装在LuaTable.cs 的Get方法中。

Get方法的主要逻辑是，

	1. 先通过保存的索引 luaReference 获取到Lua table，
	1. 通过 xlua_pgettable 将 表[key] 的值压栈，
	1. 通过 translator.Get 获取到栈顶值对应的对象

```c#
// LuaTable.cs
public TValue Get<TValue>(string key)
{
    TValue ret;
    Get(key, out ret);
    return ret;
}

// no boxing version get
// 无装箱版本去获取
public void Get<TKey, TValue>(TKey key, out TValue value)
{
#if THREAD_SAFE || HOTFIX_ENABLE
    lock (luaEnv.luaEnvLock)
    {
#endif
        var L = luaEnv.L;
        var translator = luaEnv.translator;
        int oldTop = LuaAPI.lua_gettop(L);
        LuaAPI.lua_getref(L, luaReference);  // 通过luaReference获取到对应的table
        translator.PushByType(L, key);

        if (0 != LuaAPI.xlua_pgettable(L, -2))  // 查询 表[key]
        {
            string err = LuaAPI.lua_tostring(L, -1);
            LuaAPI.lua_settop(L, oldTop);
            throw new Exception("get field [" + key + "] error:" + err);
        }

        LuaTypes lua_type = LuaAPI.lua_type(L, -1);
        Type type_of_value = typeof(TValue);
        if (lua_type == LuaTypes.LUA_TNIL && type_of_value.IsValueType())
        {
            throw new InvalidCastException("can not assign nil to " + type_of_value.GetFriendlyName());
        }

        try
        {
            translator.Get(L, -1, out value);  // 获取栈顶的元素，即 表[key]
        }
        catch (Exception e)
        {
            throw e;
        }
        finally
        {
            LuaAPI.lua_settop(L, oldTop);
        }
#if THREAD_SAFE || HOTFIX_ENABLE
    }
#endif
}

// ObjectTranslator.cs
public void Get<T>(RealStatePtr L, int index, out T v)
{
    Func<RealStatePtr, int, T> get_func;
    if (tryGetGetFuncByType(typeof(T), out get_func))
    {
        v = get_func(L, index);  // 将给定索引处的值转换为{T}类型
    }
    else
    {
        v = (T)GetObject(L, index, typeof(T));
    }
}
```

xLua也在 tryGetGetFuncByType 中为一些基本类型预定义好了对应的对象获取方法，泛型方式避免拆箱和装箱

```c#

bool tryGetGetFuncByType<T>(Type type, out T func) where T : class
{
    if (get_func_with_type == null)
    {
        get_func_with_type = new Dictionary<Type, Delegate>()
        {
            {typeof(int), new Func<RealStatePtr, int, int>(LuaAPI.xlua_tointeger) },
            {typeof(double), new Func<RealStatePtr, int, double>(LuaAPI.lua_tonumber) },
            {typeof(string), new Func<RealStatePtr, int, string>(LuaAPI.lua_tostring) },
            {typeof(byte[]), new Func<RealStatePtr, int, byte[]>(LuaAPI.lua_tobytes) },
            {typeof(bool), new Func<RealStatePtr, int, bool>(LuaAPI.lua_toboolean) },
            {typeof(long), new Func<RealStatePtr, int, long>(LuaAPI.lua_toint64) },
            {typeof(ulong), new Func<RealStatePtr, int, ulong>(LuaAPI.lua_touint64) },
            {typeof(IntPtr), new Func<RealStatePtr, int, IntPtr>(LuaAPI.lua_touserdata) },
            {typeof(decimal), new Func<RealStatePtr, int, decimal>((L, idx) => {
                decimal ret;
                Get(L, idx, out ret);
                return ret;
            }) },
            
            //xlua_tointeger就是对Lua原生API lua_tointeger的一个简单封装
            {typeof(byte), new Func<RealStatePtr, int, byte>((L, idx) => (byte)LuaAPI.xlua_tointeger(L, idx) ) },
            {typeof(sbyte), new Func<RealStatePtr, int, sbyte>((L, idx) => (sbyte)LuaAPI.xlua_tointeger(L, idx) ) },
            {typeof(char), new Func<RealStatePtr, int, char>((L, idx) => (char)LuaAPI.xlua_tointeger(L, idx) ) },
            {typeof(short), new Func<RealStatePtr, int, short>((L, idx) => (short)LuaAPI.xlua_tointeger(L, idx) ) },
            {typeof(ushort), new Func<RealStatePtr, int, ushort>((L, idx) => (ushort)LuaAPI.xlua_tointeger(L, idx) ) },
            {typeof(uint), new Func<RealStatePtr, int, uint>(LuaAPI.xlua_touint) },
            {typeof(float), new Func<RealStatePtr, int, float>((L, idx) => (float)LuaAPI.lua_tonumber(L, idx) ) },
        };
    }
```



#### 传递 Lua function 到 C#

Lua的function传递到C#后，对应的是C#的委托

```c#
// 注意，这里添加的LuaCallCSharp特性只是为了使xLua为其生成代码，不添加并不影响功能
[LuaCallCSharp]
public class TestXLua
{
    [CSharpCallLua]
    public delegate int Func(string s, bool b, float f);

    public static Func func;
}
```

点击Generate Code后，生成的部分TestXLuaWrap代码如下所示

为func变量生成了对应的 set 和 get 包裹方法

```c#
// TestXLuaWrap.cs
[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
static int _g_get_func(RealStatePtr L)
{
    try {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        translator.Push(L, TestXLua.func);
    } catch(System.Exception gen_e) {
        return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
    }
    return 1;
}

[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
static int _s_set_func(RealStatePtr L)
{
    try {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        TestXLua.func = translator.GetDelegate<TestXLua.Func>(L, 1);
    
    } catch(System.Exception gen_e) {
        return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
    }
    return 0;
}

```

为func静态变量赋值一个Lua function

```lua
-- Lua测试代码
-- 在赋值时，会最终调用_s_set_func包裹方法
CS.TestXLua.func = function(s, b, i)
    
end
```



**将方法赋值 的时候做了什么：**

Lua在调用 **_s_set_func** 前，会将参数 function 压入到栈中

因此 _s_set_func 内部需要通过 translator.GetDelegate 拿到这个function，并将其赋值给 func 静态变量

```c#
// ObjectTranslator.cs
public T GetDelegate<T>(RealStatePtr L, int index) where T :class
{
    
    if (LuaAPI.lua_isfunction(L, index))
    {
        /*
        对于Lua function类型会通过 CreateDelegateBridge 创建一个对应的委托并返回。
        CreateDelegateBridge内部会创建一个DelegateBridge对象来对应Lua function，
        原理和LuaTable类似，也是通过一个索引保持联系，
        利用这个索引可以获取到Lua function
        */
        return CreateDelegateBridge(L, typeof(T), index) as T;
    }
    else if (LuaAPI.lua_type(L, index) == LuaTypes.LUA_TUSERDATA)
    {
        return (T)SafeGetCSObj(L, index);
    }
    else
    {
        return null;
    }
}


// ObjectTranslator.cs
Dictionary<int, WeakReference> delegate_bridges = new Dictionary<int, WeakReference>();  // 弱引用创建的DelegateBridge
public object CreateDelegateBridge(RealStatePtr L, Type delegateType, int idx)
{
    LuaAPI.lua_pushvalue(L, idx);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
    // 对缓存的处理
    if (!LuaAPI.lua_isnil(L, -1))
    {
        int referenced = LuaAPI.xlua_tointeger(L, -1);
        LuaAPI.lua_pop(L, 1);

        if (delegate_bridges[referenced].IsAlive)
        {
            if (delegateType == null)
            {
                return delegate_bridges[referenced].Target;
            }
            DelegateBridgeBase exist_bridge = delegate_bridges[referenced].Target as DelegateBridgeBase;
            Delegate exist_delegate;
            if (exist_bridge.TryGetDelegate(delegateType, out exist_delegate))
            {
                return exist_delegate;
            }
            else
            {
                exist_delegate = getDelegate(exist_bridge, delegateType);
                exist_bridge.AddDelegate(delegateType, exist_delegate);
                return exist_delegate;
            }
        }
    }
    else
    {
        LuaAPI.lua_pop(L, 1);
    }

    LuaAPI.lua_pushvalue(L, idx);
    int reference = LuaAPI.luaL_ref(L);  // 将idx处的元素添加到Lua注册表中
    LuaAPI.lua_pushvalue(L, idx);
    LuaAPI.lua_pushnumber(L, reference);
    LuaAPI.lua_rawset(L, LuaIndexes.LUA_REGISTRYINDEX);  // 注册表[idx值] = reference
    DelegateBridgeBase bridge;
    try
    {
#if (UNITY_EDITOR || XLUA_GENERAL) && !NET_STANDARD_2_0
        if (!DelegateBridge.Gen_Flag)
        {
            bridge = Activator.CreateInstance(delegate_birdge_type, new object[] { reference, luaEnv }) as DelegateBridgeBase;  // 使用反射创建DelegateBridge对象
        }
        else
#endif
        {
            bridge = new DelegateBridge(reference, luaEnv);
        }
    }
    catch(Exception e)
    {
        LuaAPI.lua_pushvalue(L, idx);
        LuaAPI.lua_pushnil(L);
        LuaAPI.lua_rawset(L, LuaIndexes.LUA_REGISTRYINDEX);
        LuaAPI.lua_pushnil(L);
        LuaAPI.xlua_rawseti(L, LuaIndexes.LUA_REGISTRYINDEX, reference);
        throw e;
    }
    if (delegateType == null)
    {
        delegate_bridges[reference] = new WeakReference(bridge);
        return bridge;
    }
    try {
        /*
        在取得DelegateBridge对象后，
        还需要通过getDelegate方法，获取delegateType类型的委托，
        即C#这边指定要接收Lua function时声明的委托类型。
        在本例中是typeof(TestXLua.Func)
        */
        var ret = getDelegate(bridge, delegateType);  // 通过bridge获取到指定类型的委托
        bridge.AddDelegate(delegateType, ret);
        delegate_bridges[reference] = new WeakReference(bridge);
        return ret;
    }
    catch(Exception e)
    {
        bridge.Dispose();
        throw e;
    }
}

/*
1. 先获得delegateType委托的Invoke方法，
2. 通过反射遍历bridge类型的所有方法，找到与Invoke参数匹配的目标方法
3. 再使用bridge实例与目标方法创建一个delegateType类型的委托。
*/
Delegate getDelegate(DelegateBridgeBase bridge, Type delegateType)
{
    // ...
    Func<DelegateBridgeBase, Delegate> delegateCreator;
    if (!delegateCreatorCache.TryGetValue(delegateType, out delegateCreator))
    {
        // get by parameters
        MethodInfo delegateMethod = delegateType.GetMethod("Invoke");
        // 生成代码为配置了 CSharpCallLua的委托 生成以__Gen_Delegate_Imp开头的方法 并添加到 DelegateBridge 类中
        var methods = bridge.GetType().GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly).Where(m => !m.IsGenericMethodDefinition && (m.Name.StartsWith("__Gen_Delegate_Imp") || m.Name == "Action")).ToArray();
        // 查找bridge中与delegateMethod匹配的方法，这个方法必须是以__Gen_Delegate_Imp或Action开头
        for (int i = 0; i < methods.Length; i++)  
        {
            if (!methods[i].IsConstructor && Utils.IsParamsMatch(delegateMethod, methods[i]))
            {
                var foundMethod = methods[i];
                delegateCreator = (o) =>
#if !UNITY_WSA || UNITY_EDITOR
                    Delegate.CreateDelegate(delegateType, o, foundMethod);  // 创建表示foundMethod的delegateType类型的委托
#else
                    foundMethod.CreateDelegate(delegateType, o); 
#endif
                break;
            }
        }

        if (delegateCreator == null)
        {
            delegateCreator = getCreatorUsingGeneric(bridge, delegateType, delegateMethod);
        }
        delegateCreatorCache.Add(delegateType, delegateCreator);
    }

    ret = delegateCreator(bridge);  // 创建委托
    if (ret != null)
    {
        return ret;
    }
	
    //用于接收Lua function的委托必须添加CSharpCallLua特性也正是因为要为其生成以"__Gen_Delegate_Imp"开头的方法，如果不添加则会抛出异常
    throw new InvalidCastException("This type must add to CSharpCallLua: " + delegateType.GetFriendlyName());
}




// DelegatesGensBridge.cs
public partial class DelegateBridge : DelegateBridgeBase
{
    // ...
    // TestXLua.Func类型委托绑定
    // 生成函数来完成参数的压栈与Lua function调用
    public int __Gen_Delegate_Imp1(string p0, bool p1, float p2)
    {
#if THREAD_SAFE || HOTFIX_ENABLE
        lock (luaEnv.luaEnvLock)
        {
#endif
            RealStatePtr L = luaEnv.rawL;
            int errFunc = LuaAPI.pcall_prepare(L, errorFuncRef, luaReference);
            
            LuaAPI.lua_pushstring(L, p0);  // 压栈参数
            LuaAPI.lua_pushboolean(L, p1);  // 压栈参数
            LuaAPI.lua_pushnumber(L, p2);  // 压栈参数
            
            PCall(L, 3, 1, errFunc);  // Lua function调用
            
            
            int __gen_ret = LuaAPI.xlua_tointeger(L, errFunc + 1);
            LuaAPI.lua_settop(L, errFunc - 1);
            return  __gen_ret;
#if THREAD_SAFE || HOTFIX_ENABLE
        }
#endif
    }
}
```



C 函数 定义好了 与 Lua 通讯的协议：定义了参数以及返回值传递方法

C 函数通过Lua中的栈来接受参数，参数以正序入栈（第一个参数首先入栈）。

1. 当函数开始的时候，lua_gettop(L)可以返回函数收到的参数个数。
2. 第一个参数（如果有的话）在索引1的地方，而最后一个参数在索引lua_gettop(L)处。
3. 当需要向Lua返回值的时候，C函数只需要把它们以正序压到堆栈上（第一个返回值最先压入），然后返回这些返回值的个数。
4. 在这些返回值之下的，堆栈上的东西都会被Lua丢掉。
5. 和Lua函数一样，从Lua中调用C函数可以有很多返回值。



C#可以借助C/C++来与Lua进行数据通信：

	1.  C#在函数调用前，需要通过C API来压栈函数调用所需的参数（封装在了以"__Gen_Delegate_Imp"开头的生成方法中）
	1.  生成方法将参数压栈
	1.  通过PCall调用Lua function（PCall内部调用的就是Lua原生API lua_pcall



**总结：**

```lua
-- Lua测试代码
CS.TestXLua.func = function(s, b, i)
    
end
```

当为TestXLua.func赋值Lua function时：

	1. 会触发func变量的set包裹方法 **_s_set_func**，
	1.  _s_set_func内 部会获取一个委托设置给func变量
	1. 委托绑定的是 **DelegateBridge对象** 的 以"__Gen_Delegate_Imp"开头的生成方法
	1. DelegateBridge对象同时保存着 Lua function的索引

```c#
// CS测试代码
TestXLua.func("test", false, 3);
```



#### GC

基本原则是传递到C# 的 Lua 对象，Lua 不能自动回收，**只能 C# 在确定不再使用后通知 Lua 进行回收**

所有传递给C#的对象都会被Lua注册表引用

​	比如前面创建LuaTable或DelegateBridge时 都有调用LuaAPI.luaL_ref将对象添加到注册表中

C#这边为对应的Lua对象定义了LuaBase基类，LuaTable或DelegateBridge均派生于LuaBase，这个类实现了IDisposable接口，并且在析构函数中会调用Dispose

```c#
// LuaBase.cs
public virtual void Dispose(bool disposeManagedResources)
{
    if (!disposed)
    {
        if (luaReference != 0)
        {
#if THREAD_SAFE || HOTFIX_ENABLE
            lock (luaEnv.luaEnvLock)
            {
#endif
                bool is_delegate = this is DelegateBridgeBase;
                if (disposeManagedResources)
                {
                    // 当disposeManagedResources为true时，直接调用ReleaseLuaBase释放Lua对象
                    luaEnv.translator.ReleaseLuaBase(luaEnv.L, luaReference, is_delegate);  // 释放Lua对象
                }
                else //will dispse by LuaEnv.GC
                {
                    luaEnv.equeueGCAction(new LuaEnv.GCAction { Reference = luaReference, IsDelegate = is_delegate });  // 加入GC队列
                }
#if THREAD_SAFE || HOTFIX_ENABLE
            }
#endif
        }
        disposed = true;
    }
}
```

```c#

// ObjectTranslator.cs
// 将Lua对象从Lua注册表中移除，这样Lua GC时发现该对象不再被引用，就可以进行回收了
public void ReleaseLuaBase(RealStatePtr L, int reference, bool is_delegate)
{
    if(is_delegate)
    {
        LuaAPI.xlua_rawgeti(L, LuaIndexes.LUA_REGISTRYINDEX, reference);
        if (LuaAPI.lua_isnil(L, -1))
        {
            LuaAPI.lua_pop(L, 1);
        }
        else
        {
            LuaAPI.lua_pushvalue(L, -1);
            LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
            if (LuaAPI.lua_type(L, -1) == LuaTypes.LUA_TNUMBER && LuaAPI.xlua_tointeger(L, -1) == reference) //
            {
                //UnityEngine.Debug.LogWarning("release delegate ref = " + luaReference);
                LuaAPI.lua_pop(L, 1);// pop LUA_REGISTRYINDEX[func]
                LuaAPI.lua_pushnil(L);
                LuaAPI.lua_rawset(L, LuaIndexes.LUA_REGISTRYINDEX); // LUA_REGISTRYINDEX[func] = nil
            }
            else //another Delegate ref the function before the GC tick
            {
                LuaAPI.lua_pop(L, 2); // pop LUA_REGISTRYINDEX[func] & func
            }
        }

        LuaAPI.lua_unref(L, reference);
        delegate_bridges.Remove(reference);
    }
    else
    {
        LuaAPI.lua_unref(L, reference);
    }
}
```

```c#
// LuaEnv.cs
public void Tick()
{
#if THREAD_SAFE || HOTFIX_ENABLE
    lock (luaEnvLock)
    {
#endif
        var _L = L;
        lock (refQueue) 
        {
            while (refQueue.Count > 0)  // 遍历GC队列
            {
                GCAction gca = refQueue.Dequeue();
                //当disposeManagedResources为false时，会将其加入GC队列。
                //当主动释放Lua环境时，会遍历GC队列，再逐一调用ReleaseLuaBase进行释放
                translator.ReleaseLuaBase(_L, gca.Reference, gca.IsDelegate);
            }
        }
#if !XLUA_GENERAL
        last_check_point = translator.objects.Check(last_check_point, max_check_per_tick, object_valid_checker, translator.reverseMap);
#endif
#if THREAD_SAFE || HOTFIX_ENABLE
    }
#endif
}
```



## lua 和 Unity 的 GC



### 传参



**lua 中对 Vector3 优化：**

一个object要访问成员方法或者成员变量，都需要查找Lua userdata和C#对象的引用，或者查找metatable，耗时甚多。传统的 C#将Vector3传给Lua，整个流程如下：

1.  C#中拿到Vector3的x、y、z三个值；
2.  Push这3个float给Lua栈；
3.  然后构造一个表，将表的x,y,z赋值；

4.  将这个表push到返回值里。

​	一个简单的传参Vector3  就要完成3次push参数、表内存分配、3次表插入，性能消耗严重

**在函数中传递三个 float 比传递 Vector3 快**

​		void SetPos(GameObject obj, Vector3 pos)

​	要改为：

```c#
class LuaUtil{
    //省掉了transform的频繁返回，而且还避免了transform经常临时返回引起Lua的GC
  static void SetPos(GameObject obj, float x, float y, float z){
      obj.transform.position = new Vector3(x, y, z); 
  }
}
```

因为 主流的方案都是 将Vector3等类型实现为纯Lua代码，Vector3就是一个{x,y,z}的table，速度会更快



**将Vector3替换为数组而不是xyz的访问方式（数组访问效率优于hashtable）**



**Lua和C#之间传参、返回时，尽可能不要传递以下类型：**

1. 严重类： Vector3/Quaternion等Unity值类型，数组
2. 次严重类：bool string 各种object
3. 建议传递：int float double



**函数的参数数量的控制：**

无论是Lua的pushint/checkint，还是C到C#的参数传递，**参数转换都是最主要的消耗**，而且是逐个参数进行的，因此，Lua调用C#的性能，除了跟参数类型相关外，也跟参数个数有很大关系。一般而言，频繁调用的函数不要超过4个参数



**Lua拿着C#对象的引用时会造成C#对象无法释放，这是内存泄漏常见的起因**

C# object返回给Lua，是通过dictionary将Lua的userdata和C# object关联起来，只要Lua中的userdata没回收，C# object也就会被这个dictionary拿着引用，导致无法回收。最常见的就是gameobject和component，如果Lua里头引用了他们，即使你进行了Destroy，也会发现他们还残留在mono堆里。不过，因为这个dictionary是Lua跟C#的唯一关联，所以要发现这个问题也并不难，遍历一下这个dictionary就很容易发现。

解决：**自己分配ID去索引Object**，同时相关C#导出函数不再传递Object做参数，而是传递int

```c#
LuaUtil.SetPos(int objID, float x, float y, float z)。
```

然后我们在自己的代码里头记录objID跟GameObject的对应关系，如果可以，**用数组来记录而不是dictionary，则会有更快的查找效率**。如此下来可以进一步省掉Lua调用C#的时间，并且对象的管理也会更高效。



### 返回值



C#向Lua返回各种类型的东西跟传参类似，也是有各种消耗的。

比如

​	 Vector3 GetPos(GameObject obj) 

可以写成 

​	void GetPos(GameObject obj, out float x, out float y, out float z)。



表面上参数个数增多了，但是根据生成出来的导出代码（我们以uLua为准），会从：LuaDLL.tolua_getfloat3（内含get_field + tonumber 3次） 变成 isnumber + tonumber 3次。

get_field本质上是表查找，肯定比isnumber访问栈更慢，因此这样做会有更好的性能





### table 的优化

如果每一帧都将table初始化，类似v = {}的写法，会不停增加lua内存，

解决：可以封装一个全局的方法，做这个table的重复利用
这样不会造成他每一次都重新新建一个table,只是清空他本身的数据

```lua
--- 清空table
------@param src table 表
function table.Clear(_src)
   if _src then
      for k,v in pairs(_src) do
         _src[k] = nil
      end
   end
   return _src;
end
```

创建一个table的性能是非常耗时的，可以在一些高频使用table的地方做一个全局的缓存池



## 热更新机制

热更新主要负责更新两块内容： 

① 对打有标志的代码脚本，脚本方法进行热更新(重写，内嵌等)，同时更新版本号 

②对AB资源包进行下载更新，同时更新版本号 所谓更新，就是在载入时候，判断是否需要更新内容，如果需要更新，则从服务器上下载代码/资源到客户端进行更新。由于只是更新很小一部分，所以速度极快。 但是对于脚本较大改动的更新，还是要进行版本大更新才行(重新下载安装包)。 还有一些无法更新的地方，也要放到版本大更新中进行。 



补丁开发的过程  （重要） 

打补丁的原理是给打有标签的类设置一个委托，如果委托不为空，则执行委托，如果委托为空，则执行原方法

版本初期的时候，建议全部的标注上HotFix，等到版本稳定下来后，确定某一部分不再进行改动，则可以去掉，去掉之后，则一板块内容无法进行热更新

①首先开发业务代码 

②对所有较大可能变动的类型加上Hotfix标识，在所有Lua调用CSharp的方法上，打上LuaCallCsharp,在所有CSharp调用Lua的方法上，打上CSharpCallLua ③打包发布 

④修改Bug时，只需要更新Lua文件，修改资源时(声音，模型，贴图，图片, UI) 只需要更新ab包，用户只需要去下载lua文件跟ab包。



## lua协程

相同点：协程 与 多线程 都有 挂起、运行、死亡 等一系列状态

不同点：

 1. 多线程是独立于主线程的，同一时间可以有多个多线程在运行；

 2. 协程则是都只能存在于主线程中，同一时间最多只能有一个协程在运行

    

协程 和 子程序（函数/方法）：

相同点：

​	都是在主线程中被执行

区别：

​	协程可以被挂起从而实现异步。可以在 加载文件 的同时 也 执行着主线程中的所有代码。（**加载大资源文件**的时候，把加载文件的逻辑放入到协程中去执行，那么我们就可以让主线程执行到加载文件时，只加载 0.1s 的时间，然后就把这个协程挂起，去执行下面的程序。然后主线程下次又来到加载文件这个协程时，我们再恢复协程，然后再加载 0.1s 的时间，再去执行下面的程序）



Unity 中的协程：

开启一个协程：MonoBehaviour 类的方法 StartCoroutine(IEnumerator cor) 

只能在 Mono 或者 Mono 子类中进行调用，传入的参数是迭代器类型(IEnumerator)参数



Lua 中的协程：

coroutine.create：创建协程对象，传入一个方法作为协程对象的方法体

​	创建的协程默认挂起

coroutine.resume：运行协程，把协程对象作为参数传入

coroutine.yield：挂起协程，把 yield 传入的参数输出给 调用 resume 的方法



```lua
function foo (a)
    print("foo 函数输出", a)
    return coroutine.yield(2 * a) -- 返回  2*a 的值
end
 
co = coroutine.create(function (a , b)
    print("第一次协同程序执行输出", a, b) -- co-body 1 10
    local r = foo(a + 1)
     
    print("第二次协同程序执行输出", r)
    local r, s = coroutine.yield(a + b, a - b)  -- a，b的值为第一次调用协同程序时传入
     
    print("第三次协同程序执行输出", r, s)
    return b, "结束协同程序"                   -- b的值为第二次调用协同程序时传入
end)
       
print("main", coroutine.resume(co, 1, 10)) -- true, 4
print("--分割线----")
print("main", coroutine.resume(co, "r")) -- true 11 -9
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- true 10 end
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- cannot resume dead coroutine
print("---分割线---")
```

以上实例执行输出结果为：

```
第一次协同程序执行输出    1    10
foo 函数输出    2
main    true    4
--分割线----
第二次协同程序执行输出    r
main    true    11    -9
---分割线---
第三次协同程序执行输出    x    y
main    true    10    结束协同程序
---分割线---
main    false    cannot resume dead coroutine
---分割线---
```



#### Unity 协程 与 Lua 协程的互通

把 Lua 脚本 与 Mono 脚本进行关联，XLua 提供了 LuaBehaviour 类。其把 Lua脚本的 self 设置为 Mono 类的对象，来把 Lua 脚本 与 Mono 脚本进行关联。

![](images\LuaBehaviour 的 Awake 方法.webp)

通过 XLua 来开启一个协程：self:StartCoroutine(cor) 来开启

注意，使用的是 冒号(:) 调用 而非 点号(.) 调用，因为冒号调用是Lua调用对象方法的书写，而点号调用则常用来调用类的方法

传入 StartCoroutine 的参数 cor 需要传入一个 迭代器类型 的参数



#### Lua 中实现迭代器：使用协程

先查看C# 下的 IEnumerator 的定义：

![](images\IEnumerator 的定义.webp)

属性 Current：迭代器当前迭代到的元素

接口方法 MoveNext：表示迭代，即将 Current 更新为下一个迭代元素。如果迭代器可以迭代，则返回 true，如果 迭代器迭代元素已经全部被迭代完毕了，则返回 false

接口方法 Reset：重置迭代器。



1、Lua 侧实现的迭代器是可以创建对象的，那么就先使用最基本的 Lua 面向对象写法

```lua
local IEnumerator = {}

function IEnumerator:new()
   	local o = {} 
    self.__index = self
    setmetatable(o, self)
    return o;
end

return IEnumerator;
```

引入：local IEnumerator = require('LuaIEnumerator')

创建对象：IEnumerator:new()



2、MoveNext 的实现：（C#定义迭代器）

![](images\C#定义迭代器.webp)

第一次调用 MoveNext() 迭代器的 Current 会变为 yield return 后的 1，返回 true

第二次调用 Current 会变为 2，返回 true

第三次调用 Current 变为 3， 返回 true

第四次调用 Current 变为 4， 返回 true

第五次调用，因为没有 yield return 了，所有迭代元素已经被迭代完了，返回 false



与 Lua 的 协程是类似的：

1、可以在 Lua 迭代器对象创建时，传入一个 迭代器方法体，方法体内用 coroutine.yield(some) 来表示 yield return some

2、在 MoveNext 中，使用 coroutine.resume(cor) 来进行迭代

```lua
local IEnumerator = {
    func = nil,
    cor = nil,
    Current = nil
}

function IEnumerator:new(func, ...)
    --[[
    ... 是 Lua中的可选参数。
    在定义时无法确定用户传入的方法体 会传入哪些参数，且可选参数不能直接传入到一个方法的调用中，
    如果要传入调用方法，则必须把可选参数“解包” table.unpack
    ]]
   	local o = {} 
    self.__index = self
    setmetatable(o, self)
    
    --[[
    用 param = {...} 来把可选参数再封装一层
    因为用户可能什么参数都不传，此时可选参数为 nil，如果我们用 param = {...} 封装一层，此时 param 仍是一个表，只是表里什么都没有，这会避免 unpack 报错
    ]]
    local param = { ... }
   	o.func = function()
        func(table.unpack(param))
    end
    o.cor = coroutine.create(o.func)
    
    return o;
end

function IEnumerator:MoveNext()
    local code
    --返回两个值
    --第一个布尔值：当前协程是否开启成功 如果协程的代码都执行完了就位false
    --第二个值才是 coroutine.yield(some) 返回出来的值
    code, self.Current = coroutine.resume(self.cor)
    
    --[[
     MoveNext() 返回的是 code，而 code 会在 协程代码都被执行完毕之后，在执行一次协程时 才会为 false，因此用 code 来作为标识 不太准确，并且resume 返回 false 时 是会抛出错误的(尽管如果我们不接收这个错误，该错误就不会显示，且也不影响程序运行)。
    我们可以定义一个 表 move_end，用该表 作为唯一标识，在协程代码执行到最后时，我们返回一个这个表，那么就可以判定 self.Current == move_end 来判定当前是否迭代完毕。 
    ]]
    return code
end

function IEnumerator:Reset()
   	self.cor = coroutine.create(self.func) 
end

return IEnumerator
```



修改后版本：

```lua
local IEnumerator = {
    --Lua 侧这里的 IEnumerator 比 C# 的还多了 func、cor 这两个东西 
    --但其实在C#里，IEnumerator 是个接口，我们可以把我们在 Lua 侧定义的 IEnumerator 理解为 继承了 C# IEnumerator 接口的类，这样，多了一些东西也就没什么了
    func = nil,
    cor = nil,
    Current = nil
}

local move_end = {}

function IEnumerator:new(func, ...)
    --[[
    ... 是 Lua中的可选参数。
    在定义时无法确定用户传入的方法体 会传入哪些参数，且可选参数不能直接传入到一个方法的调用中，
    如果要传入调用方法，则必须把可选参数“解包” table.unpack
    ]]
   	local o = {} 
    self.__index = self
    setmetatable(o, self)
    
    --[[
    用 param = {...} 来把可选参数再封装一层
    因为用户可能什么参数都不传，此时可选参数为 nil，如果我们用 param = {...} 封装一层，此时 param 仍是一个表，只是表里什么都没有，这会避免 unpack 报错
    ]]
    local param = { ... }
   	o.func = function()
        func(table.unpack(param))
		return move_end
    end
    --[[
    使用 coroutine.warp(func) 来创建 “函数式协程”，这也是 Lua 协程创建更为常用的方法。
    使用 coroutine.warp 创建协程，其返回值是一个 方法，我们可以直接调用这个方法，来间接调用 coroutine.resume 且无需关心 resume 返回的 code 值。
    ]]
    --o.cor = coroutine.create(o.func)
    o.cor = coroutine.wrap(o.func)
    
    return o;
end

function IEnumerator:MoveNext()
    local code
    --返回两个值
    --第一个布尔值：当前协程是否开启成功 如果协程的代码都执行完了就位false
    --第二个值才是 coroutine.yield(some) 返回出来的值
    code, self.Current = coroutine.resume(self.cor)
    if self.Current == move_end then
        self.Current = nil
        return false
    else
        return true
    end
    return code
    
    
end

function IEnumerator:Reset()
   	--self.cor = coroutine.create(self.func) 
    self.cor = coroutine.wrap(self.func)
end

return IEnumerator
```

coroutine.wrap() :  wrap()也是用来创建协程的，只不过这个协程的句柄是隐藏的。

跟create()的区别在于：

(1)、wrap()返回的是一个函数，每次调用这个函数相当于调用coroutine.resume()。

(2)、调用这个函数相当于在执行resume()函数。

(3)、调用这个函数时传入的参数，就相当于在调用resume时传入的除协程的句柄外的其他参数。

(4)、调用这个函数时，跟resume不同的是，它并不是在保护模式下执行的，若执行崩溃会直接向外抛出

```lua
co = coroutine.wrap(  function (a,b)
             print("resume args:"..a..","..b)
           yreturn = coroutine.yield()
           print ("yreturn :"..yreturn)
       end
      )
print(type(co))
co(11,22)
co(33)

--[[
  结果如下：

function
resume args:11,22
yreturn :33
]]
```



3、使用迭代器：

**进行自定义 Lua 迭代器 与 C# 迭代器类型的互通**

xLua 自定义 互通类型 的方法：修改器 以及 RawObject 类（xLua-master/Assets/XLua/Src/RawObject.cs）

![img](images\RawObject.webp)

添加 IEnumerator 类后就可以在 Lua 侧，把定义的表转换为 System.Collections.IEnumerator 类型

![](images\添加 IEnumerator 类.webp)

在 Lua 中 使用 CS.XLua.Cast.Int32(o) 来把 o 强转为 Int32 类型：

```lua
local ue = CS.UnityEngine

function start()
    local cor = IEnumator:new(function()
            for i = 1, 10, 1 do
                coroutine.yield(ue.WaitForSeconds(1))
                print('1s')
            end
    end)
    
    --在 Lua 侧，把我们的对象转换为 System.Collections.IEnumerator 类型
    local co = CS.XLua.Cast.IEnumerator(cor)
    self:StartCoroutine(cor)
end

function update()
end

function ondestroy()
end
```



 XLua 提供的 动态添加标签：

（xLua-master/Assets/XLua/Editor/ExampleConfig.cs）

![](images\动态添加标签.webp)

使用最简单的打标签方式，直接在这个类中声明静态的列表

![](images\类中声明静态的列表.webp)

至此，Unity 协程 与 Lua 协程的互通就完毕





## 模块与包



### 模块 和 require



把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。

Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。模块的结构就是一个 table 的结构，因此可以像操作调用 table 里的元素那样来操作调用模块里的常量或函数。



创建自定义模版 module.lua

```lua
-- 文件名为 test.lua
-- 定义一个名为 module 的模块
module = {}
 
-- 定义一个常量
module.constant = "这是一个常量"
 
-- 定义一个函数
function module.func1()
    io.write("这是一个公有函数！\n")
end
 
--程序块的局部变量，即表示一个私有函数，
--因此是不能从外部访问模块里的这个私有函数，必须通过模块里的公有函数来调用
local function func2()
    print("这是一个私有函数！")
end
 
function module.func3()
    func2()
end
 
return module
```



require 函数：用来加载模块

```lua
require("<模块名>")
--或者
require "<模块名>"
```

执行 require 后会返回一个由模块常量或函数组成的 table，并且还会定义一个包含该 table 的全局变量



使用模块：

```lua
-- test.lua 文件
-- module 模块为上文提到到 test.lua
require("test")
print(module.constant)
 
--这里可以调用 因为不是local私有
module.func3()
--直接调用私有方法的时候无法调用
module.func2()
```



给加载的模块定义一个别名变量，方便调用

```lua
local m = require("test")
print(m.constant)
m.func3()
```



自定义的模块文件的存放位置：

函数 require 有它自己的文件路径加载策略，它会尝试从 Lua 文件或 C 程序库中加载模块

require 用于搜索 Lua 文件的路径是存放在全局变量 **package.path** 中，当 Lua 启动后，会以环境变量 **LUA_PATH** 的值来初始这个环境变量。

如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。



自定义设置模块文件加载路径：

在当前用户根目录下打开 .profile 文件（没有则创建，打开 .bashrc 文件也可以），例如把 "~/lua/" 路径加入 LUA_PATH 环境变量里：

```
//文件路径以 ";" 号分隔，最后的 2 个 ";;" 表示新加的路径后面加上原来的默认路径。
#LUA_PATH
export LUA_PATH="~/lua/?.lua;;"
```

接着，更新环境变量参数，使之立即生效。

```
source ~/.profile
```

这时假设 package.path 的值是：

```
/Users/dengjoe/lua/?.lua;
./?.lua;
/usr/local/share/lua/5.1/?.lua;
/usr/local/share/lua/5.1/?/init.lua;
/usr/local/lib/lua/5.1/?.lua;
/usr/local/lib/lua/5.1/?/init.lua
```

那么调用 require("module") 时就会尝试打开以下文件目录去搜索目标。

```
/Users/dengjoe/lua/module.lua;
./module.lua
/usr/local/share/lua/5.1/module.lua
/usr/local/share/lua/5.1/module/init.lua
/usr/local/lib/lua/5.1/module.lua
/usr/local/lib/lua/5.1/module/init.lua
```

如果找过目标文件，则会调用 **package.loadfile** 来加载模块。

否则，就会去找 C 程序库。搜索的文件路径是从全局变量 **package.cpath** 获取，而这个变量则是通过环境变量 LUA_CPATH 来初始。

搜索的策略跟上面的一样，只不过现在换成搜索的是 **so 或 dll 类型**的文件。如果找得到，那么 require 就会通过 **package.loadlib** 来加载它。



### 包（C 为 Lua 写包）

**与Lua中写包不同，C包在使用以前必须首先加载并连接**，在大多数系统中最容易的实现方式是通过**动态连接库**机制。

Lua在一个叫**loadlib的函数**内提供了所有的动态连接的功能。

这个函数有两个参数:

​		库的绝对路径 和 初始化函数

所以典型的调用的例子如下:

```lua
local path = "/usr/local/lua/lib/libluasocket.so"
local f = loadlib(path, "luaopen_socket")
```

loadlib 函数加载指定的库并且连接到 Lua，然而它并不打开库（也就是说没有调用初始化函数），反之他返回初始化函数作为 Lua 的一个函数，这样我们就可以直接在Lua中调用他。

如果加载动态库或者查找初始化函数时出错，loadlib 将返回 nil 和错误信息。我们可以修改前面一段代码，使其检测错误然后调用初始化函数：

```lua
local path = "/usr/local/lua/lib/libluasocket.so"
-- 或者 path = "C:\\windows\\luasocket.dll"，这是 Window 平台下
local f = assert(loadlib(path, "luaopen_socket"))
f()  -- 真正打开库
```

一般情况下我们期望二进制的发布库包含一个与前面代码段相似的 stub 文件，安装二进制库的时候可以随便放在某个目录，只需要修改 stub 文件对应二进制库的实际路径即可。

将 stub 文件所在的目录加入到 LUA_PATH，这样设定后就可以使用 require 函数加载 C 库了。



# xLua：基于 IL代码注入热更



C# 编译型语言 脚本打包成动态链接库DLL Android √	IOS ×

Lua 解释型语言 脚本运行时编译 Android √	IOS √



# xLua插件



![](C:\Users\TenderCoffee\Desktop\Unity热更新\Unity热更代码\.笔记\images\Snipaste_2023-02-04_21-37-50.png)



xLua 插件：在Unity项目里引入Lua语言支持

**1、提供相关语言底层支持，Lua虚拟机启动时候创建**

**2、提供 Mono调用 Lua 的能力**

**3、提供 Lua 调用 Mono 的能力**

4、提供 hotfix 功能 - HotFix 热补丁

	1. 开发时使用C#
	1. 打开hotfix标签
	1. 出现问题时使用lua修复
	1. 大版本更新时将lua部分更新回C#

5、等等



## xLua 原理：新旧字节码的比对和替换



使用C#开发的Lua虚拟机，使用JIT（即时编译）编译器
Lua 和 C# 相互通信，支持Lua热更新
使用新的字节码覆盖或者替换掉当前运行时的字节码，实现状态的无缝更新
1、使用动态编译的方式根据需要将 Lua 源代码编译成汇编的字节码，在运行时进行动态加载
2、热更新本质上是将这些字节码内容由旧覆盖成新的
3、xLua内置了热更新框架可以对比新旧字节码之间的差异，自动执行更新操作从而实现热更新
4、关键是，在处理完成热更新后，xLua通过比较模版的版本，字节码的大小和偏移，在不加载完整字节码的前提下，还能做出非常有效的补丁处理



### 1、在IL层注入代码的前后对比

原来的类：

```c#
public class TestXLua
{
    public int Add(int a, int b)
    {
        return a - b;
    }
}
```

在IL层面为其注入代码后：

```c#
public class TestXLua
{
    static Func<object, int, int, int> hotfix_Add = null;
    int Add(int a, int b)
    {
        if (hotfix_Add != null) return hotfix_Add(this, a, b);
        return a - b;
    }
}
```



### 2、IL层注入代码：通过Lua编写补丁

​	使hotfix_Add指向一个lua的适配函数，从而达到替换原C#函数，实现更新的目的。



#### xLua插件热更的4个步骤

```none
1、打开该特性

添加HOTFIX_ENABLE宏，（在Unity3D的File->Build Setting->Scripting Define Symbols下添加）。编辑器、各手机平台这个宏要分别设置！如果是自动化打包，要注意在代码里头用API设置的宏是不生效的，需要在编辑器设置。

2、执行XLua/Generate Code菜单。

3、注入，构建手机包这个步骤会在构建时自动进行，编辑器下开发补丁需要手动执行"XLua/Hotfix Inject In Editor"菜单。打印“hotfix inject finish!”或者“had injected!”才算成功，否则会打印错误信息。

4、使用xlua.hotfix或util.hotfix_ex打补丁
```



##### 1) 打开HOTFIX_ENABLE宏

**HOTFIX_ENABLE** 是xLua定义启用热更的一个宏，添加这个宏主要有两个作用

1. 在编辑器中出现 "XLua/Hotfix Inject In Editor" 菜单，通过该菜单可以手动执行代码注入

2. 利用HOTFIX_ENABLE进行了条件编译，定义了一些只有使用热更时才需要的方法。

   例如DelegateBridge.cs中的部分方法，这些方法会在针对泛型方法进行IL注入时用到
   
   ```csharp
   // DelegateBridge.cs
   #if HOTFIX_ENABLE
       private int _oldTop = 0;
       private Stack<int> _stack = new Stack<int>();
       public void InvokeSessionStart()
       {
           // ...
       }
       public void Invoke(int nRet)
       {
           // ...
       }
       public void InvokeSessionEnd()
       {
           // ...
       }
       // ...
   #endif
   ```

##### 2) 生成代码

为 **标记有Hotfix特性** 的方法生成对应的匹配函数

```c#
// 测试用 TestXLua.cs
[Hotfix]
public class TestXLua
{
    public int Add(int a, int b)
    {
        return a - b;  // 这里的Add方法故意写成减法，后面通过热更新修复
    }
}
```

手动执行代码注入后，会在配置的Gen目录下生成DelegatesGensBridge.cs文件，其中有对应的匹配函数 __Gen_Delegate_Imp1，**通过生成对应的匹配函数，xLua就可以通过将该C#函数替换成Lua函数来实现热更**

```c#
// DelegatesGensBridge.cs
public partial class DelegateBridge : DelegateBridgeBase
{
    // 为TestXLua.Add生成的对应的匹配函数
    /*
    xLua就是通过将该C#函数替换成Lua函数来实现热更的。
    也就是说，热更后就会出现C#调用Lua函数的情况，而C#想要调用Lua函数，就需要用到生成的匹配函数。
    */
    public int __Gen_Delegate_Imp1(object p0, int p1, int p2)
    {
#if THREAD_SAFE || HOTFIX_ENABLE
        lock (luaEnv.luaEnvLock)
        {
#endif
            RealStatePtr L = luaEnv.rawL;
            int errFunc = LuaAPI.pcall_prepare(L, errorFuncRef, luaReference);
            ObjectTranslator translator = luaEnv.translator;
            translator.PushAny(L, p0);
            LuaAPI.xlua_pushinteger(L, p1);
            LuaAPI.xlua_pushinteger(L, p2);
            
            PCall(L, 3, 1, errFunc);
            
            
            int __gen_ret = LuaAPI.xlua_tointeger(L, errFunc + 1);
            LuaAPI.lua_settop(L, errFunc - 1);
            return  __gen_ret;
#if THREAD_SAFE || HOTFIX_ENABLE
        }
#endif
    }
    // ...
}
```

调用传递给C#的Lua函数时：

​		相当于调用以"**__Gen_Delegate_Imp**"开头的生成函数，这个生成函数负责参数压栈，并通过保存的索引获取到真正的Lua function，然后使用lua_pcall完成Lua function的调用



##### 3) 注入

"XLua/Hotfix Inject In Editor"菜单 触发 **HotfixInject**方法

内部再通过xLua提供的工具 **XLuaHotfixInject.exe** 来完成代码注入

同时IL代码注入需要用到 Mono.Cecil库，这样也避免了每个项目都要额外集成这个库

```c#
// Hotfix.cs
[MenuItem("XLua/Hotfix Inject In Editor", false, 3)]
public static void HotfixInject()
{
    HotfixInject("./Library/ScriptAssemblies");
}

// 最终实际完成代码注入 HotfixInject
// injectAssemblyPath :要注入的程序集 例如./Library/ScriptAssemblies\Assembly-CSharp.dll
// xluaAssemblyPath表示LuaEnv所在程序集的完全限定路径，一般情况下和injectAssemblyPath相同
// HotfixInject的主要任务是遍历injectAssembly中的所有类型，通过InjectType依次对它们进行代码注入
public static void HotfixInject(string injectAssemblyPath, string xluaAssemblyPath, IEnumerable<string> searchDirectorys, string idMapFilePath, Dictionary<string, int> hotfixConfig)
{
    AssemblyDefinition injectAssembly = null;
    AssemblyDefinition xluaAssembly = null;
    // ...
    injectAssembly = readAssembly(injectAssemblyPath);
    
    // injected flag check
    if (injectAssembly.MainModule.Types.Any(t => t.Name == "__XLUA_GEN_FLAG"))
    {
        Info(injectAssemblyPath + " had injected!");
        return;
    }
    // 添加一个新的类型定义，以标记已注入
    injectAssembly.MainModule.Types.Add(new TypeDefinition("__XLUA_GEN", "__XLUA_GEN_FLAG", ILRuntime.Mono.Cecil.TypeAttributes.Class,
        injectAssembly.MainModule.TypeSystem.Object));

    xluaAssembly = (injectAssemblyPath == xluaAssemblyPath || injectAssembly.MainModule.FullyQualifiedName == xluaAssemblyPath) ? 
        injectAssembly : readAssembly(xluaAssemblyPath);

    Hotfix hotfix = new Hotfix();
    hotfix.Init(injectAssembly, xluaAssembly, searchDirectorys, hotfixConfig);

    //var hotfixDelegateAttributeType = assembly.MainModule.Types.Single(t => t.FullName == "XLua.HotfixDelegateAttribute");
    var hotfixAttributeType = xluaAssembly.MainModule.Types.Single(t => t.FullName == "XLua.HotfixAttribute");
    var toInject = (from module in injectAssembly.Modules from type in module.Types select type).ToList();  // injectAssembly中的各个类型
    foreach (var type in toInject)
    {
        //遍历指定类型的所有方法（根据HotfixFlag会做一些过滤），依次对它们进行代码注入
        if (!hotfix.InjectType(hotfixAttributeType, type))
        {
            return;
        }
    }
    Directory.CreateDirectory(Path.GetDirectoryName(idMapFilePath));
    hotfix.OutputIntKeyMapper(new FileStream(idMapFilePath, FileMode.Create, FileAccess.Write));
    File.Copy(idMapFilePath, idMapFilePath + "." + DateTime.Now.ToString("yyyyMMddHHmmssfff"));
    // 写入对程序集的修改
    writeAssembly(injectAssembly, injectAssemblyPath);
    Info(injectAssemblyPath + " inject finish!");
    // ...
}

// Hotfix.cs
//遍历指定类型的所有方法（根据HotfixFlag会做一些过滤），依次对它们进行代码注入。
//注入方法有两个:
//方式一：是针对泛型方法的injectGenericMethod
//方式二：是针对普通方法的injectMethod（对于要注入的方法method，先找到与其相匹配的以__Gen_Delegate_Imp开头的生成方法，然后通过IL操作为method所在类添加一个DelegateBridge类型的静态变量（变量名通过getDelegateName方法获得）。并在method方法头部插入IL指令逻辑：判断静态变量是否不为空，如果不为空，则调用DelegateBridge变量的以__Gen_Delegate_Imp开头的生成方法并直接返回不再执行原逻辑。这个生成方法在打补丁后对应的就是Lua函数。）
bool injectMethod(MethodDefinition method, HotfixFlagInTool hotfixType)
{
    var type = method.DeclaringType;  // 方法所在类
    bool isFinalize = (method.Name == "Finalize" && method.IsSpecialName);
    MethodReference invoke = null;
    int param_count = method.Parameters.Count + (method.IsStatic ? 0 : 1);
    if (!findHotfixDelegate(method, out invoke, hotfixType))  // 找到与method匹配的生成方法，以__Gen_Delegate_Imp开头的
    {
        Error("can not find delegate for " + method.DeclaringType + "." + method.Name + "! try re-genertate code.");
        return false;
    }
    if (invoke == null)
    {
        throw new Exception("unknow exception!");
    }
#if XLUA_GENERAL
    invoke = injectAssembly.MainModule.ImportReference(invoke);
#else
    invoke = injectAssembly.MainModule.Import(invoke);
#endif
    FieldReference fieldReference = null;
    VariableDefinition injection = null;
    // IntKey是xLua的标志位，可以控制不生成静态字段，而是把所有注入点放到一个数组集中管理。这里可以先忽略，主要看 is not IntKey的逻辑
    bool isIntKey = hotfixType.HasFlag(HotfixFlagInTool.IntKey) && !type.HasGenericParameters && isTheSameAssembly;  
    //isIntKey = !type.HasGenericParameters;
    if (!isIntKey)
    {
        injection = new VariableDefinition(invoke.DeclaringType);  // 新创建一个XLua.DelegateBridge类型的变量
        method.Body.Variables.Add(injection);

        var luaDelegateName = getDelegateName(method);  // 获取将要添加的静态变量的名称，这个静态变量用于保存Lua补丁设置的方法
        if (luaDelegateName == null)
        {
            Error("too many overload!");
            return false;
        }

        FieldDefinition fieldDefinition = new FieldDefinition(luaDelegateName, ILRuntime.Mono.Cecil.FieldAttributes.Static | ILRuntime.Mono.Cecil.FieldAttributes.Private,
            invoke.DeclaringType);  // 创建一个静态XLua.DelegateBridge变量，用于保存Lua补丁设置的方法
        type.Fields.Add(fieldDefinition);  // 给type添加一个luaDelegateName静态字段，这个字段值在调用xlua.hotfix时会赋值
        fieldReference = fieldDefinition.GetGeneric();
    }

    bool ignoreValueType = hotfixType.HasFlag(HotfixFlagInTool.ValueTypeBoxing);

    var insertPoint = method.Body.Instructions[0];
    // //获取IL处理器
    var processor = method.Body.GetILProcessor();

    if (method.IsConstructor)
    {
        insertPoint = findNextRet(method.Body.Instructions, insertPoint);  // 获取到下一个Ret指令
    }

    Dictionary<Instruction, Instruction> originToNewTarget = new Dictionary<Instruction, Instruction>();
    HashSet<Instruction> noCheck = new HashSet<Instruction>();

    // 真正的IL代码注入逻辑。通过Mono.Cecil库的API插入一些IL指令
    while (insertPoint != null)
    {
        Instruction firstInstruction;
        if (isIntKey)
        {
            // ...
        }
        else
        {
            firstInstruction = processor.Create(OpCodes.Ldsfld, fieldReference);  // 加载静态域fieldReference，即luaDelegateName字段
            processor.InsertBefore(insertPoint, firstInstruction);
            processor.InsertBefore(insertPoint, processor.Create(OpCodes.Stloc, injection));  // 存储本地变量，将injection变量的值设置为luaDelegateName字段的值
            processor.InsertBefore(insertPoint, processor.Create(OpCodes.Ldloc, injection));  // 加载本地变量
        }

        // Brfalse表示栈上的值为 false/null/0 时发生跳转，如果injection的值为空，就调转到insertPoint，那通过InsertBefore插入的指令就都会被跳过了
        var jmpInstruction = processor.Create(OpCodes.Brfalse, insertPoint);  
        processor.InsertBefore(insertPoint, jmpInstruction);

        if (isIntKey)
        {
            // ...
        }
        else
        {
            processor.InsertBefore(insertPoint, processor.Create(OpCodes.Ldloc, injection));  // 再加载一次injection的值
        }
        // 加载参数
        for (int i = 0; i < param_count; i++)
        {
            if (i < ldargs.Length)
            {
                processor.InsertBefore(insertPoint, processor.Create(ldargs[i]));  // 加载第i个参数
            }
            else if (i < 256)
            {
                processor.InsertBefore(insertPoint, processor.Create(OpCodes.Ldarg_S, (byte)i));
            }
            else
            {
                processor.InsertBefore(insertPoint, processor.Create(OpCodes.Ldarg, (short)i));
            }
            if (i == 0 && !method.IsStatic && type.IsValueType)
            {
                processor.InsertBefore(insertPoint, processor.Create(OpCodes.Ldobj, type));  // 加载对象
            }
            // ...
        }

        // 插入方法调用指令
        processor.InsertBefore(insertPoint, processor.Create(OpCodes.Call, invoke));  // 调用injection处值（DelegateBridge对象）的方法invoke（__Gen_Delegate_Imp开头的方法）

        if (!method.IsConstructor && !isFinalize)
        {
            processor.InsertBefore(insertPoint, processor.Create(OpCodes.Ret));  // 插入返回指令
        }

        if (!method.IsConstructor)
        {
            break;
        }
        else
        {
            originToNewTarget[insertPoint] = firstInstruction;
            noCheck.Add(jmpInstruction);
        }
        insertPoint = findNextRet(method.Body.Instructions, insertPoint);
    }
    // ...
}


```



##### 4）打补丁

通过 **xlua.hotfix（定义在LuaEnv） 或 xlua.hotfix_ex** 将C#函数逻辑替换成Lua函数

```lua
-- lua测试文件
xlua.hotfix(CS.TestXLua, "Add", function(self, a, b)
    return a + b  -- 修复成正确的加法
end)
```

```lua
-- LuaEnv
-- cs表示要修复的类，field表示要修复的变量名，func表示对应的修复函数
xlua.hotfix = function(cs, field, func)
    -- 1. 先根据一定规则计算得到真正的C#变量名
    -- TestXLua.Add => __Hotfix()_Add
    -- 2. 通过xlua.access方法为这个变量设置对应的Lua修复函数
    if func == nil then func = false end
    local tbl = (type(field) == 'table') and field or {[field] = func}
    for k, v in pairs(tbl) do
        local cflag = ''
        if k == '.ctor' then
            cflag = '_c'
            k = 'ctor'
        end
        local f = type(v) == 'function' and v or nil
        -- cflag .. '__Hotfix0_'..k 对应了前面C#代码中的 luaDelegateName
        xlua.access(cs, cflag .. '__Hotfix0_'..k, f) -- at least one
        pcall(function()
            for i = 1, 99 do
                xlua.access(cs, cflag .. '__Hotfix'..i..'_'..k, f)
            end
        end)
    end
    xlua.private_accessible(cs)
end
```

```c#
// xlua.access实际上调用的是StaticLuaCallbacks.cs的XLuaAccess方法
// StaticLuaCallbacks.cs
// 设置或访问指定字段的值
/*
主要看设置字段值的部分，参数数量大于2（参数1类型，参数2字段名，参数3要设置的值），就表示是要设置字段值。
xlua.hotfix通过XLuaAccess是为"__Hotfix0_Add"静态字段设置了一个Lua函数，在C#中这个Lua函数对应的是DelegateBridge对象（其内部保存着Lua函数的索引），这也是为什么前面IL注入时是为类添加一个DelegateBridge类型的静态变量
*/
public static int XLuaAccess(RealStatePtr L)
{
    try
    {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        Type type = getType(L, translator, 1);  // 获取第一个参数的类型
        object obj = null;
        if (type == null && LuaAPI.lua_type(L, 1) == LuaTypes.LUA_TUSERDATA)
        {
            obj = translator.SafeGetCSObj(L, 1);
            if (obj == null)
            {
                return LuaAPI.luaL_error(L, "xlua.access, #1 parameter must a type/c# object/string");
            }
            type = obj.GetType();
        }

        if (type == null)
        {
            return LuaAPI.luaL_error(L, "xlua.access, can not find c# type");
        }

        string fieldName = LuaAPI.lua_tostring(L, 2);

        BindingFlags bindingFlags = BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance | BindingFlags.Static;

        if (LuaAPI.lua_gettop(L) > 2) // set  设置字段值
        {
            // 设置字段（参数2）值为参数3
            var field = type.GetField(fieldName, bindingFlags);
            if (field != null)
            {
                field.SetValue(obj, translator.GetObject(L, 3, field.FieldType));
                return 0;
            }
            var prop = type.GetProperty(fieldName, bindingFlags);
            if (prop != null)
            {
                prop.SetValue(obj, translator.GetObject(L, 3, prop.PropertyType), null);
                return 0;
            }
        }
        else
        {
            // 获取字段（参数2）值
            var field = type.GetField(fieldName, bindingFlags);
            if (field != null)
            {
                translator.PushAny(L, field.GetValue(obj));
                return 1;
            }
            var prop = type.GetProperty(fieldName, bindingFlags);
            if (prop != null)
            {
                translator.PushAny(L, prop.GetValue(obj, null));
                return 1;
            }
        }
        return LuaAPI.luaL_error(L, "xlua.access, no field " + fieldName);  // 没有找到fieldName字段，抛出异常
    }
    catch (Exception e)
    {
        return LuaAPI.luaL_error(L, "c# exception in xlua.access: " + e);
    }
}
```



## xLua 整个热更过程总结

TestXLua.Add函数

1. 先通过 **Generate Code** 为TestXLua.Add生成与其声明相同的匹配函数"__Gen_Delegate_Imp1"
2. 匹配函数 生成在 **DelegateBridge 类** 从而 Lua 可以被传递到 C# 中
3. 通过 **IL 代码注入** 为 TestXLua.Add函数 生成一个 "__Hotfix0_Add" 的**DelegateBridge类型的静态变量**
4. TestXLua.Add 方法中注入判断 静态变量是否不为空，如果不为空就调用静态变量所对应的Lua方法的逻辑
5. 打补丁时通过 xlua.hotfix 为静态变量"**__Hotfix0_Add**"设置一个Lua函数。这样下次调用TestXLua.Add时，"____Hotfix0_Add"将不为空，此时将执行_Hotfix0_Add.__Gen_Delegate_Imp1，即调用设置的Lua函数，而不再执行原有逻辑



反编译代码如下：

```c#
using System;
using XLua;

// Token: 0x02000016 RID: 22
[Hotfix(HotfixFlag.Stateless)]
public class TestXLua
{
	// Token: 0x06000051 RID: 81 RVA: 0x00002CE0 File Offset: 0x00000EE0
	public int Add(int a, int b)
	{
		DelegateBridge _Hotfix0_Add = TestXLua.__Hotfix0_Add;
		if (_Hotfix0_Add != null)
		{
			return _Hotfix0_Add.__Gen_Delegate_Imp1(this, a, b);
		}
		return a - b;
	}

	// Token: 0x06000052 RID: 82 RVA: 0x00002D14 File Offset: 0x00000F14
	public TestXLua()
	{
		DelegateBridge c__Hotfix0_ctor = TestXLua._c__Hotfix0_ctor;
		if (c__Hotfix0_ctor != null)
		{
			c__Hotfix0_ctor.__Gen_Delegate_Imp2(this);
		}
	}

	// Token: 0x04000022 RID: 34
	private static DelegateBridge __Hotfix0_Add;

	// Token: 0x04000023 RID: 35
	private static DelegateBridge _c__Hotfix0_ctor;
}
```



## xLua 使用

### xLua API

#### 1. 执行字符串（不建议）

LuaEnv.DoString("print('hello world')")



#### 2、加载 Lua 文件（require本事是调loader去加载）

LuaEnv.DoString(“require 'byfile'")



#### 3、自定义 loader 

```c#
//lua代码里头调用require时，参数将会透传给回调，回调中就可以根据这个参数去加载指定文件，如果需要支持调试，需要把filepath修改为真实路径传出
//该回调返回值是一个byte数组，如果为空表示该loader找不到，否则则为lua文件的内容
public delegate byte[] CustomLoader(ref string filepath);
//AddLoader可以注册个回调，该回调参数是字符串
public void LuaEnv.AddLoader(CustomLoader loader)；
```



### 文件配置

#### 1、打标签 - 白名单 方便但不建议使用

```c#
//从 Lua 调用 C# 的某个类
[LuaCallCSharp]
public class A
{
    
}
```

优点：使用方便，但 IL2CPP 增加代码量

1、**[XLua.LuaCallCSharp]**：为一个C#类型增加这个标签后，xLua 会生成这个类型的适配代码（否则会尝试用性能较低的反射来访问），提供给 Lua 调用

2、**[XLua.CSharpCallLua]**：C# 调用 Lua 文件中的逻辑

3、**[XLua.Hotfix]**：标记需要热更的文件



#### 2、静态列表

系统API 或 其他没有源码的库，实例化的泛化类型，因为无法直接打标签

所以需要 **在Editor目录下（建议） 在一个静态类里 声明 一个静态字段**

该字段的类型除BlackList和AdditionalProperties之外只要实现了IEnumerable<Type>就可以了

```c#
//Editor文件夹下
//需要放到一个静态类里
public static class A
{	
    [LuaCallCSharp]
    public static List<Type> mymodule_lua_call_cs_list = new List<Type>()
    {
        typeof(GameObject),
        typeof(Dictionary<string, int>),
    };    
}
```



#### 3、动态列表

需要 **在Editor目录下（建议） 在一个静态类里 声明 一个静态属性**，打上 Hotfix标签

```c#
//Editor文件夹下
//需要放到一个静态类里
public static class A
{	
    [Hotfix]
    public static List<Type> by_property
    {
        get{
         	return (from type in Assembly.Load("Assembly-CSharp").GetTypes() where type.Namespace == "xxxx" select type).ToList();   
        }
    }    
}
```



### Lua 与 C# 交互

#### C# 访问 Lua

1、全局基本数据类型 LuaEnv.Global.Get

```lua
luaenv.Global.Get<int>("a")
luaenv.Global.Get<string>("b")
luaenv.Global.Get<bool>("c")
```



2、访问全局table

	1. 映射到普通的 class 或者 struct - 值拷贝，复杂的 class代价大，修改字段的值不会和lua中的table彼此同步
	1. **映射到一个 interface - 依赖代码生成器**
	1. 映射到 Dictionary<> List<> - 如果 table下key和 value 类型一致，则更轻量
	1. 映射到 LuaTable 类 - 不需要代码生成，但是慢很多



3、访问一个全局的 function

 1. **映射到 delegate （建议的方式，性能好，类型安全，缺点是需要代码生成）**

    delegate 的声明：

    ​	function 的每个参数：声明一个输入类型的参数

    ​	多返回值：从左到右映射到C#输出参数（返回值，out参数，ref参数）

    ​	参数、返回值类型：都支持

 2. 映射到 LuaFunction

​			优缺点和第一种相反

​			使用简单：LuaFunction上有个变参的Call函数，可以传任意类型，任意个数的参数，返回值是object的数组，对应于lua的多返回值



#### Lua 调用 C#

1、new C# 对象

C# new 一个对象：

```c#
var newGameObj = new UnityEngine.GameObject();
```

Lua new 一个对象：（没有new关键字）

所有C#相关的都放到CS下，包括构造函数，静态成员属性、方法

```lua
local newGameObj = CS.UnityEngine.GameObject()

-- 有多个构造函数 xlua支持重载
local newGameObj2 = CS.UnityEngine.GameObject('helloworld')
```



2、Lua 访问 C# 静态属性、方法

```lua
-- 读静态属性
CS.UnityEngine.Time.deltaTime
-- 写静态属性
CS.UnityEngine.Time.timeScale = 0.5

-- 调用静态方法
CS.UnityEngine.GameObject.Find('helloworld')
```

如果需要经常访问的类，可以先用局部变量引用后访问，能提高性能

```lua
local GameObject = CS.UnityEngine.GameObject
GameObject.Find('helloworld')
GameObject.Find('helloworld2')
GameObject.Find('helloworld3')
```



3、Lua 访问 C# 成员属性，方法

```lua
-- 读成员属性
testobj.DMF

-- 写成员属性
testobj.DMF = 1024

-- 调用成员方法 
-- 注意：调用成员方法，第一个参数需要传该对象，建议用冒号语法糖
testobj:DMFunc()
```



4、Lua 访问 C# 父类属性，方法

xlua支持（通过派生类）访问基类的静态属性，静态方法，（通过派生类实例）访问基类的成员属性，成员方法。



5、Lua 的 参数的输入输出属性（out，ref）

Lua调用测的参数处理规则：

​		C#的普通参数算一个输入形参，ref修饰的算一个输入形参，out修饰的不算，然后从左往右对应lua 调用测的实参列表；



Lua调用测的返回值处理规则：

​		C#函数的返回值（如果有的话）算一个返回值，out算一个返回值，ref算一个返回值，然后从左往右对应lua的多返回值。



6、Lua 重载方法 - 不建议重载因为有一对多的情况

```lua
-- 直接通过不同的参数类型进行重载函数的访问，例如：

-- 访问整数参数的TestFunc
testobj:TestFunc(100)

-- 访问字符串参数的TestFunc。
testobj:TestFunc('hello')

```

注意：xlua只有一定程度上支持重载函数的调用，因为lua的类型远远不如C#丰富，**存在一对多的情况**，比如C#的int，float，double都对应于lua的number，上面的例子中TestFunc如果有这些重载参数，第一行将无法区分开来，只能调用到其中一个（生成代码中排前面的那个）。



7、操作符

支持的操作符有：+，-，*，/，==，一元 -，<，<=， %，[]



8、参数带默认值的方法

如果所给的实参少于形参，则会用默认值补上



9、可变参数方法

对于C#的如下方法：

```c#
void VariableParamsFunc(int a, params string[] strs)
```

可以在lua里头这样调用：

```lua
testobj:VariableParamsFunc(5, 'hello', 'john')
```



10、使用 扩展方法

在C#里定义了，lua里就能直接使用



11、泛化（模版）方法

不直接支持，可以通过 扩展方法 功能进行封装后调用



12、枚举类型

枚举值就像枚举类型下的静态属性

```lua
-- 参数：Tutorial.TestEnum类型
testobj:EnumTestFunc(CS.Tutorial.TestEnum.E1)
```

枚举类加入到生成代码，枚举类将支持__CastFrom方法，可以实现从一个整数或者字符串到枚举值的转换

```lua
-- 从整数到枚举值
CS.Tutorial.TestEnum.__CastFrom(1)
-- 从字符串到枚举值
CS.Tutorial.TestEnum.__CastFrom(‘E1')
```



13、delegate 的使用（调用 + -）

C#的delegate调用：和调用普通lua函数一样

+操作符：对应C#的+操作符，把两个调用串成一个调用链，右操作数可以是同类型的 C#delegate或者是lua函数。

-操作符：把一个delegate从调用链中移除。

Ps：delegate属性可以用一个luafunction来赋值。



14、event

比如 testobj 里头有个事件定义是这样：

```c#
public event Action TestEvent;
```

增加事件回调

```lua
testobj:TestEvent('+', lua_event_callback)
```

移除事件回调

```lua
testobj:TestEvent('-', lua_event_callback)
```



15、C# 复杂类型 和 table 的自动转换

对于一个有无参构造函数的C#复杂类型，在lua侧可以直接用一个table来代替，该table对应复杂类型的public字段有相应字段即可，支持函数参数传递，属性赋值等，例如：

C#下B结构体（class也支持）定义如下：

```c#
public struct A{ public int a;}

public struct B{ public A b; public double c;}
```

某个类有成员函数如下：

```c#
void Foo(B b)
```

在lua可以这么调用：

```lua
obj:Foo({b = {a = 100}, c = 200})
```

获取类型（相当于C#的typeof）

比如要获取UnityEngine.ParticleSystem类的Type信息，可以这样

```lua
typeof(CS.UnityEngine.ParticleSystem)
```



16、强转

告诉xlua要用指定的生成代码去调用一个对象

用途：有的时候第三方库对外暴露的是一个interface或者抽象类，实现类是隐藏的，这样我们无法对实现类进行代码生成。

该实现类将会被xlua识别为未生成代码而用反射来访问，如果这个调用是很频繁的话还是很影响性能的，这时我们就可以**把这个interface或者抽象类加到生成代码，然后指定用该生成代码来访问**：

```c#
cast(calc, typeof(CS.Tutorial.Calc))
```

上面就是指定用 CS.Tutorial.Calc 的生成代码来访问calc对象。



#### **C# 与 Lua 交互的原理**

1、C#调用Lua：

​		C#先调用Lua的dll文件（C语言写的库），dll文件执行Lua代码



2、Lua调用C#：

1. Wrap方式：非反射机制，需要为源文件生成相应的wrap文件，当启动 Lua虚拟机时，Wrap文件将会被自动注册到Lua虚拟机中，之后，Lua文件将能够识别和调用C#源文件。

   总结：Lua调用Wrap文件， Wrap文件调用C#源文件

2. 反射机制



### xLua 热更新



#### 使用方式

1、添加 **HOTFIX_ENABLE 宏**打开该特性（在Unity3D的File->Build Setting->Scripting Define Symbols下添加）。编辑器、各手机平台这个宏要分别设置！如果是自动化打包，要注意在代码里头用API设置的宏是不生效的，需要在编辑器设置。

（建议平时开发业务代码不打开HOTFIX_ENABLE，只在build手机版本或者要在编译器下开发补丁时打开HOTFIX_ENABLE）

2、执行**XLua/Generate Code**菜单。

3、注入，构建手机包这个步骤会在构建时自动进行，**编辑器下开发补丁需要手动执行"XLua/Hotfix Inject In Editor"菜单**。注入成功会打印“hotfix inject finish!”或者“had injected!”。



#### 约束

不支持静态构造函数，目前只支持Assets下代码的热补丁，不支持引擎，c#系统库的热补丁。



#### xLua 热补丁 API

（1）xlua.hotfix(class, [method_name], fix)

- - 描述 ： 注入lua补丁
  - class ： C#类，两种表示方法，
    - CS.Namespace.TypeName或者字符串方式"Namespace.TypeName"，字符串格式和C#的Type.GetType要求一致，
    - 如果是内嵌类型（Nested Type）是非Public类型的话，只能用字符串方式表示"Namespace.TypeName+NestedTypeName"；
  - method_name ： 方法名，可选；
  - fix ： 如果传了method_name，fix将会是一个function，否则通过table提供一组函数。table的组织按key是method_name，value是function的方式。



base(csobj)

- - 描述 ： 子类override函数通过base调用父类实现。
  - csobj ： 对象
  - 返回值 ： 新对象，可以通过该对象base上的方法

```lua
xlua.hotfix(CS.BaseTest, 'Foo', function(self, p)
   print('BaseTest', p) 
   base(self):Foo(p)
end)
```



（2）util.hotfix_ex(class, method_name, fix)

- - 描述 ： xlua.hotfix的增强版本，可以在fix函数里头执行原来的函数，缺点是fix的执行会略慢。
  - method_name ： 方法名；
  - fix ： 用来替换C#方法的lua function。



#### Hotfix 标签

Hotfix标签可以设置一些标志位对生成代码及插桩定制化。

- - Stateless、Stateful

遗留设置，Stateful方式在新版本已经删除，因为这种方式可以用xlua.util.hotfix_state接口达到类似的效果。

- - ValueTypeBoxing

值类型的适配delegate会收敛到object，好处是代码量更少，不好的是值类型会产生boxing及gc，适用于对text段敏感的业务。

- - IgnoreProperty

不对属性注入及生成适配代码，一般而言，大多数属性的实现都很简单，出错几率比较小，建议不注入。

- - IgnoreNotPublic

不对非public的方法注入及生成适配代码。除了像MonoBehaviour那种会被反射调用的私有方法必须得注入，其它仅被本类调用的非public方法可以不注入，只不过修复时会工作量稍大，所有引用到这个函数的public方法都要重写。

- - Inline

不生成适配delegate，直接在函数体注入处理代码。

- - IntKey

不生成静态字段，而是把所有注入点放到一个数组集中管理。



## xLua 使用建议

- - 对所有较大可能变动的类型加上Hotfix标识；
  - 建议用反射找出所有函数参数、字段、属性、事件涉及的delegate类型，标注CSharpCallLua；
  - 业务代码、引擎API、系统API，需要在Lua补丁里头高性能访问的类型，加上 LuaCallCSharp

- - 引擎API、系统API可能被代码剪裁调（C#无引用的地方都会被剪裁），如果觉得可能会新增C#代码之外的API调用，这些API所在的类型要么加LuaCallCSharp，要么 反射使用；



# xLua 开发框架

 	1. UI 与功能支持模块
 	2. Net 网络支持模块
 	3. GameMainData 数据支持模块
 	4. StaticData 配置文件支持模块
 	5. Event 事件支持模块
 	6. List 列表支持模块
 	7. Public 公共业务方法支持模块
 	8. Tools 公共非业务方法支持模块
 	9. Plugin 插件支持模块





