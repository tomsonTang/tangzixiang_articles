# Lua 模块与包



## 摘要

1. 模块的概念
2. 如何实现一个模块
3. 如何引用一个模块
4. 模块加载路径 `package.path` 
5. 环境变量 `LUA_PATH` 的设置
6. 跨目录下的模块引用
7. 缓存机制
8. 执行环境
9. 参考



## Lua 中模块的概念

- 模块类似于一个封装库，从 Lua 5.1 开始，Lua 加入了标准的模块管理机制，可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。
- Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。



## 如何实现一个模块

```lua
-- 初始化一个对象
local Account = {balance = 0}

-- 对外开放 withdraw 函数
function Account.withDraw(v)
  Account.balance = Account.balance - v
end

-- 不对外开放 
function getBalance()
  return Account.balance
end

return Account
```

新建 `Account.lua` 文件，如上示例实现了一个名为 `Account` 的模块，通过 `return` 关键字实现内容的导出,其中外部可访问的内容为 `Account.blance` 字段以及 `Account.withDraw` 函数。



## 如何引用一个模块

在 lua 中通过全局函数 `require` 来实现对其他模块的引用。

使用方式:

- `require("<模块名>")`
- `require "<模块名>"`

示例:

```lua
#!/usr/local/bin/lua

-- 加载 Account.lua 文件
local Account = require("Account")

print(Account)

-- [[ 遍历内容 ]]
function printTable(_tab)
  for k, v in pairs(_tab) do
    print(k, v)
  end
end

printTable(Account)
```

执行结果如下:

```bash
tangzixiang$ ./Account_test.lua
table: 0x7ffc6b406a20
balance	0
withDraw	function: 0x7ffc6b407190
```

从执行结果可以看出导入的模块实际便是一个 table 的实现。导入的 `Account` 模块包含 `balance` 字段以及 `withDraw` 方法。

网上很多教程都只是讲到这里，实际上忽略了一个很重要的问题便是不同路径下的模块的引用。



## 模块加载路径 `package.path` 

在 Lua 中你无法像其他语言那样直接通过相对路径或绝对路径来引用模块，Lua 的模块引用与其加载机制有关，具体加载路径可以通过全局对象 `package ` 对象的`package.path` 字段获取默认的加载路径:

```lua
#!/usr/local/bin/lua

print(package.path)
```

执行结果:

```bash
tangzixiang$ ./package_path.lua
/usr/local/share/lua/5.3/?.lua;/usr/local/share/lua/5.3/?/init.lua;/usr/local/lib/lua/5.3/?.lua;/usr/local/lib/lua/5.3/?/init.lua;./?.lua;./?/init.lua
```

上述示例的 `?` 号即代表我们在 `require` 函数中的模块名，如前面的示例 `Account`。



示例引用一个不存在的模块:

```lua
#!/usr/local/bin/lua

require("XXX")
```

执行结果:

```bash
tangzixiang$ ./package_path.lua
/usr/local/bin/lua: ./package_path.lua:3: module 'XXX' not found:
	no field package.preload['XXX']
	no file '/usr/local/share/lua/5.3/XXX.lua'
	no file '/usr/local/share/lua/5.3/XXX/init.lua'
	no file '/usr/local/lib/lua/5.3/XXX.lua'
	no file '/usr/local/lib/lua/5.3/XXX/init.lua'
	no file './XXX.lua'
	no file './XXX/init.lua'
	no file '/usr/local/lib/lua/5.3/XXX.so'
	no file '/usr/local/lib/lua/5.3/loadall.so'
	no file './XXX.so'
stack traceback:
	[C]: in function 'require'
	./package_path.lua:5: in main chunk
	[C]: in ?
```

我们通过 `require` 的实际加载情况发现其对 `XXX` 的查找路径与前面输出的 `package.path` 一致。最后如果找不到则去找 so 文件( C 程序库)。



`require` 用于搜索 Lua 文件的路径是存放在全局变量 `package.path` 中，当 Lua 启动后，会以环境变量 `LUA_PATH` 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化，我们前面看到的便是默认路径。



到这里我们可以就知道了为什么前面我们可以成功通过 `require("Account")` 加载 `Account.lua` ,因为默认的 `package.path` 中包含了 `./?.lua;` ，表示会在当前同目录下寻找指定模块。



## 环境变量 `LUA_PATH` 的设置

如果没有 `LUA_PATH` 这个环境变量，也可以自定义设置。

假设我们现在有个项目库叫 `lua` ，放在了根目录下，为了平时可以更方便的引用，我们可以更新 `LUA_PATH` 让其包含该项目。 在当前用户根目录下打开 `.profile` 文件，并追加:

