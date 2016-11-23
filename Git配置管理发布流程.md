## git 管理模型

分布式但集中化，这是对 git 管理模型最好的阐述。模型图如下图所示：

![](http://ww1.sinaimg.cn/large/65e4f1e6jw1fa0kxolzo2j20da09wjs3.jpg)

上图中 origin 是一个 ” 中心库 ” ，当然这个中心库只是被认为是这样 ( 因为 Git 是分布式的，从技术层面上来说是没有中心库的 ) 。

每个开发者 pull 和 push 到 origin ，但除了中心化的 push-pull 关系外，每个开发者还可以从其他开发者那 pull changes 。比如说，对于一个比较大的新特性，在把代码提交到 origin 之前，很可能会安排 2 个或多个开发者。上图中有几个小团队： Alice 和Bob ， Alice 和 David ， Clair 和 David 。团队成员先把代码 pull 到队长那里，再由队长 pull 到 origin 库。

## 主要分支设计

中心仓库有两个分支： master 和 develop 。

origin 上的 master 分支， Git 用户应该很熟悉，跟 master 并行的有一个 develop 分支

![](http://ww1.sinaimg.cn/large/65e4f1e6jw1fa0kxr8jbnj20720aiq38.jpg)

我们把 origin/master 作为主要分支，源码的 HEAD 总是表示 production-ready( 可随时部署 ) 状态。而 origin/develop 上的代码是为下一次的代码发布准备的。每日构建也是基于此分支。

当 develop 分支达到了一个稳定状态并准备发布时，所有的改变都要合并到 master 分支，并标上版本号。这样每次与 master 合并都会会有新的部署发布。如下是相关命令：

`$git checkout –b develop master`

在 develop 分支上进行开发工作。

`$git checkout master`

`$git merge –no-ff develop`

## 支持分支设计

我们的开发模型使用了一些支持分支放在 master 和 develop 分支的旁边，方便开发小组之间的并行开发。不像主要分支，这些分支是有时间限制的，因为他们最终都会被移除。

我们会使用到的不同的分支有： Feature branches 、 Release branches 、 Hotfix branches 。

每个分支都有各自的作用，并且有严格的规定，如：只能从哪个分支上去新开分支，只能合并到那个分支。

### Feature branches

规定如下：

继承分支: **develop**

合并分支: **develop**

命名规范：除了 master,develop,release- ,hotfix-

Feature branches 是用来开发新特性的 ( 短期，远期都可以 ) 。当开始开发新特性时，很可能不知道这个特性会出现在哪个目标版本。一旦开发完成就可以合并到 develop，当然如果开发失败，就可以抛弃。

#### 创建及完成 Feature branch

根据 Feature branches 的规定，创建命令如下：

`$ git checkout -b myfeature develop`

新特性完成时，可以合并到 develop

`$ git checkout develop`

`$ git merge –no-ff myfeature`

`$ git branch -d myfeature`

`$ git push origin develop`

—no-ff ( 注： no fast foward) 标签，使得每一次的合并都创建一个新的 commit 记录。即使这个 commit 只是 fast-foward ，这样可以避免丢失信息

![](http://ww2.sinaimg.cn/large/65e4f1e6jw1fa0kxxjoq5j20cv0biaae.jpg)

### Release branch

规定如下：

继承分支: **develop**

合并分支: **develop** 和 **master**

命名规范： release-*

Release branch 是为新的 production release 准备的，可以有一些小的 bug ，并为发布准备一些元数据 ( 版本号，构建日期等等 ) 。把所有的这些工作都放到 Release branch ， develop branch 就能更清晰地知道下一个版本要开发哪些特性。

从 develop 分支合并到 release 分支的关键因素是: develop 分支达到了 release 分支所要求的状态。至少所有针对该 release 的特性要被合并。至于那些将来会有的特性可以先放一放。然后就是为接下来即将要发布的版本分配一个版本号。

#### 创建 Release branch

Release branch 是通过 develop 分支而创建。举个例子，假如 1.1.5 是当前的 production release ，然后会有一个比较大的版本发布。 develop 的状态已经可以发布版本了，经过商榷后，决定发布为 1.2 版本，所以我们创建一个 release 分支，并给这个分支一个新的版本号

`$ git checkout -b release-1.2 develop`

这个新分支可能会存在一定的时间，直到可以被合并到 production branch 。这段时间内， bug 修补可以在这个分支上进行 ( 而不是 develop 分支 ) 。添加新特性 ( 尤其比较大的 ) 是不允许的。最后还是要被合并到 develop ，然后继续在 develop 分支上开发，直到下一个版本。

#### 完成 release branch

当 release branch 已经准备就绪，需要做几件事。首先， release 分支被合并到 master 分支上 ( 每一个提交到 master 上的 commit 都是一个新版本，切记 ) 。然后 master 上的 commit 都要添加 tag ，方便将来查看和回滚。最后 release 上所做的修改必须合并到 develop 分支上，保证 bug 已被修补。

前两个步骤：

`$ git checkout master`

`$ git merge –no-ff release-1.2`

`$ git tag -a 1.2`

为了把 release 上的改变保存到 develop ，我们需要合并到 develop

`$ git checkout develop`

`$ git merge –no-ff release-1.2`

这个步骤可能会导致冲突，如果这样的话，解决冲突，然后再提交。

现在一切都完成了，可以把 release branch 干掉了。

`$ git branch -d release-1.2`

### Hotfix branch

规定如下：

继承分支: **master**

合并分支: **develop** 和 **master**

命名规范： hotfix-*

Hotfix branch 和 Release branch 有几分相似，都是为了新的 production release 而准备的。比如运行过程中发现了 bug ，就必须快速解决，这时就可以创建一个 Hotfix branch ，解决完后合并到 master 分支上。好处是开发人员可以继续工作，有专人来负责搞定这个 bug 。

#### 创建 Hotfix branch

Hotfix 是从 master 分支上创建的。假如当前运行版本是 1.2 ，然后发现有 bug ，但是 develop 还在开发中，不太稳定，这时就可以新开一个 Hotfix branch ，然后开始解决问题。

`$ git checkout -b hotfix-1.2.1 master`

解决问题，一次或几次 commit

`$ git commit -m “Fixed severe production problem”`

#### 完成 Hotfix branch

当结束时， bugfix 要被合并到 master ，同时也要合并到 develop ，保证下个版本发布时该 bug 已被修复。这跟 release branch 完成时一样。

首先更新 master 和 tag release

`$ git checkout master`

`$ git merge –no-ff hotfix-1.2.1`

`$ git tag -a 1.2.1`

接下来与 develop 合并

`$ git checkout develop`

`$ git merge –no-ff hotfix-1.2.1`

有一个例外，就是当存在一个 release branch 存在时， bugfix 要被合并到 release 而不是 develop ，因为 release 最终会被合并到 develop 。

最后移除 branch

`$ git branch -d hotfix-1.2.1`

