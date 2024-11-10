# ARK PUBLISH

```@orgs/v1/slip
ID: SLIP-9797638-bb4b877
CREATE_TIME: 2024-10-30:09:33:58
TAGS: UNASSIMILATED
CREATOR: amas
```
## 背景介绍 

在ARK体系中，最小的价值单位是SLIP，很多的SLIP可以构成一颗树，SLIP的实现被设计成目录，一方面可以容纳不同形式的附件，另一方面目录本身就是一颗树。

在多人协作的场合下，大家需要用版本控制工具管理好共同的树木。使用`ark slip`系列命令建立的SLIP保存在本地，你可以在这找到SLIP的目录：

```sh
$ ark slip add                                                                                                                                                                 ~/src/ark-base:[130]
SLIP-10764214-4a281a3
$ tree -d ~/.ark/orgs/default
/Users/amas/.ark/orgs/default
└── SLIPS
    ├── SLIP-10764214-4a281a3
```

SLIP有一个不太被察觉的属性：名字空间（namespace），可通过`ark slip -n $namespace`来调整，默认的名字空间是`default`，你可以在指定的名字空间里使用slip的相关操作，其结果互不影响。

接下来我们要谈的是如何把你的SLIP分享给别人，首先我们需要一个git仓库，然后你需要使用`ark publihser add $git`下载到本地。然后你就可以把你本地的SLIP发布到这个仓库里。

```sh
# 增加一个git publisher, ARK将会以仓库的根目录命建立新的名字空间(ark-karma)
$ ark publisher add https://github.com/arkitlabs/ark-karma

# 将本地的SLIP发布到这个仓库(注意：此处选择发布内容时:上下进行选择，TAB用于选中或者取消选中)
$ ark publish

# 编辑这个SLIP，注意此时编辑行为发生在ark-karma名字空间里，最好不要这么做
$ ark slip -n ark-karma edit 
```

这里容易令人混淆的是，publish本身会将SLIP的副本拷贝到其他地方，所以你要记住永远只编辑自己本地的SLIP，然后用publish发布到其他的地方。

> 到目前为止，SLIP基本上实现了最原始的协作，仍有很多不便之处需要改进，距离好用只是时间问题。我有必要再讲一下最小价值单位SLIP，DAO的工作总是从最小价值单位开始，这种工作模式总是强调工作在前（建立SLIP在前），完善工作在后，直到工作获得组织的认可，按照SLIP来完成最后的分配。未来所有的奖励/分配/荣誉计算总是围绕着SLIP进行的，甚至投票，分配，提案等工作也将以SLIP的方式存在。虽然到目前为止我还没有把个人签名功能引入到系统里，但这也是时间问题。
