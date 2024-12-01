# ARK-TEST

```@orgs/v1/slip
ID: SLIP-11044765-4adbdecbbf
CREATE_TIME: 2024-11-13:19:59:25
TAGS: UNASSIMILATED
CREATOR: amas
```
## 背景介绍 

只顾开发少测试或者测试驱动开发都是过分强调开发或测试方面。实际上开发和测试总是交叉进行的，有时候我们先想要达到的效果，然后再去写程序，有时候我们先写程序，观察结果，调整结果达到最终的预期。这两种方法总是交替进行，我们需要的实际上是避免重复的开发或是避免重复的测试。对于习惯于直接开始编程的人，你需要改变的习惯是把你对程序的测试数据写在对应的测试文件中，而不是写在你的终端里。当你认为开发工作已经结束的时候，原本飘散在命令行里的测试数据已经呈现在ARK-TEST的测试文件之中。

## 使用方法

```sh
# 我们使用func.add命令给ark中增加新的功能
$ ark func.add hello.world
```

```sh
# ark会帮你创建对应的脚本和测试脚本
$ tree hello
hello
├── @test
│   └── hello_test
└── hello.world
```
`hello.world:`（要开发的功能）
```sh
#!/bin/zsh
function hello.world() {
    print ARK CALL $0
}

hello.world "$@"
```
`@test/hello_test:`（对应的测试）

```sh
function test:hello.world:basic-test() {
    # @ : hello.world
    # @ arg1 arg2 ... argN @expect -o $output -e $exit_code
    @
}
```

这时候我们就可以开始调试`hello.world`这个函数了

```sh
$ cd hello
$ ark test 
ARK CALL hello.world
[PASSED][ad38f45][0] : [/Users/amas/src/ark-base/hello/@test/hello_test:4: test:hello.world:basic-test]
```

`ark test`会在当前目录下寻找并执行测试脚本并执行其中的测试函数，测试函数的名字遵循如下约定：

```sh
${test_class:=test}:${test_target}:${test_description:=basic-test}
```

- test_class：测试函数的分类（TEST_CLASS），默认是test，也可以是其他如：demo, fuzz, perf等
- test_target：被测试的函数名，在这个例子是`hello.world`
- test_description：对测试函数的描述，这个是给人看的

## 开始编程

假设我们希望hello.word函数没有输入参数的时候返回1，表示这个函数执行失败

```sh
#!/bin/zsh
function hello.world() {
    [[ -z $argv ]] && return 1
    print $1
}

hello.world "$@"
```
再次执行`ark test`，唯一的一条测试case失败了
```sh
$ ark test
[FAILED][ad38f45][1] : [/Users/amas/src/ark-base/hello/@test/hello_test:4: test:hello.world:basic-test] EXIT CODE ERROR (EXPECTED: 0 | 1)
```

因为这是符合预期的行为，所以这个时候我们要调整对测试结果的判断，在返回1时通过测试。

```sh
# @是测试执行器，或者也可以认为它起到一种代理测试的作用，你把对这个目标函数的测试参数代理给@函数，它帮你完成调用，并且记录必要的信息. @exp之前的部分是输入参数，之后则是对于测试结果的检验。这里用-e(--exit)来检测函数的返回值，我们把预期修改为1，这样就通过了测试
@ @exp -e 1
```

`@exp`已经支持的测试结果检测，包括:

- -e \${exit} : 检测函数的返回值
- -o \${output} : 检测函数的输出结果（STDIN或STDOUT）
- \${sample_id} : 测试结果可以保存在文件中，

> 思考题：你觉得在实际编程过程中还需要支持哪些检测？

我们来解释下常规的检测方式:

```sh
@ hello         @exp -o hello            # 测试输出应该是hello
@ helloworld    @exp -o 'hello*'         # 可以使用zsh的字符串模糊匹配（注意：这不是正则表达式）
@               @exp -e 2 -o 'No parmas' # 返回值是2，且输出结果是'No params'
@ '()'          @exp -o '\\\(\\\)'       # ()在pattern中有特殊含义，需要转意处理 
@ '[]'          @exp -o '\\\[\\\]'       # []在pattern中有特殊含义，需要转意处理 
@ '?'           @exp -o '\\\?'           # 转意
@ '*'           @exp -o '\\\*'           # 转意
@ 'hello world' @exp -o 'hello*'
@ 'hello'       @exp -o '????o'
```

最后，我们来用一下sample检测，注意下日志中有一个固定的7位字符串`ad38f45`，这个值是用测试case的函数名+调用参数（@exp前面的参数）经过sha256计算而来的。当你把这个ID传递给@exp的时候，它会帮你把当前输出保存在一个sample文件里，当下次运行测试用例的时候，如果 sample文件存在，它会自动与之比较。所以你要做的就是调试函数，观察到正确的输出之后，把这个Sample ID贴到@exp最后即可。

```
[FAILED][ad38f45][1] : 
[PASSED][ad38f45][0] :
```

sample文件中可以支持模式匹配，如果你需要比较HTTP返回的json内容，但其中有些字段会随每次请求发生变化，这种情形下你需要手动编辑sample文件，实现模糊匹配。

> TODO: 举个例子

## 在测试case函数里可以使用的变量

测试框架运行的时候会产生很多`TEST_`开头的变量，包括:

| 变量名           | 内容                                          | 举例                         |
| ---------------- | --------------------------------------------- | ---------------------------- |
| TEST_FILE        | 测试脚本的路径                                | /src/ark-base/@/@test/@_test |
| TEST_FUNC        | 当前测试case函数名                            | test:test.env:basic-test     |
| TEST_CLASS       | case分类（test\|demo\|fuzz\|perf）            | test                         |
| TEST_TARGET      | 测试目标函数名，用@可以调用                   |                              |
| TEST_DESC        | case描述                                      |                              |
| TEST_MOD         | 测试目标函数的模块名，测试框架会调用@load加载 |                              |
| TEST_CASE_ID     | case ID，由TEST_TARGET和TEST_ARGV计算         |                              |
| TEST_CASE_LINENO | case中@测试调用所处的行数（相对行数）         |                              |
| TEST_ARGV        | 传递给TEST_TARGET的参数（数组类型）           |                              |
|                  |                                               |                              |
|                  |                                               |                              |

## TODO

1. 支持正则表达式检测
2. 支持文件/目录是否存在的检测
3. 提供一个命令编辑sample文件
4. 设置一个exam TEST_CLASS来实现一个考试
