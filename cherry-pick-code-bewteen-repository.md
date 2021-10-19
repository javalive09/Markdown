# **如何跨仓库cherry-pick代码**
# 背景
现需要将wutong WTOS02_android9.0_qc6155仓库的代码移植到OpenOS仓库中，相同库中不同分支之间移植代码是比较容易的，直接cherry-pick即可，但是跨库如何cherry-pick呢，方法如下。
# 一，添加远程仓库
## 1.查看现有的remote仓库
```
peter@peter$ git remote -v

origin    gerritserver:/OpenOS/ww (fetch)
origin    gerritserver:/OpenOS/ww (push)
```
## 2.添加包含cherry-pick代码的仓库
```
peter@peter: git remote add peter123 gerritserver:/android9.0/peter123
```
## 3.检查是否添加成功
```
peter@peter: git remote -v

android9.0    gerritserver:/android9.0/peter123 (fetch)
android9.0    gerritserver:/android9.0/peter123 (push)
origin    gerritserver:/OpenOS/ww (fetch)
origin    gerritserver:/OpenOS/ww (push)
```

# 二，拉取代码
```
peter@peter: git fetch android9.0

remote: Counting objects: 13750, done
remote: Finding sources: 100% (419/419)
remote: Total 419 (delta 182), reused 408 (delta 182)
Receiving objects: 100% (419/419), 5.17 MiB | 10.67 MiB/s, done.
Resolving deltas: 100% (182/182), completed with 72 local objects.
From gerritserver:/WTOS02_android9.0_qc6155/wutong
 * [new branch]        code_main_dev               -> android9.0
 ```
 # 三，跨库cherry-pick代码
 ```
peter@peter:git cherry-pick 7a252c2bce84d37a7d26308adb9f45875263ed5b
[code_main_dev 6fa4ddee] IssueID=00000000 Description=init commit for android_media_AudioSystem.cpp
 Author: ronny <p_ronnyrli@xxx.com>
 Date: Mon Dec 21 11:27:22 2020 +0800
 1 file changed, 2129 insertions(+)
 create mode 100644 aosp/frameworks/base/core/jni/android_media_AudioSystem.cpp
 ```
# 四，同步cherry-pick的代码到remote仓库并添加reviewer
```
peter@peter: git push origin HEAD:refs/for/code_main_dev%r=p_123,r=p_456
```
# 五，清理添加的库，保持一个干净的原有仓库
```
git remote remove peter123
```