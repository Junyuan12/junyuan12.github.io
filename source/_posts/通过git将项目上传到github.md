---
title: 通过git将项目上传到github
updated: 2019-04-09 12:00:00
categories: git
tags:
     - git
---



原文链接：https://blog.csdn.net/jerryhanjj/article/details/72777618

<!-- more -->

### 配置Git、SSH

+ 下载、安装Git

+ 绑定用户

  ```
  $ git config --global user.name "Your Name" 
  $ git config --global user.email "email@example.com"
  ```

+ 配置SSH

  在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开GitBash），创建`SSH Key`，密码可以不设置直接回车。

  ```
  $ ssh-keygen -t rsa -C "youremail@example.com"
  ```

  如果一切顺利的话，可以在用户主目录里找到 `.ssh​ `目录，里面有 `id_rsa` 和`id_rsa.pub` 两个文件，这两个就是`SSH Key` 的秘钥对，**id_rsa 是私钥，不能泄露出去**，`id_rsa.pub`是公钥，可以放心地告诉任何人。用记事本打开 `id_rsa.pub`，得到`ssh key` 公钥。

  为 Github 账户添加 ssh key 。登录 Github，展开个人头像的小三角，点`settings`，然后打开`SSH keys`菜单，点击`Add SSH key`新增密钥，填上标题。

### 上传项目

+ **建立仓库**

  填一下仓库名称，Initialize this repository with a README是可选的，**建议在创建时选上**，可以在后面省一个步骤。填好之后，点Create repository完成仓库的建立。

+ **克隆仓库**

  **如果是全新的项目没有任何文件，也可以不用克隆仓库，跳过这一步。**点开 Git Shell，进入命令行。首先我们先要把 GitHub 上的我们新建的仓库 clone下来。在初始化版本库之前，先要确认认证的公钥是否正确。

  ```
  $ ssh -T git@github.com
  ```

  如果收到成功的确认消息，就可以开始克隆远程仓库了。

  ```
  $ git clone https://github.com/username/program-name.git
  ```

  克隆仓库之后就在文件夹中出现了项目文件夹及文件,**进入项目文件夹**，对其进行初始化。

  ```
  $ git init
  ```

+ **上传README文件**

  如果在创建 `Github` 仓库时没有勾选创建 `README.md` 文件，则要先创建 `README.md` 文件，不然上传文件会报错。如果已经勾选，可以跳过此步骤。

  ```
  $ git init
  $ touch README.md
  $ git add README.md
  $ git commit -m 'first_commit'
  $ git remote add origin 
  ```

+ **上传项目**

  跟踪项目文件夹中的所有文件和文件夹：

  ```
  $ git add . 
  ```

  输入本次的提交说明，准备提交暂存区中的更改的已跟踪文件，单引号内为说明内容：

  ```
  $ git commit -m 'first_commit'
  ```

  关联远程仓库，添加后，远程库的名字就是 `origin`，这是 `Git` 默认的叫法，也可以改成别的，但是 `origin` 这个名字一看就知道是远程库。

  ```
  $ git remote add origin https://github.com/username/program-name.git
  ```

  如果关联出现错误 `fatal: remote origin already exists`，则执行`git remote rm origin`再进行关联。

  把本地库的所有内容推送到远程库上：

  ```
  $ git push -u origin master
  ```

  如果在推送时出现错误 `error:failed to push som refs to.......`，则执行下列语句：

  ```
  $ git pull origin master
  ```

  将远程仓库 `Github` 上的文件拉下来合并之后重新推送上去。