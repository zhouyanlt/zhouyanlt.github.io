---
layout: post
title:  "ANSI Common Lisp 第二章参考答案"
date:   2018-10-01 20:26:25 +0800
categories: lisp
---

习题链接：
https://acl.readthedocs.io/en/latest/zhCN/ch2-cn.html

下面是我写的答案，仅供参考
# 1
描述下列表达式求值之后的结果：
```
(a) (+ (- 5 1) (+ 3 7))
(b) (list 1 (+ 2 3))
(c) (if (listp 1) (+ 1 2) (+ 3 4))
(d) (list (and (listp 3) t) (+ 1 2))
```
* (a) 14
* (b) (1 5)
* (c) 7
* (d) (nil 3)

# 2
给出 3 种不同表示 (a b c) 的 cons 表达式 。
```lisp
(cons 'a '(b c))
(cons 'a (cons 'b '(c)))
(cons 'a (cons 'b (cons 'c')))
```

# 3
使用 car 与 cdr 来定义一个函数，返回一个列表的第四个元素。
```lisp
(defun my-fourth (x)
    (car (cdr (cdr (cdr x))))
)
```
# 4
定义一个函数，接受两个实参，返回两者当中较大的那个。
```lisp
(defun my-greater (x y)
    (if (> x y) x y)
)
```
# 5
这些函数做了什么？
```lisp
(a) (defun enigma (x)
      (and (not (null x))
           (or (null (car x))
               (enigma (cdr x)))))

(b) (defun mystery (x y)
      (if (null y)
          nil
          (if (eql (car y) x)
              0
              (let ((z (mystery x (cdr y))))
                (and z (+ z 1))))))
```
#### a
该函数的作用：  
判断一个列表中是否有 nil 或 (),有就返回T,没有就返回NIL
```
[46]> (enigma '(1 2 3 4 5))
NIL
[47]> (enigma '(1 2 nil 4 5))
T
[48]> (enigma '(1 2 3 () 5))
T
```

源码解析
```lisp
(defun enigma (x)
    ;if x不为空 then
    (and (not (null x))
        ;这个or的第一个括号里，判断第一个元素是不是null
        ;如果是整个函数就返回了，不会再递归下去
        ;空表和nil都会被null函数视为true
        (or (null (car x))
            (enigma (cdr x))
        )
    )
)
```

对 null 函数的试验
```
[41]> (null ())
T
[42]> (null 1)
NIL
[43]> (null '(1 2))
NIL
[44]> (null nil)
T
```

#### b
函数作用：  
x 是一个数字   
y 是一个列表  
计算 y 列表中，第一个等于 x 的元素前面总共有几个元素

```
[51]> (mystery 1 '(1 2 3 4 5 6 7))
0
[56]> (mystery 1 '(2 3 4 5 6 7 1 1 1))
6
[57]> (mystery 1 '(2 3 4 5 1 6 7 1 1))
4
[58]> (mystery 1 '(2 3 4 5 6 7))
NIL
```
这个函数看的我头都大了，最后结合试验结果才看明白。还是考验对递归函数的理解能力。

```lisp
(defun mystery (x y)
    (if (null y)
        nil
        ;如果第一个元素等于x，则不再往下递归，返回0
        (if (eql (car y) x)
            0
            ;前面缓存起来的递归调用栈（不等于x的元素），这里会一一累加
            (let ((z (mystery x (cdr y)) ))
                (and z (+ z 1))
            )
        )
    )
)
```

# 6
Q: 下列表达式， x 该是什么，才会得到相同的结果？
```
(a) > (car (x (cdr '(a (b c) d))))
    B
(b) > (x 13 (/ 1 0))
    13
(c) > (x #'list 1 nil)
    (1)
```
A:
#### a
`car`
```
[59]> (car (car (cdr '(a (b c) d))))
B
```
#### b
`or`
#### c
`apply`  

这里不能填`funcall`, 因为`funcall`会把最后的nil也做为参数传给list函数，而apply只接受两个参数: 函数和参数列表。

> apply 接受一个函数和实参列表

