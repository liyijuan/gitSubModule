

# git submodule

目前主要先记录以下四种情况下 git submodule 使用及注意事项

1.  创建一个主仓库，然后添加子仓库
2.  从已有仓库拉取代码，并拉取子仓库代码
3.  本地仓库和远程仓库子模块不一致，远程仓库增加新的子模块时
4.  本地仓库的子模块和远程仓库的子模块同时修改相同文件存在冲突时



## 1. 初始化主仓库添加子仓库



### 1.1 设置主仓库

创建主仓库可以选择两种方案：

第一种：本地选择某个文件夹执行git init，然后使用git remote add origin url关联远程仓库，然后推送本地仓库内容到远程仓库

第二种：远程创建一个仓库，然后执行git clone url，创建远程仓库对应的本地仓库

补充第一种方案用到的git指令：

```bash
# 1. 初始化本地仓库
git init
# 2. 设置远程仓库
git remote add origin https://github.com/liyijuan/gitSubModule.git
# 3. 查看当前分支
git branch
# 推送本地仓库到远程
git push -u origin master
```

-u 或 --set-upstream：建立本分支和远程分支之间的追踪关系，用于第一次推送本地分支到远程仓库对应分支

origin远程仓库的别名：后面的url为仓库的具体地址，创建别名后可以使用origin这个别名与后面的url指向的仓库进行交互。比如推送git push origin或拉取git pull origin等操作



### 1.2 添加子仓库

执行git submodule add

```bash
git submodule add https://github.com/jkazama/ddd-java.git
```

执行该指令后，会在.git同级目录下生成 ddd-java, .gitmodules 两个文件，其中ddd-java是拉取的子模块的代码，是一个完整的git项目文件目录，而.gitmodules中的内容包括：按照如下格式，记录子模块的元数据。<br />注意区别git submodule init /update，这两个项目用于从远程父仓库获取子仓库。现在是从本地构建推到远程的过程。

.gitmodules文件内容：

```bash
[submodule "ddd-java"]
	path = ddd-java
	url = https://github.com/liyijuan/ddd-java.git
```

path：子模块在主项目中的相对路径，即子模块在工作目录中的存放位置<br />url：子模块的远程仓库地址，git会根据这个URL克隆子模块到指定路径



### 1.3 更改子仓库URL

如果上述.gitmodules中子仓库的URL是直接在GitHub上找的项目地址，而后期需要修改代码。就需要将地址改为fork之后的自己远程仓库的地址，否则可能无法提交。基于当前已经设置错误的情况，比如url地址为原作者的GitHub项目地址。修正方案如下：

1.  **删除子模块的本地文件夹**: 手动删除子模块的目录。比如，如果子模块名为`ddd-java`，在命令行中执行：

```bash
rm -rf ddd-java
```

2.  **从Git中移除子模块的记录**: 从Git的配置中移除子模块的引用和跟踪信息。执行以下命令：

```bash
git config --remove-section submodule.ddd-java
```

这会从`.git/config`文件中移除子模块`ddd-java`的相关配置。

3.  **清理Git的索引**: 如果存在关于子模块的残留索引项，可以通过以下命令清理：

```bash
git rm --cached ddd-java
```

注意，如果直接移除子模块目录后这个命令报错，可能是因为索引中没有直接以子模块名称为路径的条目。此时，如果确认子模块相关的所有文件都已经删除，可以忽略此步骤。

4.  **提交更改**: 提交上述变更到本地仓库，以永久移除子模块的记录：

```bash
git commit -m "Remove submodule ddd-java"
```

5.  **手动编辑或删除**`**.gitmodules**`**文件**: 使用文本编辑器直接打开并编辑或删除`.gitmodules`文件，移除与该子模块相关的条目。确保保存更改后，执行：

```bash
git add .gitmodules
git commit -m "Remove submodule entry for ddd-java in .gitmodules"
```

6.  **重新添加子模块**: 最后，可以重新添加子模块，就像第一次添加一样：

