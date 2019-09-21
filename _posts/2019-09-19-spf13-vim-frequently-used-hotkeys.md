---
layout: post
title:  "spf13-vim 介绍及常用快捷键"
date:   2019-09-19 05:00:25 +0800
categories: vim
---

# 一、什么是 [spf13-vim](https://github.com/spf13/spf13-vim)
## 官方简介：
spf13-vim is a distribution of vim plugins and resources for Vim, Gvim and MacVim.  
spf13-vim 是 Vim，Gvim 和 MacVim 的 vim 插件和资源的整合包。

It is a good starting point for anyone intending to use VIM for development running equally well on Windows, Linux, *nix and Mac.  
对于任何打算使用VIM进行开发的人来说，这是一个很好的起点，可以在Windows，Linux，\*nix和Mac上同样运行。

## 个人使用感受：
我觉得 spf13 算是整合的比较好的一个插件集了，有很多好用的插件，也有很多好用的快捷键，Vim 不是说装好插件就可以用的很舒服了，还要自定义很多快捷键，这些都是在 .vimrc 里配置的，想要用的舒服、顺手，这个配置要改很多，这是一个长期且艰辛的工作，而 spf13 都帮我们做好了，省了很多事。

spf13-vim 是一个开箱即用的 vim 整合包，就像大脚/魔盒之于WOW，可以说有了 spf13, Vim 才成为一个强力的终端 IDE。

## 关于 spf13 停止维护的问题：
spf13 的作者确实已经停止维护了， github 上最后一次提交是在2016年，但这不影响使用，它仍然是我用过最好用的 vim 整合包。

后续我希望我有精力把它维护起来，大家有余力也可以做这件事。

# 二、安装
```
curl https://j.mp/spf13-vim3 -L > spf13-vim.sh && sh spf13-vim.sh
```
等待安装完成即可，中间会有几个插件需要输入 github 的帐号和密码的，这几个插件用不了，直接按回车跳过即可。

装好后，不论是在终端下使用 Vim，还是 MacVim, 都已经有了 spf13 的环境。

# 三、常用快捷键
## 3.1 [NERDTree](http://github.com/scrooloose/nerdtree) 文件浏览器
* `<Ctrl-e>` 按一下打开文件浏览器，再按一下关闭
* `<leader>e` 打开文件浏览器，并且文件定位到当前打开的文件上

