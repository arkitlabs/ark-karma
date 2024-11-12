# ZSH学习指南

```@orgs/v1/slip
ID: SLIP-10961916-4111fd7
CREATE_TIME: 2024-11-12:20:58:36
TAGS: UNASSIMILATED
CREATOR: amas
```
## 背景

本文介绍ZSH变量的常见操作，比如大小写转换，默认值处理，字符串的查找删除替换，以及数组/字典的常用操作。个别例子可能会有问题，请反馈给我。建议随便看，边看边练习。

# Zsh Parameter Expansion

 - Parameter即等同Variable
 - 当我们说${var}时,指的是变量var的值


## 变量的定义与赋值
### 单引号和双引号
如果你的'value'中包含七七八八的奇怪字符,你可以使用单引号将value括起:
```zsh
$ echo ${var:-'%^&*()_+-=<>?:"{}[];./,'} 

# 但是如果我需要"'"该怎么办呢?
$ tmp="abcd*'\""
$ echo ${var:-$tmp}
```
### 使用vared改变变量的值
``` zsh
$ x=
# 终止编辑: CTRL-d
# 输入回车: ALT-enter
$ vared x
1

2

3
$ print $x
1

2

3
```

### 变量是否存在: ${+name} 或 ${+map[key]}
如果已经定义了name, 则返回1, 否则返回0
```zsh
$ x=0
$ print ${+x}
1
$ print ${+undefined}
0

$ typeset -A map
$ map[key1]=1
$ print ${+map[key1]}
1
$ print ${+map[undefined]}
0
```

### 设置默认值但不定义变量: ${var:-value}
 - 若var未被定义或${var}为空, 返回'value', 否则返回${var}。
 - 若var未定义，zsh也不会为你定义此变量

```zsh
$ echo ${var:-value}
value
$ var=hola
$ echo ${var:-value}
hola

# zsh不会顺便定义此变量
$ print ${not_define:-value}
value
$ (( ${+not_define} )) && print defined  
```
### 定义匿名变量: ${:-value}
```zsh
$ echo ${:-hello}

$ print -l ${(s::):-hello}
h
e
l
l
o

$ print ${(j:-:)${(s::):-hello}}
h-e-l-l-o

$ print ${(j:-:)${(Os::):-hello}}
o-l-l-h-e
```
### 只在未定义变量时返回默认值: ${var-value}
若var未定义, 返回'value', 否则返回${var}
```zsh
# var已定义,但值为空,比较${var-value}与${var:-value}的输出结果
$ var=''
$ echo ${var-value}

$ echo ${var:-value}
value
```

### 定义变量且返回默认值: ${var:=value}
若${var}为空或未定义, 则先定义var, 而后赋值为'value'.
```zsh
$ var=''
$ echo ${var:=value} # 这里包含了定义变量的操作
value
$ echo $var
value
```


我们经常利用这种形式为变量指定默认值:
```zsh
function just-do-it() {
    # 若未指定路径, 则默认使用当前路径
    local path=${$1:=.}
    ...
}
```

## 强大的Modifiers
	- http://zsh.sourceforge.net/Doc/Release/Expansion.html#Expansion

```zsh
$ pwd 
/home/amas

$ print ${:-Let\'s go}
Let's go

# a|A|P : 尽可能转换为绝对路径
$ print ${${:-.}:a}
/home/amas
$ print ${${:-dir}:a}
/home/amas/dir

# c   : 查找命令的可执行文件路径
$ print ${${:-dir}:c}
/usr/bin/ls

# e   : 获取文件名扩展名
$ print ${${:-hello.world.pdf}:e}
pdf
# r   : 获取文件名
$ print ${${:-/a/b/c/d}:r[1]}
hello.world

# h[N]: 获取父目录，类似于dirname (HEAD)
$ print ${${:-/a/b/c/d}:h}
/a/b/c
$ print ${${:-/a/b/c/d}:h1}
/
$ print ${${:-/a/b/c/d}:h2}
/a
$ print ${${:-/a/b/c/d}:h3}
/a/b

# t[N]: 类似basename (TAIL)
$ print ${${:-/a/b/c/d}:t}
d
$ print ${${:-/a/b/c/d}:t1}
d
$ print ${${:-/a/b/c/d}:t2}
c/d
$ print ${${:-/a/b/c/d}:t3}
b/c/d

# Q   : 去掉一层引号
$ print ${${:-'"hello"'}}
"hello"
$ print ${${:-'"hello"'}Q}
hello

# 字符串替换
# s/l/r    : 从左替换第一个匹配的
# gs/l/r   : 全局替换
# s/l/r/:& : &可以重复上一次替换操作
$ print ${${:-hello}:s/l/x}
hexlo
$ print ${${:-hello}:gs/l/x}
hexxo
$ print ${${:-hellllo}:gs/l/x/:&}
hexxllo
$ print ${${:-hellllo}:gs/l/x/:&:&}
hexxxlo

# u|l : 大小写
$ print ${${:-hello}:u}
HELLO
$ print ${${:-hello}:u:l}
hello
$ print ${${:-hello}:s/l/x}
```



