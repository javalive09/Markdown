# GIT命令

## 配置

```text
git --version  
git config --list
git config --global user.name "first name"
git config --global user.email "javalive09@163.com"  
git config color.ui true 
git config -e //demo/.git/config
git config -e --global // /home/peter/.gitconfig
git config -e --system  // /etc/gitconfig
ssh-keygen -t rsa -C "javalive09@163.com"
clip < ~/.ssh/id_rsa.pub
ssh -T git@github.com
```

配置多个仓库

```text
git touch ~/.ssh/config
```

```text
#GitHub  
Host github.com  
  User git  
  HostName github.com  
  IdentityFile ~/.ssh/id_rsa  

#Maxthon   
Host mgit    
  User git      
  HostName mgit.maxthon.net  
  Port 22      
  IdentityFile ~/.ssh/id_rsa  

#Bitbucket  
Host bitbucket.org  
 User git  
 HostName bitbucket.org  
 IdentityFile ~/.ssh/id_rsa
```

保存密码

```text
git config --global credential.helper store
```

### 设置vim为编辑器

```text
git config --global core.editor "vim"
```

## 创建和提交

```text
git init 
git add -A  //添加到缓存区
git commit -m "提交代码信息"  // 提交代码
git commit --amend // 修改提交信息
```

## 远程仓库

```text
git remote -v
git remote add origin https://github.com/javalive09/CodeBag.git //添加远程仓库
git remote set-url origin https://github.com/javalive09/CodeBag.git //修改远程仓库url
git push origin --delete <branchName>  //删除远程分支
git push origin master //推送到远程仓库
git clone https://github.com/javalive09/CodeBag.git // 从远程仓库clone
git pull //从远程仓库拉取pull = fetch + merge
```

## 切换和还原

reset本质：将当前分支HEAD指针指向任意提交点 checkout本质：切换分支

```text
git checkout develop -f //强制切换分支 （放弃本地修改切换)
git checkout HEAD /Users/peter/git/xui/XuiPSdk/libs/armeabi/libgetuiext.so  //还原单个文件到HEAD
git checkout abbdc /Users/peter/git/xui/XuiPSdk/libs/armeabi/libgetuiext.so  //还原单个文件到abbdc
git checkout <commit> <filepath>  //还原单个文件的某个版本
git checkout 45b92e3 /Users/peter/git/xui/CarClient/assets/video/video.mp4  
git reset --hard master  //还原整个项目到master
git reset --hard HEAD  //还原到HEAD

git reset --soft HEAD~ //还原到HEAD上次提交，缓存区和工作目录都不会被改变
git reset --mixed HEAD~ //还原到HEAD上次提交，缓冲区也还原，工作目录不变
git reset –-hard HEAD^  //还原到HEAD上次提交，缓冲区也还原，工作目录也还原
```

### 查看仓库状态

```text
git status
```

### 查看log

```text
git log  
git log --graph 
git log --oneline
git log --oneline --graph
```

## tag 标签

```text
git tag   
git tag -l  
git show 1.0  
git tag -a v1.0 -m "新加tag"  
git tag -d v1.0  //删除本地标签
git push origin --tags  
git push origin :refs/tags/<tag name>  //删除远程tag
```

### 重命名tag

```text
git tag new old
git tag -d old
git push origin :refs/tags/old
git push --tags
```

## 分支

```text
git branch  
git branch abc  
git checkout abc  
git push origin abc  
git push origin :abc //删除远程分支
git push origin --delete abc //删除远程分支
git checkout -b abc 
git branch -a   //列出所有分支（本地＋远程）
git checkout -b abc  //创建并且切换分支
git branch -d <branch name> //删除本地分支
git branch -m release_v1.24 release_v1.0.0 //修改分支名字
git remote prune origin //清除本地不存在的远程分支
git fetch -p //清理本地无效的远程分支 -p = prune
```

## stash

stash 只和当前branch 绑定 所以如果你切换分支后不会再有之前分支保存的stash信息

### 保存

```text
git stash save "peter"
```

### 使用

```text
git stash 
git stash list
git stash apply stash@{2}//使用
git stash pop stash@{2}//弹出
```

### 清除

```text
git stash clear
```

## 删除

```text
git rm --cache path   //忽略已经提交的文件
git rm path  //删除版本库中的文件
git mv welcome.txt README  //修改版本库中的文件名
```

### 出现 modified: .idea/misc.xml 如何解决

