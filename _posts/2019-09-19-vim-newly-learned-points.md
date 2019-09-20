---
layout: post
title:  "新学到的一些 Vim 知识点"
date:   2019-09-19 05:00:25 +0800
categories: Vim
---

最近发起了一个[公司内部的学习班计划](https://zhouyanlt.github.io/education/2019/09/19/online-joint-learning-plan.html), 第一期的 Vim 入门，用的[实验楼的课程](https://www.shiyanlou.com/courses/2)，看完后还是有很多收获的，我记了下来，免得以后忘掉，大家也可以看看，有些非常有用，比如 `g*`。

命令 | 说明
---|---
15G                    |  跳到15行，我之前都是 :15，感觉后面这个方便点，主要是习惯了吧
:ce                    |  center, 使一行居中
:le                    |  left, 左对齐
:ri                    |  right, 右对齐，这三个好像没什么用，文字编辑工作者用的，平时格式化代码用的比较多的还是 n<< 或 n>>, 当然这个有时候可能会比较高效吧，记住也无妨
?                      |  Like /, but ? is back search
\#                     |  同\* , 只不过是反向搜索
g*                     |  跟*一样是查找当前单词，但只要部分匹配，这个好用！！一直以为没这个功能
g#                     |  同上，反向搜索
:e#                    |  回到前一个打开个文件，猜测#是个自动变量，保存上个文件的名字，e%是重新打开当前文件，因为%是记录当前文件名的自动变量, 通过 !echo $,# 我的猜测得到了证实
:f                     |  Show current editing file's name
:f newname             |  Change current file's name to newname
:n                     |  Open the next file in the buffer
:N                     |  Open the previous file in the buffer
vim -x fileName        |  创建加密文件，这个还挺有意思的，可以加密一些私密文件
:set                   |  or :se, show all changed options
:set all               |  Show all options
:set option?           |  Show option's value
:set nooption          |  Cancel option's value
:set option=xxx        |  设置某个选项的值，不同选项的可选值需要查文档
:set autoindent(ai)    |  auto indent 自动缩进
:set autowrite(aw)     |  设置自动存档，默认未打开
:set backup(bk)        |  设置自动备份，默认未打开
:set cindent(cin)      |  设置 C 语言风格缩进，不知道干嘛的
:set shiftwidth=4      |  设置每次缩进时的空格数

最后吐槽下实验楼的这个课程，里面很多写错的，课程安排也不太好，很多没用的，这也就是给我这样的老手看还能吸收这么多有用知识，新人早劝退了。

新人入门还是推荐 CoolShell 的文章：
[简明 VIM 练级攻略](https://coolshell.cn/articles/5426.html)