### ${var::=value}

总是声明var,并为其赋值为'value', 相当于:
```zsh
$ unset var
$ var='value' 
```
这个表达式有点强劲, 它将置之既有变量定义于不顾, 完全覆盖为新的定义.

```zsh
# 定义非空变量var
$ var='my god'

# 覆盖掉之前的定义
$ echo ${var::=oops}
oops
$ echo $var
oops

# ':='操作要温和的多, 它并不影响已经定义的非空变量
$ echo ${var:=hola}
oops
```

### 没有定义变量的时候立刻终止: ${var:!?}
如果没有定义var, 则立即终止(返回值为-1)
```zsh
$ echo ${var:?}
zsh: var: parameter not set
```

### ${var:+value}
如果${var}非空, 则返回'value'. var的值并不会被修改.
```
$ var=hola
$ echo ${var:+oops}
oops
$ echo $var
hola
```

### 从文件中读取数据: $(< [file|&0] ) 
我们也可以从外部文件读取数据. 特别的**&0**代表stdin

```zsh
$ x=$(</etc/hostname)
localhost

$ x=$(<&0)
12345      # 输入完成之后按C-d结束
$ print $x
12345 


# pipe
$ print "hello pipe" | function(){ print ${(U)$(<&0)}}
HELLO PIPE
$ print "hello pipe" | ( print ${(U)$(<&0)} )
HELLO PIPE
$ print "hello pipe" | { print ${(U)$(<&0)} }
HELLO PIPE
```


## 删除与替换
### ${var#regex}
从头部保守删除符合regex的子串
```zsh
$ x="a.a.b.b.c.c"
$ echo ${x#*a}
.a.b.b.c.c
$ echo ${x#*b}
.b.c.c
```

### 从前删除: ${var##regex}
从头部贪婪删除符合regex的子串
```zsh
$ x="a.a.b.b.c.c"
$ echo ${x##*a}
.b.b.c.c
$ echo ${x##*b}
.c.c
```

### 从后删除: ${var%regex}
从尾部保守删除符合regex的子串

获取不包含后缀的文件名
```zsh
$ file=README.txt
$ echo ${file%.*}
README
```

### 从后删除 ${var%%regex}
从尾部贪婪删除符合regex的子串
```zsh
$ x=README.en.txt
$ echo ${x%.*}
README.en
$ echo ${x%%.*}
README
```

### 替换第一个匹配: ${var/regex/string}
将${var}中第一个符合regex的字串替换为string.
```zsh
$ var="README.en.txt"
$ echo ${var/en/zh}
README.zh.txt
```

### 全部替换: ${var!//regex/string}
将${var}中全部符合regex的子串替换为string
```zsh
$ x=a/b/c./d
$ echo ${x//\//.} 
a.b.c.d
```

### 压缩两个数组: ${array1:^array2} 和 ${array1:^^array2}
```zsh
$ xs=(a b c d) ys=(1 2)
$ print ${xs:^ys}
a 1 b 2
$ print ${xs:^^ys}
a 1 b 2 c 1 d 2
```

## 获取子元素
### ${name:offset} 或 ${name[start]}
### ${name:offset:length} 或 ${name[start,end]}
 - 这个方法可以用于数组或字符串
 - 下标从1开始, 而非0
 - 下标可以是负数, -1表示倒数第一个元素

### 关联数组的key集: ${(k)map}

### 关联数组的value列表: ${(v)map}

### 关联数组的key/value:  ${(kv)map}

展开关联数组(map)的key列表，这个不能与下标范围同时使用， 也就是你不能指定取某个范围内的key,

```zsh
$ typeset -A map
$ map=(k1 v1 k2 v2)
$ echo ${(k)map}
k1 k2

# 下标
$ echo ${(k)map}[1] 
zsh: no matches found: k2[1]
```

### 二次展开: ${(P)var}

