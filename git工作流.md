# 用手画了11张图终于搞明白了Git工作流，我怀疑你用的是假 Git

[原文章](https://developer.aliyun.com/article/940308)

> 简介： 用手画了11张图终于搞明白了Git工作流，我怀疑你用的是假 Git

## 集中式工作流
---
集中式工作流这种工作方式对于使用过SVN的同学想必会非常的熟悉，让我们思考下在 SVN下的协作体验，不同的开发同学需要依次将本地的修改提交到服务器，如果有冲突就先解决本地的冲突再提交，这个过程中远端的服务器就像是一个集中管理者，管理着所有人的代码提交，所以 SVN的开发协作流程就是典型的集中式工作流。

![SVN集中式](/images/SVN集中式.png)

如果切换到 Git 来维护代码仓，但是开发人员又对 Git 的分支模式不熟悉，能不能用 Git 实现类似的集中式工作流呢？答案是当然可以。

![Git仿集中式](/images/Git仿集中式.png)

每个开发人员将远程仓库的代码 clone 下来变成了属于自己的本地仓库，提交代码时先提交至本地仓库，然后再推送到远程仓库。

这种模式相比 SVN 只是多了一个本地仓库而已，有了 SVN 的经验开发人员也很快能熟悉这种模式，在前些年有很多公司都是将 Git 作为 SVN 来用的。

从提交记录来看，集中式工作流通常是一条直线往前走，如下图：

![集中式代码提交流程](/images/集中式代码提交流程.png)

> 小结：这种模式不推荐大家使用，因为完全没有发挥出 Git 的作用，类似于用倚天剑屠龙刀来切菜，太浪费了。

## 功能分支工作流
---

集中式工作流有一个很大的问题，随着团队内人员不断增多，大家每一次提交代码都可能会遇到冲突，提交代码一分钟解决冲突一小时。

为了便于大家并发开展工作，通常会基于 master 主干分支拉取几个特性分支，每个开发人员关注于自己的分支，需要提交代码时直接提交到本地库的特性分支，在合入到主干分支前通常会拉取最新的代码，如果有冲突先在本地解决好冲突，解决完提交 MR 申请将特性分支合入主干分支。

![功能分支工作流1](/images/功能分支工作流1.png)

在功能分支工作流下，不会直接将代码合入到主干分支（master），通常是通过其他分支提交 MR（Merge Request），这使得集成一些自动化操作变得简单可行了。

提交 MR 之后团队成员开始围观你写的代码，可以提交检视意见（code review），还可以进行投票（vote），团队 committer 据此合入或者驳回你的 MR。

![功能分支工作流2](/images/功能分支工作流2.png)

新功能大量合并到 master 分支后容易造成 master 分支质量不稳定，不稳定会有什么问题？比如线上突然有个 bug 要解决，可能只需要修改一行代码就能解决，但是 master 分支已经合入了大量新特性，测试人员还没来得及测试，那最稳妥的办法就是将代码回退到上一次发版本的时间节点，基于这个节点再修改一行代码，是不是太麻烦了？

为了解决这些问题，Vincent Driessen大佬基于开发实践总结了一套 Git 分支管理的流程和规范，下面详细介绍一下。

## Gitflow 工作流
---

Gitflow 工作流是目前非常成熟的一个方案，它定义了一个围绕项目发布的严格分支模型，通过为代码研发、项目发布以及维护分配独立的分支来让项目的迭代过程更加地顺畅，不同于之前的集中式工作流以及功能分支工作流，Gitflow 工作流常驻的分支有两个：主干分支 master、开发分支 develop。

和功能分支工作流相比，Gitflow工作流没有增加任何新的概念或命令，它给不同的分支指定了特定的角色，定义它们应该如何、什么时候交互。除了功能分支之外，还为准备发布、维护发布、记录发布分别使用了单独的分支。

**Gitflow 常见分支：**

* 开发主分支：master 分支
master 分支的代码是可以直接部署到生成环境的，为了保持稳定性一般不会直接在这个分支上修改代码，都是通过其他分支合并过来的。

* 开发主分支：develop分支
develop 分支是主开发分支，包含所有要发布到下一个release的代码，主要是由feature分支合并过来的。

* 临时分支：feature 分支
feature 分支主要是用来开发一个新特性，一旦开发完成会合入 develop 分支，feature 分支也随即删除掉。

* 临时分支：release 分支
当需要一个发布一个新release版本时，会基于develop分支创建一个release分支，经过测试人员充分测试后再合入 master 分支和 develop 分支。

* 临时分支：hotfix 分支
当在生成环境发现新的Bug时候，如果需要紧急修复，会创建一个hotfix分支， 充分测试后合入master和develop分支，随后删除该分支。

**各分支如何配合工作？**
（1）master/develop分支
原则上master分支上所有的commit 都应该打上Tag，因为一般情况下master不存在 直接commit；
devlop分支 是基于 master分支创建的，与 master 分支一样都是主分支，不会被删除。
develop 从 master 拉出来之后会独立发展，不会与 master 直接产生联系。

![master_develop分支](/images/master_develop分支.png)

（2）feature 分支
通常一个独立的特性都会基于 develop 拉出一个 feature 分支，feature 分支之间没有任何交互，互不影响。feature 分支一旦开发完成后会立马合入 develop 分支（采用 merge request 或者 pull request），feature 分支的生命周期也随之结束。


![feature分支](/images/feature分支.png)

（3）release 分支
通常一个迭代上线会拉一个release 分支，开发人员开发完毕所有的代码都已合入 develop 分支，这时候会基于 develop 分支拉出一个 release 分支，测试人员基于该分支进行测试。

![release分支](/images/release分支.png)

（4）hotfix 分支
hotfix分支基于master分支创建，开发完后需要同时回合到master和develop分支，同时在master上打一个tag。

![hotfix分支](/images/hotfix分支.png)

**分支命名规范**

团队内部可以约定每个分支的命名样式，这里举个例子，大家可以参考：

* feature分支：以feature_开头，如 feature_order
* release分支：以release_开头，如 release_v1.0
* hotfix分支：以hotfix_开头，如hotfix_20210117
* tag标记：如果是release分支合并，则以release_开头，如果是hotfix分支合并，则以hotfix_开头。

## Forking 工作流
---

Forking 工作流是以 Github 为代表的一种代码协作方式，开发者通过克隆（fork）源仓库进行编写代码，一旦完成会发起 pull request，源仓库作者可以选择是否接受该 PR。
下面通过 Github 详细讲解 Forking 工作流模式。
随便找一个Github 开源项目，https://github.com/CoderLeixiaoshuai/java-eight-part
右上角有三个按钮：Watch，Star，Fork
Watch 是关注的意思，一旦你点击了之后该项目有任何改动都会第一时间通知到你；
Star 类似于点赞的意思，多给开源项目点个赞，鼓励一下作者；
Fork 本意是分叉，实际上是克隆的意思，点了之后会将该项目拷贝一份到自己的 github 远程仓库中。

![Forking工作流1](/images/Forking工作流1.png)

**fork 示例**
在本地执行 git clone 命令将代码克隆到本地，一顿修改操作后提交代码并 push到个人远程仓库中，然后在界面上发起 pull request，项目的原作者会看到你提交的 PR，根据提交的质量作者可以选择接受或拒绝。

![fork示例](/images/fork示例.png)

Forking 工作流非常适合于类似 Github 这种开源项目，任何一个开发者都可以通过fork + pull request 向项目中贡献代码。

## 总结
---

文章介绍了四种工作流，分别是集中式工作流，功能分支工作流，Gitflow 工作流，Forking 工作流。
集中式工作流在 SVN 时代比较常见，切到 Git 后不建议再使用这种方式了。
功能分支工作流通常是一个主干 master 分支 + 多个 feature 分支，一般适用于小团队开发。
Gitflow 工作流是在功能分支工作流的基础上进一步演进而来，采用 master + develop 双主分支再加上多个临时功能分支，这是一个非常成熟的代码协作管理的方式，推荐大家使用。
Forking 工作流主要采取 fork + pull request 的模式进行协作，主要用于开源项目。


**最后：**
这四种工作流方式各有特色，开发团队可根据自身的特点去选择，不必严格拘泥于某一种方式，适合自己的才是最优的。