修改ignore后输出如下命令：

```text
git rm -rf .idea
git commit -m "delete .idea"
git push
```

## 恢复

```text
git reflog show // 或者简写 git reflog
git branch recover_branch commitid
```

## 对比文件

```text
git diff //对比（工作区和上次提交）的区别
git diff filename  //对比（工作区某个文件和上次提交的这个文件）的区别
git diff --cached  //对比（缓存区和上次提交）的区别
git diff –cached filename //对比（缓存区某个文件和上次提交的这个文件）的区别
git diff ffd98b291e0caa6c33575c1ef465eae661ce40c9 b8e7b00c02b95b320f14b625663fdecf2d63e74c   //对比（某两个版本）的区别
git diff ffd98b291e0caa6c33575c1ef465eae661ce40c9:filename b8e7b00c02b95b320f14b625663fdecf2d63e74c:filename  //对比（某两个版本文件）的区别
```

## 归档

```text
git archive -o latest.zip HEAD  //基于最新提交建立归档文件latest.zip
git archive -o partial.tar HEAD src doc  //只将目录src和doc建立到归档partial.tar中
git archive --format=tar --prefix=1.0/ v1.0 | gzip > foo-1.0.tar.gz  //基于里程碑v1.0建立归档，并且为归档中文件添加目录前缀1.0
```

## patch

### 生成path

```text
git format-patch -<n> <SHA1>
git format-patch -1 <sha>
```

### 应用patch

```text
git am xxx.patch
```

### 错误处理

```text
git apply --ignore-space-change --ignore-whitespace mychanges.patch
```

```text
git apply --reject --whitespace=fix mychanges.patch
```

## 场景

### 1.合并多次提交为一次提交

```text
git reset —soft <commitId>//要合并的起始提交点id
git commit -m "xxx"
```

### 2.剔除某次提交

#### 没有push到服务器

**方法1：cherry-pick**

```text
git checkout <commitId-last>//要剔除提交点的上一个提交点id
git cherry-pick <commitId-next>//要剔除提交点的下一个提交点id
... //cherry-pick 其他提交点
git checkout master
git reset --hard <commitId-end>//最后一次cherry-pick的提交点
```

**方法2：rebase --onto**

```text
git rebase --onto master issue1 sissue2 //在issue1分支合并前, 合并issue2分支
git rebase --onto <newbase>要剔除提交点的上一个提交点id <since>要剔除提交点的id <branch>
```

**方法3：rebase -i //推荐 interactive**

```text
git rebase -i <commitId-last> 
修改vim中弹出的信息，把要删除的 commitId 的pick 修改成 d 保存后退出
```

#### push到服务器

```text
git revert <commitId> // revert commitId 这次提交
```

### 3.修改提交信息

#### 修改最近一条消息

**没有push到服务器**

```text
git commit --amend
```

**push到服务器**

```text
git commit --amend
git push -f origin master
```

#### 修改以前的消息

```text
git rebase -i <commitId-last>
```

修改vim中弹出的信息，把pick 修改成 r 保存后退出

```text
git push -f origin master
```

## submodule

### add

```text
git submodule add git@git.raventech.cn:H1S/common-proto.git /support/src/main/common-proto

git status//检测添加成功

    On branch master
    Changes to be committed:    
        new file:   .gitmodules
        new file:   common-proto
    表示添加成功
```

### modify

在submodule目录下可以进行所有的git子项目操作

```text
git status
git add
git commit -m "abc"
git push
```

### update

```text
git submodule foreach git pull//项目根目录下
或
git pull//子项目目录下
```

### clone

```text
git clone git@git.raventech.cn:H1S/common-proto.git --recursive
或
git submodule init
git submodule update
```

### delete

```text
1.
git rm --cached common-proto
rm -rf common-proto
rm .gitmodules

2.
vim .git/config //update config

3.
git commit -a -m 'remove common-proto submodule'
git push
```

### update url

```text
edit .gitmodules
git submodule sync
```

## subtree

假设，你要在各个项目里的/support/src/main/common-proto这个目录对 git@git.raventech.cn:H1S/common-proto.git这个项目做Subtree

### add remote

```text
git remote add proto git@git.raventech.cn:H1S/common-proto.git  
```

### add subtree

```text
cd 项目的路径
git subtree add --prefix=用来放proto项目的相对路径 proto项目git地址 xxx分支
git subtree add --prefix=/support/src/main/common-proto proto master
```