```zsh
$ x=1
$ y=x
$ print $y
x
$ print ${(P)y}
1

# 关联数组
$ typeset -A map
$ map[a]=1
$ map[b]=2
$ map[c]=3
$ m=map
$ print ${(P)m}        #等价于 ${map}
1 2 3
$ print ${(Pkv)m}      #等价于 ${(kv)map}
a 1 b 2 c 3
$ print ${${(P)m}[a]}  #等价于 $map[a]
1
```


## 过滤
### ${array:#regex}
过滤掉指定的元素
```zsh
$ xs=(aa bb ab ac)
# 过滤掉以a开头的元素
$ print ${xs:#a*}
bb
```

### ${(M)array:#regex}
筛选出符合条件的元素
```zsh
$ xs=(aa bb ab ac)
$ print ${(M)xs:#a*}
aa bb ac
```
### 去掉重复元素: ${(u)array}
数组以集合形式展开，重复元素只保留一个。
```zsh
$ xs=(a a c b e)
$ echo ${(u)xs}
a c b e
```

### 排列 ${^spec}
这是一个非常有趣和有用的功能, 在变量展开的时候可以暂时打开RC_EXPAND_PARAM选项. 具体使用方法是:
 - ^  : 打开RC_EXPAND_PARAM选项
 - ^^ : 关闭RC_EXPAND_PARAM选项

当变量展开的时候, ${^spec}将会与前后的元素进行排列.  

```zsh
$ xs=({1..10})
$ print a${^xs}
a1 a2 a3 a4 a5 a6 a7 a8 a9 a10
$ print a${^xs}b
a1b a2b a3b a4b a5b a6b a7b a8b a9b a10b
$ print a${^^xs}
a1 2 3 4 5 6 7 8 9 10

# 注意: 数组(xs)为空时, ${^spec}将导致整个结果为空.
$ xs=()
$ echo HELLO${xs}WORLD
HELLOWORLD
$ echo HELLO${^xs}WORLD


$ xs=(1 2) ys=(a b) zs=(X Y)
$ print ${^xs}${^ys}${^zs}
1aX 1aY 1bX 1bY 2aX 2aY 2bX 2bY
```

## 获取变量信息

### ${(#)var} 或 $#var

返回列表长度，如果是var是String则返回String的长度。

```zsh
$ x=oops
$ print ${#x}
4

$ xs=({1..100})
$ print $#xs
100
```
### ${(w)#array}
### ${(W)#array}
统计数组中的单词个数.
```zsh
$ xs=(how about " zsh is bad")
$ echo $#xs
3
$ echo ${(w)#xs}
5
$ echo ${(W)#xs}
6
```

有关'W'的作用真是匪夷所思，比如:
```zsh
$ xs=(" " "  ")
$ echo ${(w)#xs}
0
$ echo ${(W)#xs}
5
# 为什么是5?
# 1个空格可以分割2个词
# 2个空格可以分割3个词
# 所以最终结果为5
```

### 数组转化为字符串后的长度: ${(c)#array}
数组中的字符数，包含分隔符（默认就是空格）。
```zsh
$ xs=(H E L L O)
$ echo $#xs
5
$ echo ${(c)#xs}
9
```

### ${(@)var}
将数组中的元素拆分成单独的词。
```zsh
$ var=(a b c d e)
$ (){ print "argc=$#"; print -l $argv  } "$var"
argc=1
argv=a b c d e
$ (){ print "argc=$#"; print -l $argv  } "${(@)var}"
argc=5
a
b
c
d
e
```

### 大写转换: ${(C)var}
以全部大写的方式展开变量。
### 小写转换: ${(L)var}
以全部小写的方式展开变量。
```zsh
$ xs=(h e l l o)
$ echo ${(C)xs}
H E L L O
$ x=hello
$ echo ${(C)hello}
HELLO
```
### 打印变量的类型: ${(t)var}
 - t展开可以让你了解变量的类型
 - 你可以使用'typeset'或者'local'之类的指令明确指定变量类型，这并非必须，因为zsh可以自己做类型推导

> 类型 = 主类型 - 属性

主类型可以是:
 * scalar
 * array
 * integer
 * float
 * association

其他属性可以是:
 * local
 * left
 * right_blanks
 * right_zeros
 * lower
 * upper
 * readonly
 * tag
 * unique
 * hide
 * special : 一些具有特殊身份的变量，一般都是由shell定义的。比如: PATH