> `<Ctrl-e>` 表示 Ctrl + e, `<leader>e` leader 是 Vim 中的一个概念，表示“前置按键”，有很多操作要先按下 leader 键再接一个按键，leader 可以在 vimrc 里配置，默认是`\`，在 spf13 里是逗号(`,`)

打开文件浏览器后，可以使用 hjkl 上下移动，按回车就会打开当前光标所在的文件，按?可以打开帮助界面，里面有详细的按键介绍，有兴趣可以看看，常用的就是移动+回车。

还有两个比较常用，按 s 在一个拆分窗口里打开文件，按 t 在新的 tab 页打开。

## 3.2 [ctrlp](http://kien.github.io/ctrlp.vim/) 全局文件搜索
* `<Ctrl-p>` 打开文件查找界面，这时可以输入文件名字，结果是秒出的，跟使用 vscode 和 sublime 一样，不一样的是不支持模糊查找，所以没 vscode/sublime 好用，但基本够用了，凑合用用吧。

## 3.3 [EasyMotion](https://github.com/Lokaltog/vim-easymotion) 任意位置跳转
* `<leader><leader>j` 按下后，会在每一行的行首高亮且有一个字母标在高亮处，这时输入指定字母就可以跳过去了，这个可以说是起飞的关键之一，必须要熟练掌握
* `<leader><leader>k` 上面是向下搜索，这个是向上，其他都一样
* `<leader><leader>w` 同`,,j`，只不过是单词级的向后搜索
* `<leader><leader>b` 同`,,k`, 只不过是单词级的向前搜索（`,,w` `,,b` 不如行级好用，因为太花了，我们可以用行级 Jump 到指定行，再用 f 过去，或者按几个 w 也可以，如果靠近行尾，可以先按$到行尾再按几个 b，方法很多。

## 3.4 多标签
* 在用 ctrl-p 搜到一个文件后，可以再按 ctrl-t 在新标签里打开
* 在文件浏览器(NerdTree)里，定位到某个文件后，按 t 在新标签里打开
* `:tabe 指定文件` 这样也可以在新标签中打开一个文件
* `:tabo` 这样就可以关闭所有其他标签，只保留当前打开的，单词的意思应该是 tab close others
* `L` 打开下一个标签(默认按键是 `gt`)
* `H` 打开前一个标签(默认按键是 `gT`)
* `ngt` n是一个数字，输入几就直接打开第几个标签，这个只在Vim 或 MacVim/GVim 下生效，VSCode 插件不生效，Sublime 插件估计也不行。

我是用 VSCode 的，切标签一般我的习惯是，当打开的标签不多时，就多按几个 H/L，如果很多，就放弃移动了，直接 Ctrl-p 重新打开吧，这样还快点，当然偶尔也会用鼠标直接点，怎么方便怎么来。

## 3.5 代码注释
* `gc` 选中若干行后, 输入`gc`注释它们。(go comment)
* `gcc` 不需要选中，直接用，可以注释一行。Vim 里一般重复字符就是对一行进行操作，比如 dd 删除一行，这里 gcc 也是类似的逻辑
* `ngcc` n 是数字，注释 n 行，这个非常好用，也是起飞的关键指令之一。

## 3.6 [Surround](https://github.com/tpope/vim-surround) Vim Action 增加
这个并不常用，但是很好用，偶尔有奇效，可以选择性的掌握。

具体用法我也不想写了，直接看作者写的英语吧。

This plugin is a tool for dealing with pairs of "surroundings." Examples of surroundings include parentheses, quotes, and HTML tags. They are closely related to what Vim refers to as text-objects. Provided are mappings to allow for removing, changing, and adding surroundings.

Details follow on the exact semantics, but first, consider the following examples. An asterisk (*) is used to denote the cursor position.

Old text | Command | New text
--- | --- | ---
"Hello *world!"          | ds"        | Hello world!
[123+4*56]/2             | cs])       | (123+456)/2
"Look ma, I'm *HTML!"    | cs"\<q\>     | \<q\>Look ma, I'm HTML!\</q\>
if *x>3 {                | ysW(       | if ( x>3 ) {
my $str = *whee!;        | vllllS'    | my $str = 'whee!';

For instance, if the cursor was inside "foo bar", you could type cs"' to convert the text to 'foo bar'.

There's a lot more, check it out at :help surround

# 四、一些自宝义设置
```vim
nnoremap <f5> :!ctags -R<CR>

set nospell
set relativenumber
set nopaste

let g:Tlist_Ctags_Cmd='/usr/local/Cellar/ctags/5.8_1/bin/ctags'
let g:ctrlp_cmd = 'CtrlPMixed'

let g:syntastic_check_on_open = 1
let g:syntastic_lua_checkers = ["luac", "luacheck"]
let g:syntastic_lua_luacheck_args = "--no-unused-args"

" To open each buffer in its own tabpage
" au BufAdd,BufNewFile * nested tab sball

set iskeyword-=.                    " '.' is an end of word designator
set iskeyword-=-                    " '.' is an end of word designator

" path指定查找的路径，详情help path
" includeexpr是尝试替换路径名中的.为/，详情help includeexpr
" suffixesadd为尝试路径后缀，详情help suffixesadd
set includeexpr=substitute(v:fname,'\\.','/','g')
set suffixesadd=.lua
```

把上面这些内容放到 ~/.vimrc.local 文件内，里面比较重要的是
* set nospell 禁用拼写检查，不禁用会有很多红色波浪线，神烦
* set relativenumber 设置相对行号，这个极其重要，起飞的关键

其他设置看不懂没关系，先放进去再说，以后再慢慢研究

# 五、后续学习
http://vim.spf13.com/

这是 spf13-vim 的官方网站，里面有很多插件介绍和快捷键，有兴趣可以看看，收集插件也是乐趣之一，用的久了自然就会越来越熟悉，这里只是抛砖引玉，希望大家后续发起更多有趣、好用的插件。

