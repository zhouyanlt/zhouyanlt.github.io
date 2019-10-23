---
layout: post
title:  "Git 提交前通过 luacheck 检查待提交代码"
date:   2019-10-23 19:56:25 +0800
categories: git lua luacheck
---

# Preface
由于脚本语言解释执行的特性，很多低级错误在运行到问题代码时才会报错，而不是像 C++ 这种静态语言那样在编译期就能由编译器检查出来，这就导致有很多本来在开发期就可以避免的问题，拖到线上才被发现。这里给出一个方案，可以在提交代码前，通过 Git 的 `hook/pre-commit` 机制，去做一些脚本代码的静态检查。

我用 Lua 比较多，这里就以 Lua 为例来进行说明。Lua 的静态检查工具基本上只有一个 [luacheck](https://github.com/mpeterv/luacheck) 可用, 安装很简单，可以使用 [luarocks](https://luarocks.org/) 安装（类似 python 的 pip, 是一个 Lua 的包管理器）：
```
luarocks install luacheck
```
安装好以后可以直接在终端使用:
```
luacheck your_lua_file.lua
```

# [luacheck 的配置](https://luacheck.readthedocs.io/en/stable/config.html)
默认的 luacheck 配置比较严格，会报很多警告，比如我们自定义的一些全局变量和函数，这当然是我们不希望看到的，既然要检查了，就要做到整个项目里所有文件都是 0 warnings / 0 errors。

配置方法：
新建 `~/.luacheckrc` 文件, 然后在里面加上下面的内容, 这里面是我们项目的一些符号，大家可以根据自己项目实际需求来添加或删除。
```
-- 这个文件是一个 Lua 文件

-- 每行最大长度，默认 120
max_line_length = 9999

-- 忽略的符号
ignore = {
    "init",
    "exit",
    "accept",
    "response",
    "class",
}

-- 全局变量
globals = {
    "Log",
    "table.empty",
    "table.size",
    "table.merge",
    "table.indexof",
    "table.keys",
    "table.values",
    "table.valuestring",
    "table.copy",
    "table.deepcopy",
    "table.first",
    "table.deepmerge",
    "table.walk",
    "table.clear",
    "string.split",
    "string.ltrim",
    "string.rtrim",
    "string.trim",
    "string.repeated",
    "string.nocase",
    "string.nocasefind",
    "enum",
    "ASSERT",
    "ANSI_COLOR",
    "const",
    "math.round",
}
```

完整的配置说明请查看：https://luacheck.readthedocs.io/en/stable/config.html

# Git pre-commit 配置

进入项目根目录下，然后 `cd .git/hooks`, 进去后输入 `ls -ahl` 会看到以下文件：
```
-rwxr-xr-x   1 zy  staff   478B Jun 26  2018 applypatch-msg.sample
-rwxr-xr-x   1 zy  staff   896B Jun 26  2018 commit-msg.sample
-rwxr-xr-x   1 zy  staff   189B Jun 26  2018 post-update.sample
-rwxr-xr-x   1 zy  staff   424B Jun 26  2018 pre-applypatch.sample
-rwxr-xr-x   1 zy  staff   1.8K Oct 23 19:47 pre-commit.sample
-rwxr-xr-x   1 zy  staff   1.3K Jun 26  2018 pre-push.sample
-rwxr-xr-x   1 zy  staff   4.8K Jun 26  2018 pre-rebase.sample
-rwxr-xr-x   1 zy  staff   544B Jun 26  2018 pre-receive.sample
-rwxr-xr-x   1 zy  staff   1.5K Jun 26  2018 prepare-commit-msg.sample
-rwxr-xr-x   1 zy  staff   3.5K Jun 26  2018 update.sample
```

我们把 pre-commit.sample 文件的后缀名去掉 `mv pre-commit.sample pre-commit`, 然后打开它，里面已经有一些内容了，我在这个基础上加上了 luacheck 的检查，可以直接用我提供的版本覆盖里面的内容：

```bash
#!/bin/sh
#
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.
#
# To enable this hook, rename this file to "pre-commit".

if git rev-parse --verify HEAD >/dev/null 2>&1
then
	against=HEAD
else
	# Initial commit: diff against an empty tree object
	against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi

# If you want to allow non-ASCII filenames set this variable to true.
allownonascii=$(git config --bool hooks.allownonascii)

# Redirect output to stderr.
exec 1>&2

# Cross platform projects tend to avoid non-ASCII filenames; prevent
# them from being added to the repository. We exploit the fact that the
# printable range starts at the space character and ends with tilde.
if [ "$allownonascii" != "true" ] &&
	# Note that the use of brackets around a tr range is ok here, (it's
	# even required, for portability to Solaris 10's /usr/bin/tr), since
	# the square bracket bytes happen to fall in the designated range.
	test $(git diff --cached --name-only --diff-filter=A -z $against |
	  LC_ALL=C tr -d '[ -~]\0' | wc -c) != 0
then
	cat <<\EOF
Error: Attempt to add a non-ASCII file name.

This can cause problems if you want to work with people on other platforms.

To be portable it is advisable to rename the file.

If you know what you are doing you can disable this check using:

  git config hooks.allownonascii true
EOF
	exit 1
fi

# If there are whitespace errors, print the offending file names and fail.
git diff-index --check --cached $against --

for file in `git diff --cached --name-only $against`; do
	#/usr/local/bin/luacheck -g -u -a --no-max-code-line-length --no-max-string-line-length --no-max-comment-line-length $file
    /usr/local/bin/luacheck $file
done
```

这里有几个点要解释下：
### 1. [exec](https://askubuntu.com/questions/525767/what-does-an-exec-command-do) 指令
```
git diff-index --check --cached $against --
```
这一行，原来前面有个 `exec`, 这会导致执行到这一行后，bash 就会退出，导致后续的脚本得不到执行，所以要去掉。

### 2. 行尾空白符的检查
```
git diff-index --check --cached $against --
```
还是这行，它的功能是检查行尾空白的，如果有行尾空白，或任何不必要的空白，就会指错，如果不喜欢可以把这行注释掉。

我是开着的，因为这种空白是不必要的，也会导致 luacheck 警告。

### 3. luacheck 的路径
在 for 循环里用到了 luacheck 的绝对路径, 同学们在用的时候要看下是否自己的也在这里，如果不在，可以用 `which luacheck` 来查看它的绝对路径，然后再改下脚本。

### 4. luacheck 的参数
* -g, --no-global, 不警告全局变量
* -u, --no-unused, 不警告未使用的变量
* -a, --no-unused-args, 不警告未使用的参数

这三个可以加上，也可以不加，不加的话会更严格一点，可能会有一点不方便，我这里就不加了，我喜欢严格一点。

其他的 --no-max 系列，我们在 .luacheckrc 里配完后已经没必要指定了。

# Postface

Git 的 Hook 机制还是很好用的，其他语言也可以用这种方法做一些静态检查，用 Hook 也有其他一些东西可以玩的, 这里提出几个想法，抛砖引玉：
1. 提交后通过钉钉机器人通知到某个群里，让审代码的人可以及时收到通知去审代码。
2. 提交后内网测试服自动同步最新代码，这样就不用 QA 每次都手动更新了。
3. 如果是静态语言，比如 C++，提交后服务器上自动编译一下，看是否能编译通过，如果不通过直接钉钉机器人通知到开发群里。

这基本上是持续集成的领域了，还有很多可玩的，可以多想想，有什么好的想法也可以告诉我。