# 7 
Q: 只使用本章所介绍的操作符，定义一个函数，它接受一个列表作为实参，如果有一个元素是列表时，就返回真。
```lisp
(defun answer-7 (x)
    (if (null x)
        nil
        (or (listp (car x))
            (answer-7 (cdr x))
        )
    )
)
```

这里有个小细节，我第一版写的是下面这样的：
```lisp
(defun answer-7 (x)
    (or (listp (car x))
        (answer-7 (cdr x))
    )
)
```
但是这样写永远都返回true, 我觉得很奇怪，正常的话，最后一个递归调用是 listp nil,
应该不会返回true啊，结果后面试验了下，发现listp函数在传入nil时居然返回T！！

```
[84]> (listp nil)
T
```
好吧，那只能加上 if 判断一下了。

# 8: 给出函数的迭代与递归版本
### a. 接受一个正整数，并打印出数字数量的点。
```lisp
;迭代版本
(defun answer-8a-iter (x)
    (do ((i 1 (+ i 1)))
        ((> i x) 'done)
        (format t ".")
    )
)

;递归版本
(defun answer-8a-recursion (x)
    (if (<= x 0)
        'done
        (progn
            (format t ".")
            (answer-8a-recursion (- x 1))
        )
    )
)

;说来惭愧，这两个写了好久，最后还是参考了第二章里讲迭代时的那个例子才写出来，其实完全一样，就稍微变形了一点。
```

### b. 接受一个列表，并返回 a 在列表里所出现的次数。
```lisp
;迭代版本
(defun answer-8b-iter (lst)
    (let ((times 0))
        (dolist (obj lst)
            (if (eql obj 'a)
                (setf times (+ times 1))
            )
        )
        times
    )
)

;递归版本
(defun answer-8b-recursion (lst)
    (if (null lst)
        0
        (+ (if (eql 'a (car lst)) 1 0)
           (answer-8b-recursion (cdr lst))    
        )
    )
)
```

# 9
一位朋友想写一个函数，返回列表里所有非 nil 元素的和。他写了此函数的两个版本，但两个都不能工作。请解释每一个的错误在哪里，并给出正确的版本。
```lisp
(a) (defun summit (lst)
      (remove nil lst)
      (apply #'+ lst))

(b) (defun summit (lst)
      (let ((x (car lst)))
        (if (null x)
            (summit (cdr lst))
            (+ x (summit (cdr lst))))))
```
解答：
### a
因为remove不会改为lst的内容，只会把删除nil的新列表以返回值的形式返回。

正确的做法应该是直接在apply里接受remove的返回值，这里注意不能用setf去改lst的值，因为这产生了副作用，不符合Lisp的“精神”。
```lisp
(defun summit (lst)
    (apply #'+ (remove nil lst))
)
```

### b
很显然b是用递归实现的，首先，我想不看题中错误代码自己实现一个。免得在错误的方向越陷越深。
```lisp
(defun summit (lst)
    (if (null lst)
        0
        (+ (if (numberp (car lst)) (car lst) 0)
           (summit (cdr lst))
        )
    )
)

;上面这个实现能正常工作，但是重复调用了两次 car lst, 不能忍，下面是看了题目的错误代码后受到启发而修改的版本
(defun summit (lst)
    (if (null lst)
        0
        (let ((x (car lst)))
            (+ (if (numberp x) x 0)
               (summit (cdr lst))
            )
        )
    )
)
```

再去看题中的实现，错误百出：
* 没有判断lst是不是null,递归没有退出条件。
* 判断 x 时用的是 null,而不是numberp,这样如果元素是个symbol，就会报错，当然这里可能出题人故意忽略这个问题的，但这确实是个bug.
* if 下面的 then 和 else 子句有重复代码，这个不影响功能

所以这个函数只要加上退出条件，就可以正常工作了:
```lisp
; Fixed version
(defun summit (lst)
    (if (null lst)
        0;这里不能返回nil,否则下面+的时候会报错
        (let ((x (car lst)))
            (if (null x)
                (summit (cdr lst))
                (+ x (summit (cdr lst)))
            )
        )
    )
)
```

***
以上就是所有第二章的习题了，做完后有点入门的感觉了，起码获得了一点Lisp的编程体验。