### update

```text
git subtree pull --prefix=/support/src/main/common-proto proto master
```

### push subtree commit

```text
git subtree push --prefix=/support/src/main/common-proto proto hotfix/proto
在common-proto项目中合并hotfix/proto即可
```

## [速查表](https://git-scm.com/docs)

## 本地多分支共享多平台代码方式

### 拉取一个平台的代码到本地

```text
git clone git@github.com:javalive09/CodeBag.git
```

### add其他仓库

```
git remote add other https://xxxxx
```

### fetch remote 仓库

```text
git remote -v git fetch other
```

### 切换到仓库分支

```
git remote -v git checkout remotes/other/master
git checkout -b other@master
```

### push新仓库分支

```text
git push other refs/heads/other@master:refs/heads/master 
// src:dest 从本地other@master push 到 远程 master 

git push other HEAD:refs/heads/master 
// src:dest 从本地HEAD push 到 远程 master
```

### pull新仓库分支

```
git pull other refs/heads/master:refs/heads/other@master 
//src:dest 从远程master pull 到本地 other@master
```

### pull 更方便的办法（设置upstream）

```text
git branch -vv//查看本地分支和远程分支关联关系

git branch -u other/master 
//设置分支的upstream == git branch --set-upstream-to=other/master other@master

git branch -vv //检查是否关联上了
```

## The Refspec

其中 spec 是 specification 的简写。最早把 refspec 翻译为“引用表达式”，现已经修改为“引用规则”，相应的 pathspec 翻译为“路径规则”。 [官网](https://git-scm.com/book/zh/v1/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-The-Refspec) Refspec 的格式是一个可选的 + 号，接着是 : 的格式，这里  是远端上的引用格式，  是将要记录在本地的引用格式。可选的 + 号告诉 Git 在即使不能快速演进的情况下，也去强制更新它。

### fetch其他库的分支

如你有一个QA组，他们推送一系列分支，你想每次获取 master 分支和QA组的所有分支，你可以在.git/config中使用这样的配置：

```text
[remote "origin"] url = git@github.com:schacon/simplegit-progit.git 
fetch = +refs/heads/master:refs/remotes/origin/master 
fetch = +refs/heads/qa/:refs/remotes/origin/qa/
```

### push到其他库分支

如果QA组成员想把他们的 master 分支推送到远程的 qa/master 分支上，可以这样运行：

```text
git push origin master:refs/heads/qa/master
```

如果他们想让 Git 每次运行 git push origin 时都这样自动推送，在.git/config中自定义push

```text
[remote "origin"] url = git@github.com:schacon/simplegit-progit.git 
fetch = +refs/heads/:refs/remotes/origin/ 
push = refs/heads/master:refs/heads/qa/master
```

### 删除分支

因为 refspec 的格式是 :, 通过把  部分留空的方式，这个意思是是把远程的 topic 分支变成空，也就是删除它。

```text
git push origin :topic
```

### refs/for/ 和refs/heads/

[详细解释](https://blog.csdn.net/caomiao2006/article/details/44243309) 

1. 这个不是git的规则，而是gerrit的规则， 

2. Branches, remote-tracking branches, and tags等等都是对commite的引用（reference）,引用都以 “refs/……”表示. 比如remote branch: origin/git\_int\(=refs/remotes/origin/git\_int\)， local tag: v2.0\(=refs/tags/v2.0\)， local branch: git\_int\(=refs/heads/git\_int\)… 

3. 简单点说，就是refs/for/mybranch需要经过code review之后才可以提交；refs/heads/mybranch不需要code review。

## gitk\(The git repository browser\)

[文档](https://docs.oracle.com/cd/E56344_01/html/E54075/gitk-1.html#scrolltoc)

gitk \[...\] \[\] \[--\] \[...\] //命令格式

gitk //显示当前分支 gitk --all //显示当前目录下 全部分支 --all 为 option gitk ./ //显示当前目录下 当前分支

## 跨仓库cherry-pick

```text
//添加reomote仓库
git remote add WTOS02_android9.0_qc6155 gerritserver:/WTOS02_android9.0_qc6155/wutong

//拉取代码
git fetch WTOS02_android9.0_qc6155

//cherry-pick
git cherry-pick 7a252c2bce84d37a7d26308adb9f45875263ed5b

//清理remote仓库
git remote remove WTOS02_android9.0_qc6155
```

