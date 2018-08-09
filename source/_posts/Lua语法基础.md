---
title: Lua语法基础
date: 2018-08-09 17:30:30
tags: 
	- lua
	- redis
categories: lua
description: 主要还是为了redis,内容来自:http://www.runoob.com/lua/lua-tutorial.html
---
#### 常用语法
- 单行注释: 两个减号
- 多行注释: 
```
--[[
 多行注释
 多行注释
]]--
```
- 标示符: 不允许使用@$%等特殊字符,并且区分大小写
- 关键字: 与一般与语言差别不大,特别的有until,elseif
- 全局变量: 默认情况下,变量总是全局变量,变量不需要声明,没有初始化也不会错,只不过得到的结果是nil,删除一个全局变量,只需要给它赋值nil
- 数据类型:
```
nil 表示一个无效值
boolean 布尔类型:true/false
number 表示双精度类型的实浮点数
string 字符串由一对双引号或者单引号来表示
function 由C或者Lua编写的函数
userdata 表示任意存储在变量中的C数据结构
thread 表示执行的独立线路,用于执行协同程序
table Lua中的表其实是一个"关联数组",表的创建通过"构造表达式"来完成
```
- nil 打印一个没有赋值的变量,会输出nil
    - 对于table和全局变量,nil有删除的作用
    - nil比较时应该加上双引号: `type(x)=="nil"`  输出true