```bash
#LUA_PATH
export LUA_PATH="~/lua/?.lua;;"
```

文件路径以 ";" 号分隔，最后的 2 个 ";;" 表示新加的路径后面加上原来的默认路径。

接着，更新环境变量参数，使之立即生效。

```bash
source ~/.profile
```

我们这时再来看下 `package.path` 的输出:

```bash
tangzixiang$ ./package_path.lua
~/lua/?.lua;/usr/local/share/lua/5.3/?.lua;/usr/local/share/lua/5.3/?/init.lua;/usr/local/lib/lua/5.3/?.lua;/usr/local/lib/lua/5.3/?/init.lua;./?.lua;./?/init.lua;
```

可以看到此时的 `package.path` 已经包含了我们需要的库路径。

注：如果  `.profile`  不生效，则可以在尝试 `.bash_profile` 或则 `.bashrc` 文件重复上述操作。



## 跨目录下的模块引用

通常我们在实际开发是都会在工作目录下通过不同的目录来对功能模块进行划分，这时便需要动态的更改加载路径。

假设有如下路径:

```bash
tangzixiang$ tree
.
├── package_path.lua
├── model
│   ├── Account.lua
└── test
    └── Account_test.lua

2 directories, 3 files
```

我们需要在 test 目录下执行 `Account_test.lua` 文件，其中依赖于 model 目录的 `Account.lua` 文件

```lua
#!/usr/local/bin/lua

print("test/Account_test.lua\n")

-- 加载 Account.lua 文件
local Account = require "Account"

print(Account)

function printTable(_tab)
  for k, v in pairs(_tab) do
    print(k, v)
  end
end

printTable(Account)
```

执行结果:

```bash
tangzixiang$ ./Account_test.lua
test/Account_test.lua

/usr/local/bin/lua: ./Account_test.lua:7: module 'Account' not found:
	no field package.preload['Account']
	no file '~/lua/Account.lua'
	no file '/usr/local/share/lua/5.3/Account.lua'
	no file '/usr/local/share/lua/5.3/Account/init.lua'
	no file '/usr/local/lib/lua/5.3/Account.lua'
	no file '/usr/local/lib/lua/5.3/Account/init.lua'
	no file './Account.lua'
	no file './Account/init.lua'
	no file '/usr/local/lib/lua/5.3/Account.so'
	no file '/usr/local/lib/lua/5.3/loadall.so'
	no file './Account.so'
stack traceback:
	[C]: in function 'require'
	./Account_test.lua:7: in main chunk
	[C]: in ?
```

不出意外的发现异常了，因为我们引用的 `Account` 不在任何已有的加载路径下。如果想要能够正确的动态引用我们需要的模块，则需要在实际引用前动态的更新 `package.path` :

```lua
#!/usr/local/bin/lua

print("test/Account_test.lua\n")

-- 更新 package.path
package.path = ";../model/?.lua;" .. package.path

local Account = require "Account"

print(Account)

function printTable(_tab)
  for k, v in pairs(_tab) do
    print(k, v)
  end
end

printTable(Account)
```

执行结果：

```bash
tangzixiang$ ./Account_test.lua
test/Account_test.lua

table: 0x7f93f9d00830
balance	0
withDraw	function: 0x7f93f9d00150
```

效果完美。

注意：由于 `package.path` 是全局的故一旦更新则会在当前项目内生效。



## 缓存机制

Lua 也类似其他大部分语言的模块导入机制，存在缓存机制，模块一旦导入有且只在第一次导入时执行一次模块内容。

A.lua

```lua
local Account = require("Account")
print(Account)
```

B.lua:

```lua
#!/usr/local/bin/lua

package.path = ";./model/?.lua;" .. package.path

require("A")

local Account = require("Account")
print(Account)
```

执行结果:

```bash
tangzixiang$ ./B.lua
table: 0x7fabd8600880
table: 0x7fabd8600880
```

可以看出 `Account` 对象只实例化了一次。



## 执行环境

![](https://raw.githubusercontent.com/tangzixiang/tangzixiang_articles/master/img/QQ20180624-113702%402x.jpg)



```bash
tangzixiang:~ tangzixiang$ lua -v
Lua 5.3.4  Copyright (C) 1994-2017 Lua.org, PUC-Rio
```



## 参考

1. http://www.runoob.com/lua/lua-modules-packages.html 
2. 【Nginx Lua 开发实战】



![](https://github.com/tangzixiang/tangzixiang_articles/blob/master/img/wechat.png?raw=true)