```
$ var
$ echo ${(t)var}
scalar
$ var=(a b c)
$ echo ${(t)var}
array

# 观察一下只读变量
$ typeset -r x=1
$ echo ${(t)x} 
scalar-readonly

#
$ echo ${(t)PATH}
scalar-export-special
$ echo ${(t)?}
integer-readonly-special

# 可以使用数组充当集合
$ typeset -U set
$ set=(1 1 2 2 3 3 4 5)
$ echo $set
1 2 3 4 5
$ echo ${(t)set}
array-unique
```

### 求值并展开: ${(e)var}
展开变量时执行[CommandSubstitution]和[ArithmeticExpansion]. 这种展开可以嵌套，但是层级不能太深，容易产生不可预见的效果。
```sh
$ expr='1+2=$(( 1+2 ))'
1+2=$(( 1+2 ))
$ echo ${(e)expr}
1+2=3

# 对数组也是有效的
$ xs=('$(( 1+2 ))' '$((2+4))' '$((3+9))')
$ echo $xs
$(( 1+2 )) $((2+4)) $((3+9))
$ echo ${(e)xs}
3 6 12

# command substitution
$ cmd='$(echo HelloZsh)'
$ echo $cmd
$(echo HelloZsh)
$ echo ${(e)cmd}
HelloZsh
```


## 排序
### ${(a)array}
### 升序: ${(o)array}
### 降序: ${(O)array}
```zsh
$ xs=(1 2 3 4 5)  
$ echo ${(a)xs}
1 2 3 4 5
$ echo ${(O)xs}
5 4 3 2 1
```

### ${(i)var} : 大小写不敏感的排序方式
排序时采用大小写不敏感的方式.
```bash

```

### ${(n)var}

### ${(nO)var}

### ${(no)var}

### ${(ni)var}

- 尽可能按照是十进制整数顺序展开
- 在元素进行比较时，如果它们之间的第一个不同的字符不是数字，则按照字典顺序展开。

```
$ xs=(001 011 012 03)
# 按照字典顺序03是最靠后的
$ echo ${(o)xs}
001 011 012 03

# 按照数字顺序，03的位置发生了变化
$ echo ${(on)xs}
001 03 011 012

```

### ${(o)var}: 升序展开

### ${(oi}var}: 大小写不敏感的升序展开

### ${(O)var}: 降序展开

Sort the resulting words in descending order; 'O' without 'a', 'i' or 'n' sorts in reverse lexical order. May be combined with 'a', 'i' or 'n' to reverse the order of sorting.



## 分割与合并
### ${=spec}
 - 按照**SH_WORD_SPLIT**的设定进行分割
 - ${==spec} : 关闭分割功能

```zsh
$ x="a b c" 
$ echo $x
a b c
$ print -l $x
a b c
$ print -l ${=x}
a
b
c
$ print -l ${==x}
a b c
```

### 用换行回车拆分为数组: ${(f)var}
 - 用换行回车分割输入
 - 等价于`${('ps:\n:'var)}`

num.txt:
```
1

2

3
```

```zsh
$ cat num.txt | wc -l
5

# 注意: 必须加引号
$ xs=("${(f)$(<num.txt)}")
$ echo $#xs
3

# 空元素被滤掉了
$ echo ${(f)xs}
1
2
3

# 需要保留空元素
$ xs=("${(@f)$(<num.txt)}")
$ print -l $xs
1

2

3
```
### 按照shell语法进行分割: ${(z)var}: 
按照shell的语法进行分割, 引号扩起的字符串将被作为一个完整的元素.
```zsh
$ s='"Hello world" This is not bad '
$ print -l ${(s: :)x}
"Hello
world"
This
is
not
bad
$ print -l ${(z)x}
"Hello world"
This
is
not
bad
```

### '\0'作为分隔符: ${(0)var}
 - 等价于: '${(ps:\0:)var}', 使用'\0'作为分隔符。

### ${(s:string:)var}

### ${(ps:string:)var}

使用`string`分割字符串。

```zsh
$ x="a b c"
$ print -l ${(s: :)x}
a
b
c

# 按照:分割
$ x="a:b:c"
$ print -l ${(s=:=)x}
a
b
c

# 默认分割不保留空行
$ x="a::c"
$ print -l ${(s=:=)x}
a
b

# 保留空行
$ print -l "${(@s=:=)x}"
a

b
```
### ${(j:string:)array}
用`string`联接数组中的每个元素。

```zsh 
$ xs=(1 2 3)
$ print ${(j:-:)xs}
1-2-3

# 使用:进行链接
$ print ${(j=:=)xs}
1:2:3
```

