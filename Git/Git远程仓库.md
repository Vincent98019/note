
gitHub（ 地址：[https://github.com/](https://github.com/) ）是一个面向开源及私有软件项目的托管平台，因为只支持 Git 作为唯一的版本库格式进行托管，故名gitHub。

码云（地址： https://gitee.com/ ）是国内的一个代码托管平台，由于服务器在国内，所以相比于 GitHub，码云速度会更快。

GitLab （地址： https://about.gitlab.com/ ）是一个用于仓库管理系统的开源项目，使用Git作 为代码管理工具，并在此基础上搭建起来的web服务,一般用于在企业、学校等内部网络搭建git私服。


## 配置SSH公钥

### 生成SSH公钥

```bash
ssh-keygen -t rsa
```

不断回车，如果公钥已经存在，则自动覆盖



### 获取公钥

```bash
cat ~/.ssh/id_rsa.pub
```


### 验证是否配置成功

```bash
ssh -T git@gitee.com
```


## 操作远程仓库

### 添加远程仓库

此操作是先初始化本地库，然后与已创建的远程库进行对接。

```bash
git remote add <远端名称> <仓库路径>
# 远端名称，默认是origin，取决于远端服务器设置
# 仓库路径，从远端服务器获取此URL
# 例如: git remote add origin git@gitee.com:czbk_zhang_meng/git_test.git
```


### 查看远程仓库

```bash
git remote
```


### 推送到远程仓库

```bash
git push [-f] [--set-upstream] [远端名称 [本地分支名][:远端分支名] ]

# 如果远程分支名和本地分支名称相同，则可以只写本地分支
# git push origin master

# -f 表示强制覆盖

# --set-upstream 推送到远端的同时并且建立起和远端分支的关联关系。
# git push --set-upstream origin master

# 如果当前分支已经和远端分支关联，则可以省略分支名和远端名。
# git push 将master分支推送到已关联的远端分支。
```


### 本地分支与远程分支的关联关系

```bash
 git branch -vv
```


### 从远程仓库克隆

如果已经有一个远端仓库，可以直接clone到本地。

```bash
git clone <仓库路径> [本地目录]
# 本地目录可以省略，会自动生成一个目录
```


### 从远程仓库中抓取和拉取

远程分支和本地的分支一样，可以进行merge操作，只是需要先把远端仓库里的更新都下载到本地，再进行操作。

#### 抓取

抓取指令就是将仓库里的更新都抓取到本地，不会进行合并，如果不指定远端名称和分支名，则抓取所有分支。

```bash
git fetch [remote name] [branch name]
```

#### 拉取

拉取指令就是将远端仓库的修改拉到本地并自动进行合并，等同于fetch+merge，如果不指定远端名称和分支名，则抓取所有并更新当前分支。

```bash
git pull [remote name] [branch name]
```


### 解决合并冲突

远程分支也是分支，所以合并时冲突的解决方式也和解决本地分支冲突相同相同。