```bash
git submodule add https://github.com/liyijuan/ddd-java.git
git push
```



## 2. 初次克隆主仓库并拉取子仓库

拉取远程仓库代码

```bash
git clone https://github.com/liyijuan/gitSubModule.git
```

执行这一步后，拉取到本地仓库的内容如下：其中，ddd-java中的内容目前为空![image.png](https://cdn.nlark.com/yuque/0/2024/png/39158306/1717902917843-29a579f4-5a92-42ff-98a8-885cf62c79f1.png#averageHue=%23272625&clientId=ud718a8e0-cd92-4&from=paste&height=146&id=ub7acce1b&originHeight=131&originWidth=622&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=10136&status=done&style=none&taskId=u3b8f30b0-48ac-43fe-8c61-8629d77dc8e&title=&width=691.1111294193037)



### 2.1 git submodule init

git submodule init 用于初始化Git仓库的子模块<br />执行这一步后，上述ddd-java文件夹还是处于空状态，唯一变化的是同级的.git/config文件<br />原本的config目录：

```shell
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[remote "origin"]
	url = https://github.com/liyijuan/gitSubModule.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```

执行完git submodule init之后，增加了子模块的元数据信息

```shell
[submodule "ddd-java"]
	active = true
	url = https://github.com/jkazama/ddd-java.git
```



### 2.2 git submodule update

执行这步指令后会触发两个文件夹的变动，分别是.git/modules和ddd-java<br />其中modules是新增的，里面还有一层文件是ddd-java，该文件下的内容就是和父级项目类似的.git文件夹<br />ddd-java是从空状态变成了有代码的状态<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/39158306/1717903601125-76fa78bb-c86b-492d-8741-106386d445dc.png#averageHue=%23272524&clientId=ud718a8e0-cd92-4&from=paste&height=308&id=u29836c68&originHeight=277&originWidth=533&originalType=binary&ratio=0.8999999761581421&rotation=0&showTitle=false&size=18902&status=done&style=none&taskId=uac0ac115-ea52-41f0-aaf4-92b4116e2b4&title=&width=592.2222379107538)<br />还有一点可以注意下：ddd-java子模块下的.git文件的内容，是一个链接文件

```shell
gitdir: ../.git/modules/ddd-java
```



### 2.3 git submodule foreach 'git checkout master'

这一步的目的主要是：

1. 查看子模块的分支：

```shell
DEEP@DEEP-Q23X99M3 MINGW64 /h/code/gitmoduledemo/gitSubModule/ddd-java ((v3.3.0))
$ git branch
* (HEAD detached at da58616)
  master
```

注意上述git bash是在子模块的ddd-java文件下执行的，也即有.git链接文件的那层目录<br />而在父级目录下，查看当前分支为：

```shell
DEEP@DEEP-Q23X99M3 MINGW64 /h/code/gitmoduledemo/gitSubModule (master)
$ git branch
* master
```

当前子模块处于指向一个特定提交状态，一般建议将其切换到稳定的分支上，可以执行

```shell
DEEP@DEEP-Q23X99M3 MINGW64 /h/code/gitmoduledemo/gitSubModule (master)
$ git submodule foreach 'git checkout master'
Entering 'ddd-java'
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

也即在父级.git同级目录下执行将子模块切换到master分支上的操作，然后去子模块目录下查看分支状态，发现已经指向了master分支上

```shell
DEEP@DEEP-Q23X99M3 MINGW64 /h/code/gitmoduledemo/gitSubModule/ddd-java ((v3.3.0))
$ git branch
* master
```



### 2.4 子仓库提交更新

分为两部分：子仓库修改提交并推送，父仓库同步子仓库的更改<br />**子仓库修改提交并推送**

1. 子仓库修改完毕后，在命令上进入子仓库所在的目录下执行暂存并提交

```bash
PS H:\code\gitSubModule\ddd-java> git add .
PS H:\code\gitSubModule\ddd-java> git commit -m 'modify readme.md test submodule commit'
```

2. 推送更改到远程子仓库

```bash
PS H:\code\gitSubModule\ddd-java> git push origin master
```

**父仓库同步子仓库的更改**

1. 回到父仓库目录，更新子模块引用

```bash
PS H:\code\gitSubModule> git submodule update --remote
```

不指定具体子模块名称时，如果像同时更新所有子仓库以获取远程的最新更改，可以直接运行不带子模块目录的命令。如果只想更新某个子模块，可以后面加上子模块名称，如ddd-java

3. 提交子模块引用更新

```bash
PS H:\code\gitSubModule> git add .gitmodules ddd-java
PS H:\code\gitSubModule> git commit -m "Update submodule ddd-java to its latest commit"
```

4. 推送父仓库更改

```bash
PS H:\code\gitSubModule> git push origin master
```

在维护含有子模块的项目时，确保父子仓库的同步性和关联性关键在于正确管理.gitmodules文件及子模块的实际内容。当子仓库有更新时，首先在子仓库内部完成修改并推送。随后，在父仓库层面，执行git submodule update --remote以拉取子模块的最新版本，并根据需要使用git add .gitmodules和git commit来记录子模块引用的任何变化，最后推送父仓库的更新。

补充：执行git submodule update --remote。这一步骤会将子模块更新到它们各自远程仓库的最新版本，同时自动处理.gitmodules文件中记录的子模块URL。注意，git submodule update --remote并不直接修改.gitmodules文件中的URL，而是根据该文件中的URL去获取最新的子模块代码。如果如果git submodule update --remote导致了.gitmodules文件内容的实际改变（例如，子模块的默认分支发生改变），需要通过git add .gitmodules来将这些更改添加到暂存区，然后执行git commit来记录这些更新。



## 3. 本地仓库获取远程新增子仓库

如果远程仓库新增子仓库，而本地没有，父仓库需要首先解决的是.gitmodules冲突问题，新增子仓库会在该文件下增加子仓库的元数据信息。

1. 获取最新.gitmodules，并解决可能存在的冲突

```shell
# 1. 备份本地修改
cp .gitmodules .gitmodules.backup
# 2. 丢弃本地修改
git checkout -- .gitmodules
# 拉取远程更改
git pull origin master
```

2. 初始化和更新子模块

```shell
git submodule update --init --recursive
```

3. 合并备份修改

补充：`git submodule update --init --recursive`<br />**git submodule update**: 用于更新项目的子模块。会检查每个子模块的HEAD指向的提交，并确保子模块的工作目录与该提交匹配。也即会把每个子模块的文件恢复到与父级项目记录的提交相匹配的状态。<br />**--init**: 告知Git不仅要更新已存在的子模块，还要初始化那些还没有被克隆过的子模块。这意味着如果你刚刚克隆了一个包含子模块的仓库，使用这个选项会自动下载并设置好所有子模块。<br />**--recursive**: 当子模块中还包含了其他子模块（即嵌套子模块）时，这个选项会递归地进入每个子模块并执行相同的操作，确保所有层级的子模块都被正确地初始化和更新。



## 4. 子仓库合并冲突

针对远程子仓库有更新，而本地子仓库也进行了独立的修改，为避免数据丢失或冲突。下面是实验过的解决方案：进入到子仓库根目录下

1. 修改代码前先获取子仓库更新

```shell
git submodule update --remote
# 切换分支，否则上一步会使HEAD处于游离状态
git submodule foreach 'git checkout main'
```

2. 如果是修改完本地代码准备提交时，发现远程子仓库更新【可能有问题，后续完善】

```shell
# 先将本地修改放入暂存区
git add .
git commit -m ''
# 获取远程代码
git pull 
# 处理冲突之后
git push
```

3. 父仓库更新子模块
4. 父仓库下的子模块更新也是要作为父仓库更新的一部分，也需要执行git add, commit