### 使用换行回车链接数组所有元素: ${(F)array}
等价于`${('pj:\n:'var)}`， 以'\n'作为数组展开的元素分隔符。
```zsh
$ xs=(1 2 3 4 5)
$ echo "${(F)xs}"
1
2
3
4
5

# 用print也可以办到
$ print -l $xs
1
2
3
4
5
```



## 引号问题
### ${(q)var}
```zsh
$ x="hello\nzsh"
$ echo $x
hello
zsh
$ echo ${(q)x}
hello\nzsh
```

### ${(qq)var}
### ${(qqq)var}
Quote the resulting words with backslashes; unprintable or invalid characters are quoted using the $'\NNN' form, with separate quotes for each octet. If this flag is given twice, the resulting words are quoted in single quotes and if it is given three times, the words are quoted in double quotes; in these forms no special handling of unprintable or invalid characters is attempted. If the flag is given four times, the words are quoted in single quotes preceded by a $.
### ${(Q)var}: 去除外层引号
```
$ x='"egg"'
$ echo $x
"egg"
$ echo ${(Q)x}
egg
```





## 错误处理
### ${(XQ)var}
### ${(X)var#pattern}
### ${(Xe)var}
这个用于改变e,#,Q展开时遇到错误改如何是好, 加上X出现错误不会默认被或略掉
```
$ x='"abc'
$ print ${(Q)x}
"abc
$ print ${(XQ)x}
zsh: unmatched "
```


## 其他

### ${(p)var}
Recognize the same escape sequences as the print builtin in string 
arguments to any of the flags described below that follow this argument.

### ${(~)var}

Force string arguments to any of the flags below that follow within the parentheses
 to be treated as patterns. 
 Compare with a ~ outside parentheses, 
 which forces the entire substituted string to be treated as a pattern.
  Hence, for example,

[[ "?" = ${(~j.|.)array} ]]
with the EXTENDED_GLOB option set succeeds if and only if $array contains the string '?' as an element. 
The argument may be repeated to toggle the behaviour; its effect only lasts to the end of the parenthesised group.




## 格式化打印
### ${(l:N:)var}
### ${(r:N:)var}
 - l : 左留白
 - r : 右留白
 - N : 显示宽度,单位是字符数,如果变量的内容超过N则会发生截断

```
$ x="123456"
$ print "'${(l:10:)x}'"
'    123456'
$ print "'${(r:10:)x}'"
'123456    '
$ print "'${(l:4:)x}'"
'3456'
$ print "'${(r:4:)x}'"
'1234'
```
### ${(m)var}

### ${(r:expr::string1::string2:)var}
As l, but pad the words on the right and insert string2 immediately to the right of the string to be padded.
Left and right padding may be used together. In this case the strategy is to apply left padding to the first half width of each of the resulting words, and right padding to the second half. If the string to be padded has odd width the extra padding is applied on the left.



### ${(S)var}
Search substrings as well as beginnings or ends; with # start from the beginning and with % start from the end of the string. With substitution via ${.../...} or ${...//...}, specifies non-greedy matching, i.e. that the shortest instead of the longest match should be replaced.

