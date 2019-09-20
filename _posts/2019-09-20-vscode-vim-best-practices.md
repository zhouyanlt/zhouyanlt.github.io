---
layout: post
title:  "VSCodeVim 最佳实践"
date:   2019-09-20 10:36:25 +0800
categories: vim
---

# 1. Preface
初学者在学了一些 Vim 入门教程，掌握了一些基本操作后，往往不知道从何下手，日常工作中不太可能在终端下直接写代码，何况直接终端用 Vim 写代码，还需要大量插件的支持。

其实我认识的很多 Vim 用户，现在已经不用原生 Vim 了，都是先选一个自己喜欢的编辑器/IDE, 再装 Vim 插件，基本上没有哪个现代编辑器/IDE没有 Vim 插件的了。

我比较喜欢 VSCode，日常工作也是用 VSCode 写代码，所以今天就来介绍下 VSCode 里 Vim 插件的一些设置和应用技巧，至于是不是最佳实践不重要，起这样的标题完全是不知道用什么好，权当做一回标题党吧。

# 2. 编辑器选择
可能有些人并不喜欢用 VSCode，或者更习惯 Sublime 等待，这里介绍几个编辑器的 Vim 插件现状
* VSCode, 目前我最喜欢的，Vim 插件支持非常成熟，其他编辑器的 Vim 插件一般只是提供了 Vim 的原生功能，而 VSCode 的 Vim 插件居然提供了很多原生 Vim 里的插件功能，也就是说，它是一个 Vim 插件+Vim 插件的插件的集合体，功能非常强大，像是最常用的 easymotion 就提供了支持。原生 Vim 其实并不好用，想要高效还是要配合插件一起用，而 VSCode 的 Vim 插件做到了，我认为现在没有什么编辑器的 Vim 插件能够做到。
* Sublime Text, 编辑器内建了对 Vim 的支持，所以不装插件只要改下配置就能用，但是功能比较原始，实在谈不上好用，只能说能用。也许有很多第三方插件提供了类似 easymotion 这样的功能吧，我没研究过，想用的可以看看有没有，如果有还是可以用的。
* VisualStudio, 由于 VS 内建了海量的快捷键，且用熟了之后也很方便，所以对于 VS 资深用户，装个 Vim 插件反而不好用，这上面的 Vim 插件我也用过，体验非常糟糕，不推荐。
* MacVim, 配合 [spf13-vim](https://zhouyanlt.github.io/vim/2019/09/18/spf13-vim-frequently-used-hotkeys.html) 还是可以用的，我个人在对 VSCode 审美疲劳时也会用一用，轻量、好看、好用。
* IntelliJ IDEA, 这是个 IDE 了，非常重，但功能还是十分强大的，Vim 插件也基本够用，但没有 easymotion，加上太重，开起来电脑很卡，所以也不推荐用，但是喜欢的人特别喜欢。

# 3. VSCode Vim 插件的安装
上面说了那么多，对于初学者来说，建议直接用 VSCode 吧，不折腾。

安装非常方便，打开插件界面，输入 Vim，第一个就是，点安装即可。

800多万的安装量，感受下 Vim 用户的庞大，是不是感觉不用 Vim 和时代脱离了：）

# 4. 最佳实践
重头戏来了，下面装会介绍一些干货。

## 4.1 settings.json 配置
```json
{
    "vim.easymotion": true,
    "vim.leader": ",",
    "vim.searchHighlightColor": "#5f00af",
    "vim.hlsearch": true,
    "vim.normalModeKeyBindingsNonRecursive": [
        { "before": ["H"], "after": ["g", "T"], },
        { "before": ["L"], "after": ["g", "t"] },
    ],

    "editor.lineNumbers": "relative",
    "files.autoSave": "onFocusChange",
    "workbench.editor.enablePreviewFromQuickOpen": false,
    "editor.renderWhitespace": "boundary",
    "editor.detectIndentation": false,
    "showMusicMetrics": false,
    "showGitMetrics": false,
    "showWeeklyRanking": false,
    "editor.minimap.enabled": false,
    "git.autofetch": true,
}
```
首先把上面的配置放到自己的配置文件里，Mac 平台点左上角 Code->Preferences->Settings-> 在右边往下拉，找到 Edit in settings.json, 把上面内容放进去，大括号自己去掉。

解释几个：
### Vim 相关的
* "vim.easymotion": true, 打开 easymotion, 这个是文件内快速跳转的插件(Jump everywhere)
* "vim.leader": ",", 设置 leader 键为逗号，leader 键是 Vim 里某些指令的前置按键，默认是反斜线，比较难按。这个按键非常常用，必须改成逗号
* "vim.searchHighlightColor": "#5f00af", 设置搜索高亮，不设的话没有高亮，很蠢
* "vim.hlsearch": true, 高亮显示最近的搜索结果，就是你按一下 *, 把结果都高亮出来，这个很重要
* "vim.normalModeKeyBindingsNonRecursive", 这个是改按键映射关系的，改完可以使用 H/L 切换标签
### Vim 无关的
* "editor.lineNumbers": "relative", 设置相对行号，这个对于多行删除，多行注释非常重要，有了这个才算是可以用，否则自己数行数那基本上没法用
* "files.autoSave": "onFocusChange", 设置文件自动保存，onFocusChange 指的是焦点离开后就保存，这个跟 Vim 无关，但非常好用，不用一直按 Cmd+s 了
* "workbench.editor.enablePreviewFromQuickOpen": false, 用 ctrl-p 打开的文件，默认是处理预览状态的，重新预览其他文件时就没了，要双击标签才会固定住，这个设置就是解决这个问题的，设置完后打开就是固定住的，读代码很方便
* "editor.renderWhitespace": "boundary", 设置显示行首的空白字符（空格和制表符），这个也很不错，直观的看到自己输出的空白，有些开源项目要求必须全用空格。
* "editor.minimap.enabled": false, 关掉右边的小地图，就是宽宽的那个，这个看个人喜好了，我不太喜欢，开分栏时会减少显示面积

## 4.2 常用技巧
### 4.2.1 行级操作（删除、注释、左移右移）
上面设置了相对行号后，才能方便的做多行操作，在做具体操作前，先说单行操作分别是什么：
* `dd` 删除一行, `d` 是 `delete`, 两个 d 表示对一行操作，Vim 里的约定吧
* `gcc` 注释一行, `gc` 是 `go comment`, 两个 c 表示注释一行
* `<<` 左移一行
* `>>` 右移一行

上面的操作，在 v 模式下，都可以生效，可以试一下按 `Shift+v` 选中一行，然后分别按 `d`, `gc`,`<`, `>`,  看看是不是删除、注释、左移、右移

好了，有了上面的这些知识，我们可以开始多行操作了，具体操作方法如下：
* 光标移到想要删除或注释的第一行
* 通过左边的相对行号，看一下想操作的最后一行的相对行号n
* 得出要操作的行数 m = n+1 (因为光标所在那行相当于第0行，所以要+1)
* 这时就可以用 Vim 的多行操作了，删除一行是 dd, 删除多行是 mdd; 注释一行是 gcc, 注释多行是 mgcc, 其他的类推

### 4.2.2 切换标签
* `H` 大写的 H 移动到左边那个标签
* `L` 大写的 L 移动到右边那个标签
* `ngt` 移动到第 n 个标签（从左往右从1开始），当标签开的比较多时，由于标签上没显示数字，要自己数是第几个，所以基本上没法用。

对于标签开的比较多时怎么操作，我的习惯是这样的：
* 如果标签离的近，就是 H/L 移动过去
* 如果离的不是特别远，按5下 H/L 还是可以接受的
* 如果特别远，对于要长时间写代码的几个标签，用鼠标把几个常用标签拖到一起；如果是一次性的，就用鼠标直接点了；也会用 ctrl-p 直接搜文件重新打开。

### 4.2.3 调整光标所在行的位置(Vim基本功能)
* `zz` 调整光标所在行到屏幕中间 (z 什么意思我也不知道，就硬记吧)
* `zt` 调整光标所在行到屏幕最上方 (t == top)
* `zb` 调整光标所在行到屏幕最下方 (b == bottom)

这三个用的非常多，尤其是 `zz`, 一定要熟练使用

### 4.2.4 文件内快速搜索某个函数、变量(Vim基本功能)
把光标移到想搜索的那个函数名或变量名上，按`*`就可以搜索了，按`n`下一个，按`N`上一个。

比如看到类里有个成员变量，想去回到它的定义处看有没有注释说它是干嘛的，一般定义的地方肯定是文件内第一次出现的地方，我们可以这么操作：
* 先在这个变量上按下`*`锁定它
* 按 `gg` 回到行首
* 按 `n` 到下一个它出现的地方
* 按 `zz` 调整到屏幕中间，方便阅读

这4步熟悉后基本一气呵成

### 4.2.3  对一个单词进行操作
这里先说两个概念，操作符(operator)和动作命令(motion), 操作符就是 `d(delete)`, `c(change)`, `y(yank)` 等等，动作命令是`iw(in word)`, `aw(a word)`这些，可以通过 `操作符 + 动作命令`的方式，组合出千变万化的操作出来，帮助我们快速移动、修改等等。

完整列表可以在 Vim 里输入 :h motion.txt 查看，这里只列出一些常用的。

#### 4.2.3.1 操作符(Operators)

操作符 | 功能
---|---
c | change (先删除再进入插入模式)
d | delete
y | yank into register (does not change the text)
~ | swap case (转换大小写)
gu | make lowercase (转为小写)
gU | make uppercase (转为大写)
\> | shift right (右移)
< | shift left (左移)

#### 4.2.3.2 动作命令(Motions)
动作命令 | 功能
---|---
f{char} | find, 行内搜索一个字符
t{char} | till, 功能类型 f{chat}, 但是是在这个搜索到的字符前停下，意思是到这个字符之前，很常用的motion
gg | 跳到行首
G | 跳到行尾
w | 移到下个单词的第一个字符
b | 移到上个单词的第一个字符
e | 移到单词最后一个字符
ge | 上个单词的最后一个字符
aw | "a word", 选择一整个单词，包括它边上的空格
iw | in word, 选择一整个单词，不包括空格

#### 4.2.3.3 举例：
* `diw`, delete in word, 删除一个单词，只删除单词本身，不会旁边的空格
* `daw`, delete a word, 删除一个单词，并删除旁边的一个空格（至于是左还是右边空格，Vim 会根据上下文自己判断，非常智能）
* `d` 是删除， `diw` 就是删除一个单词
* `v` 是进入 visual 模式的，如果用 `viw`, 就表示选中一个单词
* `y` 是复制(yank), `yiw` 表示复制一个单词
* `gu` 是变成小写，`guiw` 就是让一个单词变成小写
* `gU` 是变成大写, `gUiw` 就是让一个单词变成大写
*

还有很多，这样只要记住一个，就可以举一反三，所以 Vim 是不需要死记硬背的，理解着记会更快。（当然不排除一些要死记硬背，但比较少）

### 4.2.4 把一个单词改成另一个（通过复制）
比如有下面一段代码：
```lua
local _hello = true
function test()
    print(_world)
end
-- 用复制的方式把 _world 改成 _hello
```
这个问题看起来很简单，但实际上用起来后就会发现并不简单，我们用鼠标时的操作逻辑是下面这样的：
* 双击 _hello，Ctrl+C 复制
* 双击 _world，Ctrl+V 粘贴

想象中的 Vim 操作逻辑是下面这样的：
* 光标移到 `_hello`, `yiw` 复制
* 光标移到 `_world`, `diw` 删除，再 `p` 粘贴
这时你会发现，咦，贴出来的还是 `_world`, 而不是期望中的 `_hello`, 这是因为 Vim 里的删除，实际上是“剪切”，会把删除的东西放到“剪切板”里，所以我们不能这样操作，而应像下面这样操作：

* 光标移到 `_hello`, `yiw` 复制
* 光标移到 `_world`, `viw` 选中，再 `p` 粘贴

这个操作是目前的最优解了，《Vim 实用技巧》这本书里给的也是这个方案（我是先自己发明这个方案才看的书：），这个方案虽然也很别扭，但是够用了，习惯了也不是不能接受。

# 5. Postface
暂时就想到这些，还有很多比较常用的，想到了再补充。以上这些如果能掌握，基本上在 VSCode-Vim 环境下写代码足够了，等真正用进去了，才会发现有很多不会的或者不方便的点，这时要学会去 Google 或请教别人，只有这样才能不断强化，最终让 Vim 操作变成肌肉记忆。