# ARK-SLIP

```@orgs/v1/slip
ID: SLIP-10890612-d03d970
CREATE_TIME: 2024-11-12:01:10:12
TAGS: UNASSIMILATED
CREATOR: amas
```
## 背景介绍

在ARK中，orgs包用来实现web3组织相关的协作功能。提供基于SLIP的知识管理/分享功能/投票等协作功能。使用`ark orgs`系列命令可以管理本地的组织。一个orgs要有唯一的名字，以及一个git仓库用于存储组织相关的数字资产。

```sh
$ ark orgs.root # 查看slip的保存目录
$ ark slip add  # 新建一个SLIP,会顺便建立一个orgs并设置为default orgs, orgs的名字使用$USER环境变量
$ ark orgs ls   # 查看现有的组织
amas
$ ark orgs add https://github.com/arkitlabs/ark-karma.git # 增加一个远程组织
$ ark orgs ls
amas
ark-karma
$ ark slip -n ark-karma # 查看ark-karma下的SLIPS
$ ark use ark-karma     # 改变default orgs (注意：目前没有限制切换到远程组织，所以修改后其实slip add等操作都会直接发生在这个仓库里，正常流程应该是使用ark publish功能)

# 发布SLIP, 跟之前操作一样
$ ark publish
```

个人使用：

1. 首先建议你自己的SLIP可以保存到GITHUB的仓库里，需要你建立一个空仓库
2. 使用`ark orgs add $git`命令导入这个仓库
3. 使用`ark use`命令把这个orgs设置为默认orgs
4. SLIP操作后，ark会帮你同步git仓库
5. 使用`ark orgs sync`命令同步orgs仓库

## TODO: 

设计一个命令

```sh
$ ark slip -d
```

监控剪切版，如果发现匹配的内容，则按照某种规则建立新的SLIP，或者将剪切板内容整理到指定的SLIP里。



## 阅读SLIP

非常关键的设计在于SLIP的随机阅读，或者这个功能也可以做成随机拼接2个SLIP投喂给读者。

```zsh
# 阅读3篇SLIP
$ ark read -n 3
```

## SLIP的变种

目前我一直没有给SLIP增加类型，原因是还没有想好如何继续分割。比如会议/投票/分配/提案/工作很多信息都可以看作是某种类型的SLIP。这是很容易想到的一个方法。

## SLIP的检索

1. 通过SLIP名称检索
2. 通过SLIP的TAG进行检索
3. 通过SLIP中包含的文字检索

## 版本控制

本地的SLIP需要纳入版本控制系统
