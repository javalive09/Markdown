# GIT分享

## 常用命令

```text
git clone //克隆仓库

git init  //创建仓库

git add -A  //添加到缓存区

git commit -m "提交代码信息"

git commit --amend // 修改提交信息

git checkout abc //切换分支

git branch a //创建分支

git checkout -b abc //创建分支并切换

git reset --hard HEAD  //还原到HEAD

git diff //对比（工作区和上次提交）的区别

git status //状态

git log  // log

git pull //从远程仓库拉取pull = fetch + merge
```

## 一些使用场景

### 合并和冲突解决

```text
      D---E test  
     /  
A---B---C---F master
```

#### 在master中执行 merge

```text
      D--------E  
     /          \  
A---B---C---F----G   test, master
```

#### 在master中执行 rebase

```text
A---B---D---E---C'---F'   test, master
```

#### 在master中执行 cherry-pick

```text
A---B---C---F---D'----E'  master
```

#### 三方合并\(Three-way merge\)

```text
   |------102
   |
100|------101
```

1. 以父节点做为判断标准
2. 101，102 有不同的文件的改变，留下这些改变。
3. 101，102 有相同的文件的改变，

1）如果改变相同，留下这些改变。

2）如果改变不同，提示冲突让用户去解决。

#### 树冲突

两个分支都修改了同一个文件的名字 全部保留这些文件。

### 回滚

#### 没有push到remote 仓库的回滚

```text
git reset --hard commitid
```

#### 已经push到remote仓库的回滚

```text
git revert commitid
```

#### 定位并回滚到引入bug的版本

```text
git bisect start  
git bisect bad  
git bisect good commtid  
git bisect bad  
...  
git bisect good  
git bisect reset
```

### 当前工作被打断

#### 保存工作进度

```text
git stash  
git stash list  
git stash apply stash@{2}
```

#### 补充提交

```text
git commit --amend
```

### 挽救误操作

分支丢失

```text
git reflog show
```

## .git文件夹结构及git内部原理

### git-stage 暂存区

![](/images/git-stage.png)

### .git 文件夹结构

HEAD config objects logs refs index

## 远程仓库的使用及github

## 使用github搭建免费博客

