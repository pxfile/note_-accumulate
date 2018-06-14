Git 操作
===


## git 命令

###   创建本地仓库

```
git init

```

###   获取远程仓库

```
git clone [url]
例：git clone https://github.com/you/yourpro.git

```

###   创建远程仓库

```
// 添加一个新的 remote 远程仓库
git remote add [remote-name] [url]
例：git remote add origin https://github.com/you/yourpro.git
origin：相当于该远程仓库的别名

// 列出所有 remote 的别名
git remote

// 列出所有 remote 的 url
git remote -v

// 删除一个 renote
git remote rm [name]

// 重命名 remote
git remote rename [old-name] [new-name]

```

###   从本地仓库中删除

```
git rm file.txt         // 从版本库中移除，删除文件
git rm file.txt -cached // 从版本库中移除，不删除原始文件
git rm -r xxx           // 从版本库中删除指定文件夹

```

###   从本地仓库中添加新的文件

```
git add .               // 添加所有文件
git add file.txt        // 添加指定文件

```

###   提交，把缓存内容提交到 HEAD 里

```
git commit -m "注释"

```

###   撤销

```
// 撤销最近的一个提交.
git revert HEAD

// 取消 commit + add
git reset --mixed

// 取消 commit
git reset --soft

// 取消 commit + add + local working
git reset --hard

```

###   把本地提交 push 到远程服务器

```
git push [remote-name] [loca-branch]:[remote-branch]
例：git push origin master:master

```

###   查看状态

```
git status

```

###   从远程库中下载新的改动

```
git fetch [remote-name]/[branch]

```

###   合并下载的改动到分支

```
git merge [remote-name]/[branch]

```

###   从远程库中下载新的改动

```
pull = fetch + merge

git pull [remote-name] [branch]
例：git pull origin master

```

###   分支

```
// 列出分支
git branch

// 创建一个新的分支
git branch (branch-name)

// 删除一个分支
git branch -d (branch-nam)

// 删除 remote 的分支
git push (remote-name) :(remote-branch)

```

###   切换分支

```
// 切换到一个分支
git checkout [branch-name]

// 创建并切换到该分支
git checkout -b [branch-name]

```
[Git笔记(三)——[cherry-pick, merge, rebase]](http://pinkyjie.com/2014/08/10/git-notes-part-3/)

### cherry-pick - 妈妈，我也要

比如我在A分支做了几次commit以后，发现其实我并不应该在A分支上工作，应该在B分支上工作，这时就需要将这些commit从A分支复制到B分支去了，这时候就需要cherry-pick命令了，B分支指着这些commit说：妈妈，我也要！

这个时候先新建一个分支`git checkout -b branch3 1a222c3`，注意这里的最后一个参数是新分支的起点，也就是说，新的分支branch3是从“commit 8,9”开始的，现在我们需要把刚才的两次提交移动到新的分支上。运行`git cherry-pick 0bda20e 1a04d5f`，

确定这两个commit被复制到指定分支以后，在master分支上将这两个commit删除。先切回master分支：`git checkout master`，运行`git reset --hard 1a222c3`，

### rebase - 我是直男，不喜欢弯的

总结一下rebase：

1.  `git rebase 目标分支`原理其实是先将HEAD指向目标分支和当前分支的共同祖先commit节点，然后将当前分支上的commit一个一个的apply到目标分支上，apply完以后再将HEAD指向当前分支。
2.  rebase与merge的区别：
    *   把master分支合并到别的分支用rebase，把别的分支合并到master分支上用merge
    *   rebase不会产生多余的commit，并且保持直线

_关于rebase其他要补充的_：

当然rebase还有其他很多很牛逼的功能，其“交互模式”可以让你干很多事情，比如调整commit的顺序啊，合并一些commit啊，删除一些commit啊等等，通过`-i`参数可以实现，当然这个命令有些复杂


## 与github建立ssh通信，让Git操作免去输入密码的繁琐。

*   首先呢，我们先建立ssh密匙。

> ssh key must begin with 'ssh-ed25519', 'ssh-rsa', 'ssh-dss', 'ecdsa-sha2-nistp256', 'ecdsa-sha2-nistp384', or 'ecdsa-sha2-nistp521'. -- from github

```
根据以上文段我们可以知道github所支持的ssh密匙类型，这里我们创建ssh-rsa密匙。
在command line 中输入以下指令:``ssh-keygen -t rsa``去创建一个ssh-rsa密匙。如果你并不需要为你的密匙创建密码和修改名字，那么就一路回车就OK，如果你需要，请您自行Google翻译，因为只是英文问题。

```

* $ ssh-keygen -t rsa Generating public/private rsa key pair. //您可以根据括号中的路径来判断你的.ssh文件放在了什么地方 Enter file in which to save the key (/c/Users/Liang Guan Quan/.ssh/id_rsa):

*   到 [https://github.com/settings/keys](https://github.com/settings/keys) 这个地址中去添加一个新的SSH key，然后把你的xx.pub文件下的内容文本都复制到Key文本域中，然后就可以提交了。
*   添加完成之后 我们用`ssh git@github.com` 命令来连通一下github，如果你在response里面看到了你github账号名，那么就说明配置成功了。 _let's enjoy github ;)_

## [](https://github.com/francistao/LearningNotes/blob/master/Part1/Android/Git%E6%93%8D%E4%BD%9C.md#gitignore)gitignore

* * *

在本地仓库根目录创建 .gitignore 文件。Win7 下不能直接创建，可以创建 ".gitignore." 文件，后面的标点自动被忽略；

```
/.idea          // 过滤指定文件夹
/fd/*           // 忽略根目录下的 /fd/ 目录的全部内容；
*.iml           // 过滤指定的所有文件
!.gitignore     // 不忽略该文件
```