---
layout: post
title:  git相关知识总结
date:   2015-11-23 00:15:00
comment: true
category: "开发工具"
---

本文并不会详细讲git怎么用, 这只是我对git相关知识的总结, 供自己在记忆卡壳时查找, 如果你也记心不好, 可以整个类似的. git详细教程网上很多, 我看的是廖雪峰版, 简单明了非常推荐. 《Pro Git》讲的更详细, 薄薄一本, 也不会让人抓狂.

---

##### 安装后初次使用

    git config --global user.name "Your Name"
    git config --global user.email "email@example.com"

##### 创建版本库
* git init: 把这个目录变成Git可以管理的仓库.
* git add [file]:  添加一个或多个文件.
* git commit -m "": 提交文件, 可一次提交多个.

##### 查看状态和版本回退
* git status: 查看仓库当前的状态.
* git diff: 查看修改内容.
* git log: 显示从最近到最远的提交日志.
	- \--pretty=oneline: 每个提交只显示一行.
	- commit id(版本号)是一个SHA1计算出来的一个非常大的数字, 用十六进制表示.
* git reflog: 查看命令历史.
* git reset \--hard HEAD^: 回到上一个版本. HEAD^^, HEAD~100: 前n个版本.
* git reset \--hard 3628164: 回到某个版本

	> Git内部有个指向当前版本的HEAD指针, 当你回退版本时, Git仅仅改变HEAD指向.HEAD严格来说不是指向提交, 而是指向master, master才是指向提交的, 所以, HEAD指向的就是当前分支.

##### 撤销、删除
* git checkout \-- file: 丢弃工作区的修改. **\--最好加上, 没有\--, 有可能与切换分支命令冲突.**
* git reset HEAD file: 把暂存区的修改撤销掉(unstage), 重新放回工作区.
* git rm: 与git add相对应, 用于提交一个删除操作, 随后进行git commit.
* git checkout 无论工作区是修改还是删除, 都可以**"一键还原"**.
* git rm \--cached file: 删除已提交的文件.

##### 远程仓库
> Git支持多种协议, 包括https, 但通过ssh支持的原生git协议速度最快, 使用https除了速度慢以外, 还有个最大的麻烦是每次推送都必须输入口令.

* 生成ssh-key: ssh-keygen -t rsa -C "youremail@example.com"
* 先有本地库, 后有远程库的时候, 如何关联远程库:
	1. git remote add origin git@server-name:path/repo-name.git: 关联一个远程库, 远程库的名字就是origin, 这是Git默认的叫法.
	2. git push -u origin master: **第一次推送**master分支的所有内容.之后可用git push origin master推送最新修改.

* 从远程仓库克隆: git clone git@server-name:path/repo-name.git
* 建立远程仓库, 让别人clone: git clone yourip:/path/.git
* 修改远程仓库地址: git remote set-url
* 推送到远端: git push origin develop:develop (本地: 远端)
* 拉取到本地: git pull origin develop:develop (远端: 本地)

##### 分支管理
> 分支作用: 不完整的代码库会导致别人不能干活了.如果等代码全部写完再一次提交, 又存在丢失每天进度的巨大风险. Git鼓励大量使用分支.

* git branch: 查看分支.
* git branch [name]: 创建分支.
* git checkout [name]: 切换分支
* git checkout -b [name]: 创建+切换分支. 相当于git branch [name]; git checkout [name].
* git merge [name]: 将[name]合并到master分支上, 加上\--no-ff参数就可以用普通模式合并, 合并后的历史有分支, 能看出来曾经做过合并, 而fast forward合并就看不出来曾经做过合并.
* git branch -d [name]: 删除分支.
* git log \--graph \--pretty=oneline \--abbrev-commit: 查看图形化的提交日志.
* git branch -D <name>: 丢弃一个没有被合并过的分支.

---

### 分支策略
> 在实际开发中, 我们应该按照几个基本原则进行分支管理:
>
> 首先, master分支应该是非常稳定的, 也就是仅用来发布新版本, 平时不能在上面干活. 那在哪干活呢? 干活都在dev分支上, 也就是说, dev分支是不稳定的, 到某个时候, 比如1.0版本发布时, 再把dev分支合并到master上, 在master分支发布1.0版本.
你和你的小伙伴们每个人都在dev分支上干活, 每个人都有自己的分支, 时不时地往dev分支上合并就可以了. 所以, 团队合作的分支看起来就像这样:
![img]({{ site.baseurl }}/images/git-branch-in-dev.png "git branch in dev")

##### bug分支
> 修复bug时, 我们会通过创建新的bug分支进行修复, 然后合并, 最后删除. 当手头工作没有完成时, 先把工作现场git stash一下, 然后去修复bug, 修复后, 再git stash pop, 回到工作现场.

* git stash list: 查看stash内容

多人协作的工作模式

- 首先, 可以试图用git push origin branch-name推送自己的修改.
- 如果推送失败, 则因为远程分支比你的本地更新, 需要先用git pull试图合并.
- 如果合并有冲突, 则解决冲突, 并在本地提交.
- 没有冲突或者解决掉冲突后, 再用git push origin branch-name推送就能成功！
- git checkout -b branch-name origin/branch-name: 在本地创建和远程分支对应的分支.

> 如果git pull提示"no tracking informatio", 则说明本地分支和远程分支的链接关系没有创建, 用命令 git branch \--set-upstream branch-name origin/branch-name.

##### 标签管理
> 标签是指向某个commit的指针(跟分支很像对不对? 但是分支可以移动, 标签不能移动).

* git tag <name>: 打一个标签在最新提交的commit上.
* git tag: 查看所有标签.
* git show <tagname>: 查看标签信息.
* git tag -a <name> -m "": 创建带有说明的标签, 用-a指定标签名, -m指定说明文字.
* git tag -d <name>: 删除标签.
* git push origin <tagname>: 推送某个标签到远程.
* git push origin \--tags: 一次推送全部未推送的标签.
* git tag -d <tagname>: 删除一个本地标签.

> 删除远程标签比较麻烦, 先删除本地标签, 再执行git push origin :refs/tags/<tagname>

##### 其他事项
* .gitingore: 忽略某些文件. **不需要从头写.gitignore文件**, GitHub已经有[repo](https://github.com/github/gitignore){:target="_blank"} 为我们准备了各种配置文件.
* git configure \--global alias.st status: 配置别名.

> 每个仓库的Git配置文件都放在.git/config文件中, 别名就在[alias]后面, 要删除别名, 删去对应行即可.当前用户的Git配置文件放在用户主目录下的一个隐藏文件.gitconfig中.

> 分布式版本控制系统通常也有一台充当"中央服务器”的电脑, 但这个服务器的作用仅仅是用来方便"交换”大家的修改, 没有它大家也一样干活, 只是交换修改不方便而已.

另外，[git-flow](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html#release){:target="_blank"} 能更方便的进行分支模式开发, 有兴趣的同学可以了解下. 有的开发团队可能会用到.

### 参考文献
1. 廖雪峰, [Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000){:target="_blank"} .
2. Scott Chacon, [Pro Git](http://book.douban.com/subject/3420144/){:target="_blank"} .
