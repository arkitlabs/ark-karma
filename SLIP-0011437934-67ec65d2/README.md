# derrick

```@orgs/v1/slip
ID: SLIP-0011437934-67ec65d2
CREATE_TIME: 2024-11-18:09:12:14
TAGS: UNASSIMILATED
CREATOR: chenlin
```
## 背景介绍 ：路径的处理

```text
$ E003 /a/b/c/d/e/f/g/h.md
PARENT   : /a/b/c/d/e/f/g/
FILENAME : h.md
EXTENSION: md
FILENAME : h
a b c d e f g
A B C D E F G
```

```shell
#!/bin/zsh

## basename 是去除目录后剩下的名字  
## dirname 是取目录  

# 定义一个名为 parse_path 的函数
function parse_path() {
    # 将第一个参数赋值给变量 filepath
    local filepath=$1
    # 获取文件路径的父目录
    local parent=${filepath:h}
    # 获取文件名
    local filename=${filepath:t}
    # 获取文件扩展名
    local extension="${filename:e}"
    # 获取没有扩展名的文件名
    local filename_no_ext="${filename:r}"
    # 将父目录路径按 '/' 分割成数组
    local path_parts=("${(@s:/:)parent}")
    # 将路径数组中的每个部分转换为大写
    local upper_parts=("${(@U)path_parts}")
    # 打印父目录
    print "PARENT   : $parent"
    # 打印文件名
    print "FILENAME : $filename"
    # 打印文件扩展名
    print "EXTENSION: $extension"
    # 打印没有扩展名的文件名
    print "FILENAME : $filename_no_ext"
    # 打印路径数组
    print "${path_parts[@]}"
    # 打印大写的路径数组
    print "${upper_parts[@]}"
}

# 调用 parse_path 函数并传入一个文件路径
parse_path "/a/b/c/d/e/f/g/h.md"

```

