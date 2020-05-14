---
layout: post
title:  "skynet 单步调试方案(zbstudio remote debug)"
date:   2018-01-05 20:26:25 +0800
categories: skynet
---

先上效果图：
![在单步调试snax服务的一个函数](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTgwMTA0MjE0NjQ5ODAx?x-oss-process=image/format,png)


# skynet 调试难题
skynet 的业务代码全是用 lua 写的，其实现在 lua 的调试工具已经很多了，也有很多好用的，但是由于 skynet 是多服务的，每个服务都是一个 lua 虚拟机，成百上千的 lua vm 导致现有的 lua 调试工具都没法直接使用。

虽然云风提供了一个简单的调试控制台，但实在谈不上好用，基于命令行的模式对于用惯各种现代 IDE 调试的我们来说实在无法接受。

这篇文章是云风对 skynet debug console 的介绍：
[在线调试 Lua 代码](https://blog.codingnow.com/2015/02/skynet_debugger.html)
里面也有云风对于调试的态度：
> 单步跟踪调试单个 lua coroutine 的能力。这对许多新手来说是个学走路的拐杖，虽然有人一辈子都扔不掉。

对于新生代的程序员来说，他们从入行开始，使用的就是 VS+VA/Eclipse 等等现代 IDE，也习惯了这种方便的调试方式，没理由开历史的倒车，去使用命令行调试吧？

当然不是说命令行调试不好，这也是一个合格的服务端程序必备的技能（查线上问题），但是调试效率就不那么高了。

工具就只是工具，既然有更高效的工具，能提高生产效率，那没理由不用是吧，程序员的时间可都是钱啊！


# 使用 zbstudio 远程调试单个服务
这里给出一个使用 zbstudio 调试的方案，该方案可以单步调试 skynet 的 lua 代码，但是只能调试单个 lua vm，无法跨服务调试，遇到 `skynet.call` 可以正确返回结果。

但该方案比较麻烦的是，每次调试之前，都要先用 zbstudio 开启调试服务器，然后在你要调试的服务开始处增加一段代码，用来连接调试服务器，整个过程比较繁琐。

如果调试一些简短的逻辑，建议还是 print 吧，对于非常大块的逻辑，比如上千行的大功能，用工具单步调试的效率更高。

# 具体步骤
### 1. 下载 zbstudio
下载链接：https://studio.zerobrane.com/download?not-this-time)

 * 如果是在 Mac 平台下开发，只下载个 Mac 版就好。
 * 如果是 Windows 开发，Linux 虚拟机跑，要分别下载 Windows 版和 Linux 版。

为啥要下两个呢？

由于 zbstudio 在调试时是一个服户端的角色，所以本身没有平台限制，用哪个都一样，但是一般我们的开发平台都是 Windows 的，所以当然也是用 Windows 了。

另一个 skynet 运行平台的 zbstudio, 比如你的 skynet 跑在 Linux 下，那就下载一个 Linux 版的 zbstudio, 这么做的原因是为了提取其中的 `*.so` 库。

前面不是说过了在要调试的服务开始处要加一些代码，用来连接调试服务器么，这些代码就要用到这些 `*.so` 库。


### 2. 安装 zbstudio
* Windows 下比较简单了，绿色版的，解压就可以用，安装版的也很省事。
* Linux 版的下载下来是个 `*.sh` 文件，直接 bash 文件名执行就安装好了，安装完进入虚拟机（注意这里不要在 ssh 里操作了）桌面，打开终端，输入 `zbstudio` 即可启动。
* Mac 版下载下来是个 `*.dmg` 文件，安装也很简单。

### 3. 提取 `*.so` 和 `*.lua` 文件

我们用来打开调试服务器的代码长下面这样：
```lua
local mobdebug = require "mobdebug")
local ip = "192.168.0.100"
local port = 8172 -- 可以不填，默认端口就是8172
mobdebug.start(ip, port)
```

可见我们就用到这一个 `mobdebug.lua` 文件，只要把这个文件依赖的所有的 `*.so` 和 `*.lua` 文件全部提取出来就可以了。

经过实际测试，只要以下 3 个文件就可以了：
* `/opt/zbstudio/bin/linux/x64/clibs53/socket/core.so`
* `/opt/zbstudio/lualibs/mobdebug/mobdebug.lua`
* `/opt/zbstudio/lualibs/socket.lua`

我们把这三个文件放到工程目录下，并在 luapath 和 luacpath 里加上它们所在目录。

### 4. 修改代码：在 skynet 服务初始化时连接调试服务器
为了使用的方便，我们把这些代码封装到一个全局函数内，放在 preload 内预先加载到全局空间中。
```lua
-- zbstudio remote debug
-- 开启远程调试
function startRemoteDebug()
    require("mobdebug").start(dbconf.remotedebug.ip, dbconf.remotedebug.port)
    Log.d(">>Start zbstudio's remote debug.")
end

-- 关闭远程调试
function stopRemoteDebug()
    require("mobdebug").done()
    Log.d("<<Stop zbstudio's remote debug.")
end
```
注：由于同一时间只能有一个 mobdebug 连接调试服务器，所以在 `startRemoteDebug` 函数内，可以加个全局标记，防止多次调用，我这里偷个懒，先不加。

不加的原因还有一个，就是我们自己调试时，肯定都是只调一次这个函数，调试完也会习惯性的把这段代码注释掉。

***
好了，现在封装了开始调试的函数了，让我们实际用一下吧。

这里我们用一个 snax 服务举例：
```lua
function init()
	 startRemoteDebug()
end

function exit()
	-- 这里也可以不加，因为我们调试完一般都会 ctrl+c 强退进程
	-- 反正我是从来不加这句的的，这里加上只是为了演示下还有 stop 这种“优雅”的用法，其实对调试来说没什么必要
	stopRemoteDebug()
end

-- 在调试完后，不要忘记把这两个函数注释掉。
```

### 5. 打开 zbstuido 并定位到要调试的 skynet 工程
打开后默认是 zbstudio 自己的目录，要改成自己工程的目录。

说来惭愧，当初改这个目录都改了好久，我觉得不是我搓，而是这个工具太难用：）

方法：
鼠标点中左侧工程根目录处，按F2，填入自己工程的路径，回车。

### 6. 开启 zbstudio 调试服务器
在 zbstudio 主界面中，点击菜单中的 `Project --> Start Debugger Server`。

### 7. 下断点
`Ctrl + P` 打开要下断点的文件。

在自己要调试的逻辑处按 `Ctrl+F9` 下个断点，仅限你要调试的那个服务，其他的下了断点也没用。

### 8. 开启你的 skynet 进程
这时如果正常，你应该能在 zbstudio 的 Output 栏看到：
`Debugging session started in 'xxxxx'`

这就说明已经连上调试服务器了，之后 zbstudio 会断在调用 startRemoteDebug() 那，这时按F5，就会到你下断点的地方了。


# 跨文件 Step into 的方法
在按 F10 Step into 时，如果某个函数在另一个 lua 文件内，这时要先打开那个文件（只要开着就行，在标签上能看到），再按 F10，才能过去，否则不会过去，这个比较坑，我也是用了一段时间后发现的，之前一直以为跨文件无法 Step into，其实是可以的！

有了这个技巧后，这个工具才算是真的好用了起来。

# 结束语
好了，本教程到此结束，剩下的关于 zbstudio 的功能，自己可以慢慢摸索，它的目录里面也有一些不错的 lua 库，可以拿来用用。

其实这个 IDE 功能并不多，你看它十几 MB 的身板，功能也多不了，有个比较好玩的功能是 lua 注释里支持 markdown, 简直要上天=。=

另外，本调试方案最初是 [lf723](https://github.com/lf723) 同学摸索出来的，非常感谢！

**END**