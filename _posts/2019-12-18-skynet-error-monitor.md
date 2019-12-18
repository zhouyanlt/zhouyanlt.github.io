---
layout: post
title:  "30行代码在skynet中实现预警机器人"
date:   2019-12-18 20:20:25 +0800
categories: skynet
---

# 1. 预警机器人的定义
预警机器人就是当线上有任何错误发生时，它会把错误信息以某种形式通知到某处。

# 2. 思路
## 2.1 报错拦截
在 skynet 构建的系统中，报错一般是 lua 引起的，比如 attemp index a nil value, 这些报错的位置虽然分散在成百上千个文件里，但入口其实非常有限，我们只要在入口处拦截掉这些报错，就能把错误信息发送到出去。

以下列出不同业务类型其报错的入口点：

函数类型 | 入口点 | 底层代码位置
---|---|---
skynet.dispatch | dispatch 函数自身 | 无
skynet.fork | assert | 
skynet.timeout | error | 
snax 服务 | assert | snaxd.lua Line 46

可以看到，除了 dispatch 外，其他几类都在 assert/error 里，所以我们只需要改写 assert/error 函数，并在 dispatch 里使用 pcall 把真正的业务函数包起来，再把结果送到 assert 或 error 里，整个错误拦截就完成了。

之所以这么简单，还是得益于云风良好的编码风格，几乎底层在执行上层业务函数时，都是 pcall 包起来，再由 assert/error 抛出这么个模式。

## 2.2 服务设计
我们需要创建一个专门发错误信息的服务，在改写 assert/error 时，把错误信息统一转发到这个服务，再由这个服务处理。

我们给这个服务起名叫 error_monitor 吧，这里面只做一件事，接收其他服务发来的报错信息，整理一下（带上集群名和节点ID即可），发到目的地即可。

## 2.3 错误信息发送目的地
拦截到错误信息后，想发到哪都可以，我们不弄太复杂，直接用钉钉机器人。不熟悉的看[这里](https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq)。

发送的方式也不打算用 libcurl, 而是直接用终端里的 curl, os.execute("curl xxxx"), 这里只是为了简单，因为假设我们的报错信息不是很多，用这种方式也完全够用了。如果你觉得你们的报错很多，可以改用 libcurl, 当然这样30行代码搞不定了。

# 3. 实现
## 3.1 实现 error_monitor 服务
```lua
local skynet = require "skynet"
skynet.start(function()
    local worldName = skynet.getenv("world_name") or "Unknown(Please add world_name to config)" -- worldName 也就是集群名，比如是内服还是外服
    local url = "https://oapi.dingtalk.com/robot/send?access_token=xxxxx" -- 后面的xxx改成自己机器人的token
    local selfnodeid = 1 -- 这里想办法取到自己的节点id
    skynet.dispatch("lua", function(_,_,...)
        os.execute(string.format([[curl '%s' -H 'Content-Type: application/json' -d '{"msgtype":"markdown","markdown":{"title":"ERROR","text":"* World: %s\n* Node: %d\n* Traceback:\n```%s```"}}']],
            url, worldName, selfnodeid, select("#",...)>1 and table.concat({...}," ") or tostring(...)))
    end)
end)
```
## 3.2 改写 assert/error
```lua
function util.registerErrorMonitor()
    local addr = skynet.uniqueservice("error_monitor")
    sutil.registerServerLaunchCallback(function()
        local _error = error
        function error(...)
            skynet.send(addr, "lua", ...)
            _error(...)
        end
        local _assert = assert
        function assert(...)
            if not ... then
                skynet.send(addr, "lua", ...)
            end
            return _assert(...)
        end
    end)
end
```
改写完后，在需要监控报错的服务的初始化那里，调一下这个函数即可。

这里不推荐把改写函数放到 preload 里去，因为并不是所有服务都需要改写的，一般我们只改几个大的业务服务即可，更不用说还不能在 preload 里调 uniqueservice，当然这个有办法克服，但没有必要。

最后不要忘记 skynet.dispatch 改成 pcall+assert 的模式，这样才可以把报交出去，如果不想要重复写(DRY原则)，可以简单封装一下。（我们业务层基本都是 snax 服务，所以没这个烦恼：）
