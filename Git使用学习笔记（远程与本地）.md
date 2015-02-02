
今天大体写完了Anti-Spam的代码。我本想搞的高大上一点，就开了一个分支(branch)，然后就各种捉鸡了，以前会的那一点点命令似乎都不会用了，比如推送(push)的时候需要用`git push origin database:database`,但是不知道为什么要这么写。合并(merge)的时候就完全不会了，以前看了[git-简易教程](http://rogerdudler.github.io/git-guide/index.zh.html)和这个[try github](https://try.github.io)小游戏以及[Learn Git Branching](http://pcottle.github.io/learnGitBranching/?demo)就感觉自己似乎很懂了，其实看到黑压压的命令行还是束手无策。

首先来一张我昨天不明白分支/合并瞎倒腾的截图

![my gitk screenshot](http://zchang.me/wp-content/uploads/2015/02/git-tree-of-Anti-Spam.png)

本来想的很简单，来一个分支database，写好之后合并到主分支(master)就行了，原来远不只这么简单。原来仓库(repository)节点是分远程(remote)和本地(local)的，而我一开始在远程又建了分支，却又把本地分支提交到远程上，之后就是各种不知所云了……

今天看了阮一峰的[Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)，在加上之前玩小游戏对本地分支的理解（捂脸..），我简单的总结一下github的用法，对于我自己已经熟悉的地方(clone,add,remote等)，这里就不再解释了。

首先，git的操作历史是一个有向无环图，比如，每一次提交(commit)都是一个节点，而所谓的分支比如我的截图里的master、database其实都是一个指针，指向某个节点。创建分支、合并甚至提交的过程其实都是创建节点和移动指针的过程。


![git platform](http://image.beekka.com/blog/2014/bg2014061202.jpg)
（从阮一峰的博客盗图）

HEAD指针相当于一个指针指向你的当前位置，一个位置代表一种状态，即仓库中所有文件的内容，每次使用签出(checkout)命令来切换这个指针所指的位置。

从这个图上可以看出来，本地的操作有`checkout`,`commit`,`add`，此外还有`merge`,`rebase`,`reset`等等，而对远程的操作只有`fetch`,`clone`,`pull`,`push`，我们就是通过这些命令来新建/移动指针和添加新边的。

我在开源中国(OSC)上开了一个仓库名叫git-learning专门用来试验各种命令的使用，下面是我实验的过程

首先，我克隆(clone)了到本地仓库，没有新的分支，只有master 和 origin/master，他们之间相互关联：

	$ git branch -a
	
![\[图1\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE1.png)

接着，我新建一个本地分支`dev`

	$ git checkout -b dev
这句话相当与

	$ git branch dev
	$ git checkout dev
	
![\[图2\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE2.png)

然后，我做一些修改

	$ vim test.md
	(添加了一句话)
	$ git add .
	$ git commit -m "add a sentence"
dev就往前移动了一格

![\[图3\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE3.png)

现在，我用本地的dev 分支在远端创建一个分支
	
	$ git push origin dev
这句话相当与
	
	$ git push origin dev:dev
	
这个时候，远程会用你的dev新建一个origin/dev ，并默认关联起来，当然，你在远端新建的分支不一定也要叫dev，你可以起名为develop

	$ git push origin dev:develop
	
![\[图4\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE4.png)

现在，两个指针dev和origin/develop指向同一个位置,master和origin/master没有动。

![\[图5\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE5.png)

然后，开始我的工作，我的所有工作都是在dev上的，假装提交了很多次，每次删一行句子

现在dev指针遥遥领先了

![\[图6\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE6.png)

我觉得差不多了，可以把我的修改上传到远程了

	$ git push origin dev
	
由于疏忽大意，我忘记了在远端的指针名不叫dev而是develop，而我这条命令会在远端创建一个新的指针。不过没有关系，我可以删除掉刚才错误创建的分支

	$ git push origin :dev

这个语句把一般空的分支传给了远程的dev，等于执行了删除语句

	$ git push origin --delete dev
	
![\[图7\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE7.png)

现在，我可以重新执行推送(push)了

	$ git push origin dev:develop

![\[图8\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE8.png)

这样origin/develop就跑上去了

在之后的很多天内，我都一相同的方式在origin/develop上开发，指导有一天，我觉得差不多了，可以把它合并到主线上了


为了确保没有冲突，我先用拉取(pull)把origin/develop更新到dev上来。

	$ git pull origin develop:dev
	
这个":"前后的顺序是与推送是相反的，分别代表远程分支和本地分支.现在。

没有更新，好的，把最新状态的dev并到master去

	$ git checkout master
	$ git merge dev
	
master直接跑了上去。

![\[图9\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE9.png)

因为这里使用了叫做"fast-forward"的模式来合并，直接将master指针移到dev的位置，所以merge的过程似乎悄无声息的发生了。
然后，再通过master把origin/master也拉上来。

	$git push origin master
	
![\[图10\]](http://zchang.me/wp-content/uploads/2015/02/%E5%9B%BE10.png)

这样就完成了把 origin/dev 合并到 origin/master的过程。
现在四个指针都指向一处了。


其实总结起来，还是上面的话，在本地需要的是用`commit/merge/branch/checkout/reset/rebase`这样的命令来操作,在远端用`clone/fetch/pull/push`这样的命令，我并没有介绍每个命令是干什么用的，因为我这一篇博客的主题是远程与本地控制，命令的作用可以玩玩小游戏什么的学习到。还有关于冲突解决的问题这里也没有提到，因为没有多人协作过，所以这样的问题暂时还没有遇到，不太熟悉所以先不写了。

我有一次不知为何无法使用master来推送了，必须加上master:master，也就是是origin/master 没有和 master关联了，可以使用以下代码设置关联:

`git branch --set-upstream master origin/master`

还有不知道为什么之前在github的时候使用`$ git push origin dev`来创建远程分支没有成功，然后就一直磕在这里：(


> Written with [StackEdit](https://stackedit.io/).