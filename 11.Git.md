## Git

### 工作区

工作区相当于本地，工作区通过git add将文件提交到暂存区

### 暂存区

暂存区相当于一个独立的storage，即使是离线也可以先add到暂存区
暂存区内的内容需要通过git commit才能提交到分支

### 分支

分支相当于最后的服务器

### 在本地创建版本库(repository)

```
mkdir newFile
cd newFile
git init
mkdir -- 创建新文件夹
git init --在文件夹下创建版本库
```

### 添加修改的文件到暂存区

```
git add changedFile.js
git status
git diff changedFile.js
git commit -m "新增了changedFile"
git add --添加文件到暂存区
git status --查看暂存区状态
git diff --查看工作区与暂存区的差异
git commit -m --将暂存区内的内容提交到分支，-m后面跟的是这次提交内容的说明
```

### 远程仓库

#### 先本地创建了版本库

1. github上点击**Create a new repo**钮，创建一个新的仓库
1. 在**Repository name**填入**仓库名称**，在**Description**填入**说明(可选)**，点击**Create repository**按钮
1. 使用`git remote add origin git…………………………`代码关联本地版本库
1. 使用`git push -u origin master`将所有内容推送到远程库
1. 或者**使用github提示的代码**将所有内容推送到远程库
1. 之后本地做了提交，可以使用`git push origin master`将**master分支**的最新修改推送到github

#### 先创建远程库

1. github上点击**Create a new repo**钮，创建一个新的仓库
1. 在**Repository name**填入**仓库名称**，在**Description**填入**说明(可选)**，勾选**Initialize this repository with a README**让github为我们创建一个readme.md文件，点击**Create repository**按钮
1. 使用`git clone git…………………………`克隆到本地库

### 获取新版本

#### git fetch

从远程获取最新版本到本地，但不会自动合并

#### git pull

相当于是从远程获取最新版本并合并到本地
`git pull` = `git fetch` + `git merge`

### 暂存修改内容

#### git stash

可以将当前修改的内容暂存

#### git stash list

查看暂存内容列表

#### git stash pop

还原暂存内容

#### git stash clear

清空stash list

### 分支

#### git switch -c dev(推荐)

创建并切换到dev分支
`git switch -c dev` = `git branch dev` + `git switch dev`

#### git checkout -b dev

创建并切换到dev分支
`git checkout -b dev` = `git branch dev` + `git checkout dev`

#### git branch

`git branch`会列出所有分支，当前分支前会有`*`

#### git merge

用于合并**指定分支**到**当前分支**
比如要将dev分支合并到master分支

```
git checkout master //切换到master分支
git merge dev //合并dev分支
```

#### git branch -d dev

删除本地dev分支

#### 修改当前项目的用户名与邮箱

```
git config user.name username
git config user.email useremail
```

#### 修改全局的用户名与邮箱

```
git config --global user.name username
git config --global user.email useremail
```

#### 让git使用clash的socks代理通道

在`.git/config`中，输入：

```
[core]
	gitproxy = socks5://127.0.0.1:7890
[http]
	postBuffer = 524288000
	postBuffer = 524288000
	proxy = socks5://127.0.0.1:7890
[https]
	postBuffer = 524288000
	postBuffer = 524288000
	proxy = socks5://127.0.0.1:7890
```

#### 不让git使用代理

```
git config --global --unset http.proxy
```

#### 更新windows的git版本

```
git update-git-for-windows
```

#### 避免重复输入git账号和密码（凭证储存）

在`.git/config`中，输入：

```
[credential]
	helper = store
```
或者

```
[credential]
	helper = cache
```

区别是cache临时的，store是永久的

全局命令行：

```
git config --global credential.helper store
git config --global credential.helper cache
```

#### 提交时使用personal access token代替password登录git

[github官网](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)