- boolean 布尔
```lua
if false or nil then
    print("至少一个是true")
else
    print("false和nil都为false")
end
```
- **string 字符串类型**
    - 可以用一对双引号或者单引号来表示
    - 也可以用两个方括号来表示"一块"字符串
    - 在对一个数字字符串进行运算时,lua会尝试将这个数字字符串转为数字
        - `print("2" + 6)`&rArr;8.0
    - 连接两个字符串使用" .. ":
        - `print("a" .. 'b')`&rArr;ab
        - `print(123 .. 1243)`&rArr;1231243
    - 使用#来计算字符串的长度,放在字符串的前面
        - len = "helloworld"
        - print(#len)&rArr;10
- **table 表**
    - 创建一个空的表: local tbl1 = {}
    - 直接初始化: local tbl2 = {"apple", "grape"}
    - table不会固定长度大小,有新的数据时table长度会自动增长
- **function 函数**
    - 在Lua中,函数被看作是"第一类值",可以存在变量里
    ```lua
    function factorial1(n)
        if n == 0 then
            return 1
        else
            return n * factorial1(n - 1)
        end
    end
    print(factorial1(5))
    factorial2 = factorial1
    print(factorial2(5))
    
    -- ------------------------------
    -- 通过匿名函数的方式传参
    function testFun(tab,fun)
        for k, v in pairs(tab) do
            print(fun(k,v));
        end
    end
    
    tab={key1="var1",key2="val2"}
    testFun(tab,
        function(key,val)
            return key.."="..val;
        end
    );
    ```
- **thread 线程**
    - 在 Lua 里，最主要的线程是协同程序（coroutine）。它跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。
    - 线程跟协程的区别：线程可以同时多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起（suspend）时才会暂停。
- **userdata 自定义类型**
    - userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。
- **lua中的变量**
    - 共有三种变量: 全局变量,局部变量,表中的域
    - lua中所有的变量都是全局变量,哪怕是语句块或者函数里,除非用local显示的声明为局部变量
    - 局部变量的作用域为从声明位置开始到所在的语句块结束
    - 变量的默认值均为nil
- **赋值语句:**
    - lua可以对多个变量同时赋值,`a = "hello" .. "world"`,`a,b = 10,2*x`, `x,y = y,x`
    - 当变量的个数和值的个数不一致时, lua会以变量个数为基础采取以下策略
        1. 变量个数 > 值的个数 --> 按照变量个数补足nil
        2. 变量个数 < 值得个数 --> 多余得值会被忽略
- **索引**
    - 对table的索引使用[]
        1. t[i]
        2. t.i
- **循环**
    - 几种方式:
        - while 循环
        - for循环
        - repeat..until
        - 循环嵌套
    - 循环控制:
        - `break`
```lua
    while(true)
    do
        print("循环将永远执行下去")
    end
```
- **流程控制**
    - 控制结构的条件表达式结果可以是任何值, lua认为false和nil为假,true和非nil为真,所以lua中0也是真
    ```lua
    --[ 0 为 true ]
    if(0)
    then
        print("0为true")
    end
    ```
    - lua中的控制结构的语句
        - if语句
        - if..else 语句
        - if嵌套语句
- **函数定义**
    ```lua
        optional_function_scope function_name(argument1, argument2,argument3...,argumentn)
            function_body
            return result_params_comma_separated
        end
    ```
    - 解析
        - optional_function_scope: 该参数是可选的, 制定行函数是全局函数还是局部函数,未设置参数则默认是全局的,局部函数需要使用关键字local
        - function_name: 指定函数的名称
        - argument1,argument2,argument3...,函数的参数,会以逗号隔开
        - function_body: 函数体,函数中需要执行的代码语句块
        - result_params_comma_separated: 函数返回值,Lua语言函数可以返回多个值, 以逗号隔开
    - 实例
    ```lua
    --[[ 函数返回两个值的最大值 --]]
    function max(num1, num2)
        if(num1 > num2) then
            result = num1;
        else
            result = num2;
        end
        return result;
    end
    -- 调用函数
    print("两个值比较最大值为 ", max(10,4))
    print("两个值比较最大值为 ", max(5,6))
    ```
    - **lua中,将函数作为参数传递给函数**
    ```lua
    myprint = function(param)
        print("这是打印函数 - ##", param, "##")
    end
    
    function add(num1, num2, functionPrint)
        result = num1 + num2
        -- 调用传递的函数参数
        functionPrint(result)
    end
    myprint(10)
    -- myprint 函数作为参数传递
    add(2,5,myprint)
    ```
    - **多返回值**
        - lua函数可以返回多个结果值,比如string.find,其返回匹配串"开始和结束的下标"(如果不存在匹配串返回nil)
        ```
        > s,e = string.find("www.runoob.com","runoob")
        > print(s,e)
        5, 10
        ```
        - lua函数中, 在return后列出要返回的值的列表即刻返回多值
        ```lua
        function maximun(a)
            local mi = 1 -- 最大值索引
            local m = a[mi] -- 最大值
            for i, val in ipairs(a) do
                if val > m then
                    mi = i
                    m = val
                end
            end
            return m, mi
        end
        print(maximun({8,10,23,12,5}))
        ```
    - **可变参数**
        - lua函数可以接受可变数目的参数,和C类似,在函数参数列表中使用`...` 表示有可可变的参数
        ```lua
        function add(...)
        local s = 0
            for i,v in ipairs{...} do --> {...}表示一个由所有的变长的参数构成的数组
                s = s + v
            end
            return s
        end
        print(add(33,4,5,6,7)) ---> 25
        ```
    - 我们可以将可变参数赋值给一个变量
    ```lua
    function average(...)
        result = 0
        local arg={...} ---> arg为一个局部表,局部变量
        for i,v in ipairs(arg) do
            result = result + v
        end
        print("总共传入 ".. #arg .. "个数")
        return result/#arg
    end
    print("平均值为",average(10,5,3,4,5,6))
    
    // -------------
    执行结果为:
    总共传入 6 个数
    平均值为 5.5
    
    // ------------------------
    -- 有时候我们可能需要几个固定参数加上可变参数,固定参数必须放在变长参数之前:
    function fwrite(fmt, ...) -- -> 固定参数fmt
        return io.write(string.format(fmt, ...))
    end
    fwrite("runoob\n") -- -> fmt = "runoob", 没有变长参数
    fwrite("%d%d\n",1,2) -- > fmt = "%d%d", 变长参数为1和2
    ```
    - 通常在遍历变长参数的时候只需要使用{...}, 然而变长参数可能会包含一些nil, 那么就可以用select函数来访问变长参数了: select('#', ...) 或者 select(n, ...)
        - select('#', ...) 返回可变参数的长度
        - select(n, ...) 用于访问n到select('#', ...)的参数
    - 调用select时, 必须传入一个固定实参selector(选择开关)和一系列变长参数,如果selector为数字n,那么select返回它的第n个可变实参,否则只能为字符串"#",这样select会返回变长参数的总数
    ```lua
    do
        function foo(...)
            for i = 1, select('#', ...) do -- -> 获取参数总数
                local arg = select(i, ...); -- -> 读取参数
                print("arg", arg);
            end
        end
    
        foo(1,2,3,4);
    end
    ```
- **常用的字符串的操作**
    - `string.upper(argument)`: 字符串全部转为大写字母
    - `string.lower(argument)`: 字符串转换为小写字母
    - `string.gsub(mainString,findString,num)`: 在字符串中替换, mainString为要替换的字符串,findString为被替换的字符,replaceString要替换的字符, num替换次数(可以忽略,则全部忽略)<br/>`string.gsub("aaaa","a","z",3)`
    - `string.find(str,substr,[init,[end]])`: 在一个指定的目标字符串中搜索指定的内容(第三个参数为索引),返回其具体位置,不存在就返回nil
    - `string.reverse("Lua")`: 字符串的反转
    - `string.format(..)`: 返回一个类似printf的格式化字符串
    - `string.char(arg)`和`string.byte(arg[,int])`: char将整型数字转成支付并连接,byte转换字符为整数值(可以指定某个字符,默认第一个字符)
    - `string.len(arg)`: 计算字符串长度
    - `string.rep(string,n)`: 返回字符串string的n个拷贝
    - `string.gmatch(str,pattern)`: 回一个迭代器函数，每一次调用这个函数，返回一个在字符串 str 找到的下一个符合 pattern 描述的子串。如果参数 pattern 描述的字符串没有找到，迭代函数返回nil
    ```lua
    for word in string.gmatch("Hello Lua user","%a+") do print(word) end
    ```
    - `string.match(str, pattern, init)`: string.match()只寻找源字串str中的第一个配对. 参数init可选, 指定搜寻过程的起点, 默认为1. 在成功配对时, 函数将返回配对表达式中的所有捕获结果; 如果没有设置捕获标记, 则返回整个配对字符串. 当没有成功的配对时, 返回nil。
    ```lua
    string.match("I have 2 questions for you.", "%d+ %a+")
    ```
- **lua迭代器**
    - 泛型for迭代器,泛型 for 在自己内部保存迭代函数，实际上它保存三个值：迭代函数、状态常量、控制变量。泛型 for 迭代器提供了集合的 key/value 对
    ```lua
    -- 语法格式
    for k,v in pairs(t) do
        print(k,v)
    end
    
    -- k,v 为变量列表,pairs为表达式列表
    -- 例子
    array = {"Lua", "Tutorial"}
    for key,value in ipairs(array)
    do
        print(key,value)
    end
    ```
    - **泛型for的执行过程**
        1. 初始化，计算in后面表达式的值，表达式应该返回泛型 for 需要的三个值：迭代函数、状态常量、控制变量；与多值赋值一样，如果表达式返回的结果个数不足三个会自动用nil补足，多出部分会被忽略
        2. 将状态常量和控制变量作为参数调用迭代函数（注意：对于for结构来说，状态常量没有用处，仅仅在初始化时获取他的值并传递给迭代函数）
        3. 将迭代函数返回的值赋给变量列表
        4. 如果返回的第一个值为nil循环结束，否则执行循环体
        5. 回到第二步再次调用迭代函数
    - **无状态迭代器**
        > 无状态迭代器是指不保留任何状态的迭代器,因此在循环中我们可以利用无状态迭代器避免创建闭包花费额外的代价,每一次迭代,迭代函数都是用两个变量(状态常量和控制常量)的值作为参数被调用,一个无状态的迭代器只利用这两个值可以获取下一个元素
        
        ```lua
        -- 使用一个简单的函数来实现迭代器,实现数字n的平方:
        function square(iteratorMaxCount,currentNumber)
            if currentNumber < iteratorMaxCount
            then
                currentNumber = currentNumber + 1
            return currentNumebr, currentNumber*currentNumber
            end
        end
        
        for i,n in square,3,0
        do 
            print(i,n)
        end
        
        -- 迭代的状态包括被遍历的表(循环过程中不会改变的状态的常量)和当前的索引的下标(控制变量),ipairs和迭代函数
        function iter(a,i)
            i = i + 1
            local v = a[i]
            if v then
                return i,v
            end
        end
        
        function ipairs(a)
            return iter, a, 0
        end
        
        -- 当Lua调用ipairs(a)开始循环时，他获取三个值：迭代函数iter、状态常量a、控制变量初始值0；然后Lua调用iter(a,0)返回1,a[1]（除非a[1]=nil）；第二次迭代调用iter(a,1)返回2,a[2]……直到第一个nil元素
        ```
    - **多状态的迭代器**
        > 很多情况下,迭代器需要保存多个状态信息而不是简单的状态常量和控制变量,,还有一种方法是将所有的状态信息封装到table内,将table作为迭代器的状态常量,因为这种情况下可以将所有的信息存放在table内,所以迭代函数通常不需要第二个参数
        
        ```lua
        -- 创建自己的迭代器
        array = {"lua","tutorial"}
        function elementIterator(collection) 
            local index = 0
            local count = #collection
            -- 闭包函数
            return function()
                index = index + 1
                if 
                    index <= count
                then
                    -- 返回迭代器的当前的元素
                    return collection[index]
                end
            end
        end
        
        for element in elementIterator(array)
        do
            print(element)
        end
        ```

- **table表**
    > table 是lua的一种数据结构用来帮我们创建不同的数据类型,如:数组,字典等<br/>lua table 使用关联型数组,你可以用任意类型的值来做数组恩典索引,但是这个值不能是nil<br/>lua也是通过table来解决模块(module),包(package)和对象(Object)的
    
    ```lua
    -- 初始化表
    mytable = {}
    -- 指定值
    mytable[1] = "lua"
    -- 移除引用
    mytable = nil
    -- lua垃圾回收会释放内存
    
    --------------------------------------
    -- 我们为a设置元素,然后将a赋值给b,则a与b都指向同一个内存.如果a设置为nil,则b同样能访问table元素.如果没有指定的变量指向a,lua的垃圾回收机制会清理相对应的内存
    
    -- 简单的table
    mytable = {}
    print("mytable的类型是 ",type(mytable))
    mytable[1] = "lua"
    print("mytable 索引为1的元素是:",mytable[1])
    print("mytable 索引为wow的元素是",mytable["wow"])
    
    -- alternatetable和mytable是指向同一个table
    alternatetable = mytable
    
    print("alternatetable索引为1的元素是 ", alternatetable[1])
    print("mytable索引为wow的元素是 ",alternatetable["wow"])
    
    alternatetable["wow"] = "修改后"
    
    print("mytable索引为wow的元素是 ",mytable["wow"])
    
    -- 释放变量
    alternatetable = nil
    print("alternatetable是 ",alternatetable)
    
    -- mytable 仍然可以访问
    print("mytable索引为wow的元素是 ", mytable["wow"])
    
    mytbale = nil
    print("mytable是 ",mytable)
    ```
- **table 操作**
    - 常用的方法

<style>
table th:first-of-type {
    width: 100px;
}
</style>

序号 | 方法&用途
---|---
1 | table.concat(table[,sep[,start[,end]]]):<br>concat是concatenate(连锁, 连接)的缩写. table.concat()函数列出参数中指定table的数组部分从start位置到end位置的所有元素, 元素间以指定的分隔符(sep)隔开
2 | table.insert(table,[pos[,value]]):<br/>在table数组部分指定位置(pos)插入值为value的一个元素,pos参数可选,默认为数组部分末尾
3 | table.maxn(table):<br/>指定table中所有正数key值中最大的key值. 如果不存在key值为正数的元素, 则返回0。(Lua5.2之后该方法已经不存在了,本文使用了自定义函数实现)
4 | table.remove(table[,pos]):<br/>返回table数组部分位于pos位置的元素. 其后的元素会被前移. pos参数可选, 默认为table长度, 即从最后一个元素删起
5 | table.sort(table[,comp]):<br/>对给定的table进行升序排列

```lua
-- 方法实例:

-- table连接

-- 使用concat()方法来连接两个table
fruits = {"banana","orange","apple"}
-- 返回table连接后的字符串
print("连接后的字符串",table.concat(fruits))
-- 指定连接字符
print("连接后的字符串 ",table.concat(fruits,", "))
-- 指定索引来连接table
print("连接后的字符串 ",table.concat(fruits,", ", 2, 3))

-- 插入和移除

-- 演示table的插入和移除操作
-- 在末尾插入
table.insert(fruits,"mango")
print("索引为4的元素为 ",fruits[4])
-- 在索引为2的键处插入
table.insert(fruits,2,"grapes")
print("索引为2的元素为 ",fruits[2])
print("最后一个元素为 ",fruits[5])
table.remove(fruits)
print("移除后最后一个元素为 ",fruits[5])

-- table排序

fruits = {"banana","apple","grapes"}
print("排序前")
for k,v in ipairs(fruits) do
    print(k,v)
end

table.sort(fruits)
print("排序后")
for k,v in ipairs(fruits) do
    print(k,v)
end

-- table最大值

-- 自定义table_maxn来实现
-- 获取table中的最大值
function table_maxn(t)
    local mn=nil
    for k,v in pairs(t) do
        if(mn==nil) then
            mn=v
        end
        if mn < v then
            mn = v
        end
    end
    return mn
end
tb1 = {[1] = 2, [2] = 6, [3] = 34, [26] = 5}
print("tb1最大值: ",table_maxn(tb1))
print("tb1长度 ",#tb1)

-- 代码执行的结果是: 
-- tb1最大值: 34
-- tb1长度: 3

-- 当我们获取 table 的长度的时候无论是使用 # 还是 table.getn 其都会在索引中断的地方停止计数，而导致无法正确取得 table 的长度。
-- 使用以下方法替换:
function table_leng(t)
    local leng = 0
    for k,v in pairs(t) do
        leng=leng+1
    end
    return leng
end
```