### ${(I:expr:)var}
Search the exprth match (where expr evaluates to a number). This only applies when searching for substrings, either with the S flag, or with ${.../...} (only the exprth match is substituted) or ${...//...} (all matches from the exprth on are substituted). The default is to take the first match.
The exprth match is counted such that there is either one or zero matches from each starting position in the string, although for global substitution matches overlapping previous replacements are ignored. With the ${...%...} and ${...%%...} forms, the starting position for the match moves backwards from the end as the index increases, while with the other forms it moves forward from the start.
Hence with the string

which switch is the right switch for Ipswich?
substitutions of the form ${(SI:N:)string#w*ch} as N increases from 1 will match and remove 'which', 'witch', 'witch' and 'wich'; the form using '##' will match and remove 'which switch is the right switch for Ipswich', 'witch is the right switch for Ipswich', 'witch for Ipswich' and 'wich'. The form using '%' will remove the same matches as for '#', but in reverse order, and the form using '%%' will remove the same matches as for '##' in reverse order.


### ${(B)var}
Include the index of the beginning of the match in the result.

### ${(E)var}

Include the index of the end of the match in the result.

### ${(M)var}

Include the matched portion in the result.

### ${(N)var}

Include the length of the match in the result.

### ${(R)var}

Include the unmatched portion in the result (the Rest).

###  代换规则(substitutions rule)

```
Here is a summary of the rules for substitution; this assumes that braces are present around the substitution, i.e. ${...}. Some particular examples are given below. Note that the Zsh Development Group accepts no responsibility for any brain damage which may occur during the reading of the following rules.
=== 1. Nested Substitution
If multiple nested ${...} forms are present, substitution is performed from the inside outwards. At each level, the substitution takes account of whether the current value is a scalar or an array, whether the whole substitution is in double quotes, and what flags are supplied to the current level of substitution, just as if the nested substitution were the outermost. The flags are not propagated up to enclosing substitutions; the nested substitution will return either a scalar or an array as determined by the flags, possibly adjusted for quoting. All the following steps take place where applicable at all levels of substitution. Note that, unless the '(P)' flag is present, the flags and any subscripts apply directly to the value of the nested substitution; for example, the expansion ${${foo}} behaves exactly the same as ${foo}.
At each nested level of substitution, the substituted words undergo all forms of single-word substitution (i.e. not filename generation), including command substitution, arithmetic expansion and filename expansion (i.e. leading ~ and =). Thus, for example, ${${:-=cat}:h} expands to the directory where the cat program resides. (Explanation: the internal substitution has no parameter but a default value =cat, which is expanded by filename expansion to a full path; the outer substitution then applies the modifier :h and takes the directory part of the path.)
=== 2. Internal Parameter Flags
Any parameter flags set by one of the typeset family of commands, in particular the L, R, Z, u and l flags for padding and capitalization, are applied directly to the parameter value.
=== 3. Parameter Subscripting
If the value is a raw parameter reference with a subscript, such as ${var[3]}, the effect of subscripting is applied directly to the parameter. Subscripts are evaluated left to right; subsequent subscripts apply to the scalar or array value yielded by the previous subscript. Thus if var is an array, ${var[1][2]} is the second character of the first word, but ${var[2,4][2]} is the entire third word (the second word of the range of words two through four of the original array). Any number of subscripts may appear.
=== 4. Parameter Name Replacement
The effect of any (P) flag, which treats the value so far as a parameter name and replaces it with the corresponding value, is applied.
=== 5. Double-Quoted Joining
If the value after this process is an array, and the substitution appears in double quotes, and no (@) flag is present at the current level, the words of the value are joined with the first character of the parameter $IFS, by default a space, between each word (single word arrays are not modified). If the (j) flag is present, that is used for joining instead of $IFS.
=== 6. Nested Subscripting
Any remaining subscripts (i.e. of a nested substitution) are evaluated at this point, based on whether the value is an array or a scalar. As with 2., multiple subscripts can appear. Note that ${foo[2,4][2]} is thus equivalent to ${${foo[2,4]}[2]} and also to "${${(@)foo[2,4]}[2]}" (the nested substitution returns an array in both cases), but not to "${${foo[2,4]}[2]}" (the nested substitution returns a scalar because of the quotes).
=== 7. Modifiers
Any modifiers, as specified by a trailing '#', '%', '/' (possibly doubled) or by a set of modifiers of the form :... (see the section 'Modifiers' in the section 'History Expansion'), are applied to the words of the value at this level.
=== 8. Forced Joining
If the '(j)' flag is present, or no '(j)' flag is present but the string is to be split as given by rules 8. or 9., and joining did not take place at step 4., any words in the value are joined together using the given string or the first character of $IFS if none. Note that the '(F)' flag implicitly supplies a string for joining in this manner.
=== 9. Forced Splitting
If one of the '(s)', '(f)' or '(z)' flags are present, or the '=' specifier was present (e.g. ${=var}), the word is split on occurrences of the specified string, or (for = with neither of the two flags present) any of the characters in $IFS.
=== 10. Shell Word Splitting
If no '(s)', '(f)' or '=' was given, but the word is not quoted and the option SH_WORD_SPLIT is set, the word is split on occurrences of any of the characters in $IFS. Note this step, too, takes place at all levels of a nested substitution.
=== 11. Uniqueness
If the result is an array and the '(u)' flag was present, duplicate elements are removed from the array.
=== 12. Ordering
If the result is still an array and one of the '(o)' or '(O)' flags was present, the array is reordered.
=== 13. Re-Evaluation: 小e展开
Any '(e)' flag is applied to the value, forcing it to be re-examined for new parameter substitutions, but also for command and arithmetic substitutions.
=== 14. Padding
Any padding of the value by the '(l.fill.)' or '(r.fill.)' flags is applied.
=== 15. Semantic Joining
In contexts where expansion semantics requires a single word to result, all words are rejoined with the first character of IFS between. So in '${(P)${(f)lines}}' the value of ${lines} is split at newlines, but then must be joined again before the P flag can be applied.
If a single word is not required, this rule is skipped.


Examples

The flag f is useful to split a double-quoted substitution line by line. For example, ${(f)"$(<file)"} substitutes the contents of file divided so that each line is an element of the resulting array. Compare this with the effect of $(<file) alone, which divides the file up by words, or the same inside double quotes, which makes the entire content of the file a single string.
The following illustrates the rules for nested parameter expansions. Suppose that $foo contains the array (bar baz):

"${(@)${foo}[1]}"
This produces the result b. First, the inner substitution "${foo}", which has no array (@) flag, produces a single word result "bar baz". The outer substitution "${(@)...[1]}" detects that this is a scalar, so that (despite the '(@)' flag) the subscript picks the first character.
"${${(@)foo}[1]}"
This produces the result 'bar'. In this case, the inner substitution "${(@)foo}" produces the array '(bar baz)'. The outer substitution "${...[1]}" detects that this is an array and picks the first word. This is similar to the simple case "${foo[1]}".
As an example of the rules for word splitting and joining, suppose $foo contains the array '(ax1 bx1)'. Then
${(s/x/)foo}
produces the words 'a', '1 b' and '1'.
${(j/x/s/x/)foo}
produces 'a', '1', 'b' and '1'.
${(s/x/)foo%%1*}
produces 'a' and ' b' (note the extra space). As substitution occurs before either joining or splitting, the operation first generates the modified array (ax bx), which is joined to give "ax bx", and then split to give 'a', ' b' and ''. The final empty string will then be elided, as it is not in double quotes.


从这里开始，flag可以接收额外的参数， 这些参数默认以':...:'作为分割符，除了小p之外，任何相同的字符('a...a', 'x...x')，或者符号对儿(比如: '(...)', '[...]', '<...>')都可以作为分隔符。
```

这里面的'...'为一个参数，如果有多个参数，必须确保所有的参数都被分隔符包围。


你可能注意到，它们的效果都是一样的:
```zsh
$ x=(1 2 3)
$ echo ${(j:--:)x}
1--2--3
$ echo ${(ja--a)x}
1--2--3
$ echo ${(j{--})x}
1--2--3
```


### ${(A)array=elem1 elem2 ... elemeN}
### ${(A)array:=elem1 elem2 ... elemN}
### ${(A)array::=elem1 elem2 ... elemN}
有关'...=...',`...:=...`,`...::=...`的使用方法请参看前面的章节，经过`A`修饰后，将建立数组变量。

## ${~var}
允许FileGlobbing.
```zsh
$ ls
dir1 dir2
$ var=*
$ echo $var
*
$ echo ${~var}
dir1 dir2
```

### ${(v)var}

Make any special characters in the resulting words visible.

## 控制数组
### 声明数组:
```zsh
set -A name
typeset -a name
```

### 定义数组:
```zsh
set -A name 1 2 3 4 5 6
typeset -a name ; set -A name 1 2 3 4 5 6
name=(1 2 3 4 5 6)
```

### 索引
假如: 
```zsh
xs=(a b c d)
```

```zsh
$ echo $xs[1]
a
$ echo $xs[-1]
a
$ echo ${xs[3]}
c
$ echo ${xs[$((1+1))]}
b
$ echo $xs[1,3] 
a b c
```

### 数组和字符串
字符串可以看作字符数组.
```zsh
$ x=1234567
$ echo $x[1,3]
123
```

### 数组的长度
```zsh
$ xs=({1..10})
$ echo ${#xs}
10
$ xs=({1..100})
$ echo ${#xs}
100
```

```zsh
# 从数组中随机挑选元素
$ birthday=({1..12})
$ echo "$birthday[$[${RANDOM}%${#birthday}+1]]"
```


### 追加元素: +=
> array+=element|array
> 比如:
```zsh
$ nums=({1..3})  ; echo $nums
1 2 3 
$ nums+=4        ; echo $nums
1 2 3 4
$ nums+=({5..9}) ; echo $nums
1 2 3 4 5 6 7 8 9
```

### 查找元素的下标: $array[(i)pattern]
 - 从左向右依次查找(反向查找使用:I)
 - 如果匹配pattern, 则返回该元素的下标
 - 如果零匹配，则返回数组长度+1(即: $#array+1)

```zsh
$ x=(apple age ago)
$ print $x[(i)ago]
3
$ print $x[(i)notfound]
4
$ print $x[(i)a*]
1

# 通常都是这么判断数组中是否包含某个元素
$ [[ $x[(i)ago] -le $#x ]] && print "found: ago"
found: ago
```

### 查找元素: $array[(r)pattern]: 
 - 从左向右依次查找(反向查找使用:R)
 - 如果匹配pattern, 则返回该元素
 - 如果无法匹配，则返回空

```zsh
$ x=(apple age ago)
$ print $x[(r)ago]
ago
$ print $x[(r)notfound]

$ print $x[(i)a*]
ago

# 通常都是这么判断数组中是否包含某个元素
$ [[ -n $x[(r)ago] ]]  && print "found: ago"
found: ago
```

### 删除元素: $array[n]=()
```zsh
$ x=(a b c)
$ x[2]=
$ print $x
a c
```

### 获取全部元素: $array[@] 和 $array[*]
 - 他们都会扩展为数组的所有元素
 - "$X[@]" 等价于 $X[*]
 - "$X[@]" 与 "$X[*]"意义不同`
  - "$X[@]" : 相当于 "$1" "$2" ... "$N"` (N个参数)
  - "$X[*]" : 相当于 "$1 $2 ... $N"` (1个参数)

例如:
我们用echo-args打印参数明细:
```
#!sh
#----------------------
#!/bin/zsh

msgI() {
    echo "$FG[green]$*"
}

msgI '[ARGS VALUES]':$*
msgI '[ARGS COUNTS]':$#
#----------------------
```

做如下试验:
```
$ ./echo-args $xs[*]                                                    /data/src/zsh/array
[ARGS VALUES]:a b c d
[ARGS COUNTS]:4
$ ./echo-args $xs[@]                                                    /data/src/zsh/array
[ARGS VALUES]:a b c d
[ARGS COUNTS]:4
$ ./echo-args "$xs[*]"                                                  /data/src/zsh/array
[ARGS VALUES]:a b c d
[ARGS COUNTS]:1
$ ./echo-args "$xs[@]"                                                  /data/src/zsh/array
[ARGS VALUES]:a b c d
[ARGS COUNTS]:4
```

### 笛卡尔积: $^array
 - 数组本身构成了一个数据集合(不考虑有相同元素的情形)，2个集合以上可进行求笛卡尔积运算. '$!^^算符即可完成此运算.
 - $^告诉zsh在expansion时按照笛卡尔积展开

```zsh
$ x=(a b c)
$ y=(1 2 3)
$ z=(A B C)
$ print $^x$^y
a1 a2 a3 b1 b2 b3 c1 c2 c3
$ print $^x$^y$^z
a1A a1B a1C a2A a2B a2C a3A a3B a3C b1A b1B b1C b2A b2B b2C b3A b3B b3C c1A c1B c1C c2A c2B c2C c3A c3B c3C
```

## 关联数组
关联数组是特殊的数组, 其元素个数总是偶数, 保存多个key/value. 其数据类型需要通过typeset -A 来指定, 指定后可以使用数组的赋值方式.
```zsh
typeset -A name
name=(k1 v1 k2 v2 ... kx vx)
```

### 定义关联数组:
```zsh
$ typeset -A people 
$ set -A people name x sex F 
$ echo $people
x F

# 等同于
$ typeset -A people
$ people=(name x sex F)

# 等同于
$ typeset -A people
$ people[name]=x
$ people[sex]=F

# 等同于
$ typeset -A people
$ typeset "people[name]=x"
$ typeset "people[sex]=F"
```

### 打印所有key:
```zsh
$ echo ${(k)people}
```

### 打印所有value:
```zsh
$ echo ${(v)people}
$ echo $people
$ echo ${people[*]}
$ echo ${people[1,-1]}

# for可以绑定两个以上的变量
for key value in $people; do
    print $key=$value
done

# 健壮性更好的方式如下，可以兼容value为空的情况
for k v in "${(Pkv)${map}[@]}"; do
    print $k $v
done
```

### 删除键值
```zsh
$ typeset -A people
$ people=(:name x :sex F :age 18)
$ print $people
18 x F
$ unset "people[:sex]" # 注意不带'$'号!!!
$ print $people
18 x
```

### 反转数组: ${(Oa)array}
```zsh
$ x=(a b c d e)
$ echo $x
a b c d e
$ echo ${(Oa)x}
e d c b a
```

## Brace Expansion
讲讲数组的BraceExpansion
比如:
```zsh
$ xs={m n q}
$ echo X{x,$xs}Y
$ echo X${^xs}Y
$ echo X${xs}Y
```
