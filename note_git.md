---
title: git
created: 2023-10-17
---
> [!cite]- References  
> [Pro Git](https://git-scm.com/book/zh/v2)
>
## 简介

### 版本控制

版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统  

1. 本地版本控制系统  
    使用某种简单的数据库记录文件的历次更新差异  
     - RCS(Revision Control System)  
        RCS在本地保存文件修改前后的变化集合，通过应用这些变化可以计算出各个版本文件内容  
2. 集中化版本控制系统(CVCS, Centralized Version Control System)  
    为解决不同系统开发者协同工作的问题  
    有一个单一的集中管理的服务器保存所有文件的修订版本  
3. 分布式版本控制系统(DVCS, Distributed Version Control System)  
    为解决集中化版本控制系统中央服务器单点故障问题  
    客户端会将代码仓库包括历史记录完整地镜像下来，若服务器出现故障可由本地仓库恢复  

### git 与其他版本控制工具差异  

1. 直接记录快照而非差异比较  
    **基于差异**(delta-based)的版本控制系统采用增量存储，即每次改动时存储相对于上一个版本变化的内容  
    git 采用快照存储：对于没有变化的文件，生成指向原文件的引用，对于变化了的文件则存储整个文件  
    > 实际上并非每次都记录完整文件，git 对于这个问题做了[优化](#包文件)  
2. 几乎所有操作在本地执行  
    绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其它计算机的信息  
    无需在线提交，修改先提交到本地仓库，必要时再提交到远程仓库  
3. 保证完整性  
    git 数据在存储前会通过 SHA-1 哈希(或散列)记录校验和，通过校验和来引用数据  
4. 一般只添加数据  
    用户执行的 git 操作几乎只向 git 数据库中添加数据，几乎不会导致文件不可恢复  
5. git 的三种状态  
    - **已修改**(modified)  
        已修改表示修改了文件，但还没保存到数据库中  
    - **已暂存**(staged)  
        对已修改文件的当前版本做了标记，使之包含在下次提交的快照中  
    - **已提交**(committed)  
        数据已经安全地保存在本地数据库中  

    基本工作流程如下  
    1. 在工作区中修改文件  
    2. 将你想要下次提交的更改选择性地暂存，这样只会将更改的部分添加到暂存区  
    3. 提交更新，找到暂存区的文件，将快照永久性存储到 ==.git/== 目录  

### git 配置  

git 配置文件存储在三个位置，每一级的配置变量会覆盖上一级的配置变量  
详见[配置 git](#配置%20git)  

1. ==/etc/gitconfig==  
    适用于系统上每一个用户的通用配置  
    执行 git 命令时指定 `--system` 会强制使用该配置  
2. ==~/.gitconfig==  
    当前用户的配置  
    执行 git 命令时指定 `--global` 会强制使用该配置  
3. ==.git/config==  
    当前仓库的配置  

```shell
#列出配置变量
git config --list   
#列出特定配置变量，如user.name等
git config <key>
#列出所有配置变量及其所在的配置文件路径
git config --list --show-origin
#配置文本编辑器路径
git config --global core.editor <path>
#配置用户名和邮件，其会写入每一次提交
git config --global user.name <name>
git config --global user.email <email-address>
```

## git 基础  

### 获取 git 仓库  

1. 尚未进行版本控制的本地目录转换为 git 仓库  

    ```shell
    git init
    ```

    该命令在目录下创建 ==.git/== 目录，其中存有 git 仓库的所有必须文件  
2. 从其他服务器克隆已存在的 git 仓库  

    ```shell
    git clone <url>
    ```

    该命令在当前目录下新建与仓库同名目录，并在新建目录下初始化 ==.git/== 目录，将远程仓库数据放入 ==.git/== 目录，同时读取最新版本的文件  

### 记录更新到仓库  

1. 检查文件状态  

    ```shell
    git status
    ```

    该命令列出工作目录下哪些文件处于什么状态  
2. 跟踪新文件  

    ```shell
    git add <filename>
    ```

    对于未跟踪的文件，该命令开始跟踪一个文件  
3. 暂存已修改的文件  

    ```shell
    git add <filename>
    ```

    对于已跟踪的文件，该命令将其放到暂存区  
    `git add` 更精确的含义为 "将内容添加到下一次的提交中"  
    当已暂存文件后再次修改文件，需要重新使用 `git add` 将最新修改版本暂存  

4. 忽略文件  
    通过 ==.gitignore== 文件来描述要忽略的模式，其格式规范如下  
    - 忽略空行以及 `#` 开头的行  
    - 可以使用 glob 模式匹配  
        > glob 是 shell 使用的简化版正则表达式  
    - 匹配模式以 `/` 开头防止递归  
    - 匹配模式以 `/` 结尾指定目录  
    - `!` 表示取反，即指定模式以外的文件或目录  

5. 查看已暂存和未暂存的修改  

    ```shell
    #显示尚未暂存的文件与已暂存的文件内容的差异部分
    git diff
    #显示已暂存的文件与最后一次提交的文件内容的差异部分
    git diff --staged
    #或者
    git diff --cached
    ```

6. 提交修改  

    ```shell
    #提交已暂存的修改
    git commit -m <comment>
    #自动暂存所有已跟踪的文件并提交修改
    git commit -a -m <comment>
    ```

7. 移除文件  

    ```shell
    #停止跟踪文件并删除本地文件
    git rm <filename>
    #停止跟踪文件但保留本地文件
    git rm --cached <filename>
    #停止跟踪已经放到暂存区的文件
    git rm -f <filename>
    ```

8. 移动文件  

    ```shell
    git mv <old_filename> <new_filename>
    ```

    实际上等价于  

    ```shell
    mv <old_filename> <new_filename>
    git rm <old_filename>
    git add <new_filename>
    ```

### 查看提交历史  

```shell
#列出提交历史，包括每次提交的SHA-1校验和，作者及其邮件，提交时间，提交说明
git log
#额外显示每次提交的差异
git log -p
git log --patch
#额外显示简略版的提交差异
git log --stat
#以特定格式显示提交历史
git log --pretty=<option>
```

`--pretty` 的可选项有 `online`，`full`，`short`，`fuller`，`format` 等  
对于 `format`，其类似 C 语言的 `printf` 的格式化输出字符串，以自定义格式输出提交历史，形如  

```shell
git log --pretty=format:"%h - %an, %ar"
```

> [!note]- `pretty` 格式占位符  
> 
> | 选项  | 说明  |
> | -- | -- |
> |%H |提交的完整哈希值|
> |%h|提交的简写哈希值|
> |%T|树的完整哈希值|
> |%t|树的简写哈希值|
> |%P|父提交的完整哈希值|
> |%p|父提交的简写哈希值|
> |%an|作者名字|
> |%ae|作者的电子邮件地址|
> |%ad|作者修订日期（可以用 --date=选项 来定制格式）|
> |%ar|作者修订日期，按多久以前的方式显示|
> |%cn|提交者的名字|
> |%ce|提交者的电子邮件地址|
> |%cd|提交日期|
> |%cr|提交日期（距今多长时间）|
> |%s|提交说明    |
> 


> [!note]- 过滤提交历史参数  
> 
> | 选项                | 说明                   |
> | ----------------- | -------------------- |
> | -\<n\>            | 仅显示最近的 n 条提交         |
> | --since, --after  | 仅显示指定时间之后的提交         |
> | --until, --before | 仅显示指定时间之前的提交         |
> | --author          | 仅显示作者匹配指定字符串的提交      |
> | --committer       | 仅显示提交者匹配指定字符串的提交     |
> | --grep            | 仅显示提交说明中包含指定字符串的提交   |
> | -S                | 仅显示添加或删除内容匹配指定字符串的提交 |
> 

### 撤销

1. 重新提交  
    本次提交将覆盖上一次提交  
    对于提交信息，本次将覆盖上一次；对于暂存区的修改，会将两次的修改合并，若有冲突则需解决冲突  
    从提交历史来看，相当于上一次的 `git commit` 没有执行  

    ```shell
    git commit --amend
    ```

2. 取消暂存  
    将文件移出暂存区  

    ```shell
    git reset HEAD <filename>
    ```

> [!danger] `git reset` 本身作用不止于此，且存在一定危险性

3. 撤销对文件的修改/将文件还原上次提交的样子  
    用上次提交的版本覆盖当前版本  

    ```shell
    git checkout -- <filename>
    ```

> [!danger]  
> 该文件本地的任何修改都会消失且无法恢复  
> git中已提交的数据几乎总是可以恢复，但提交前的数据丢失后无法恢复  

### 远程仓库  

1. 查看远程仓库  

    ```shell
    #列出当前文件夹下本地仓库对应的远程仓库
    git remote
    #额外列出其URL
    git remote -v
    ```

2. 添加远程仓库  

    ```shell
    git remote add <shortname> <url>
    ```

3. 从远程仓库拉取(pull)与获取(fetch)  

    ```shell
    git fetch <remote>
    ```

    `git fetch` 访问远程仓库获取还没有的数据到本地仓库，但不会应用到本地工作目录  

    ```shell
    git pull
    ```

    `git pull` 适用于当前分支设置了跟踪远程分支时，访问远程仓库拉取还没有的数据到本地仓库，并将其与本地分支合并(merge)，若有冲突则需解决冲突  

4. 推送到远程仓库  

    ```shell
    git push <remote> <branch>
    ```

    将本地当前分支推送到远程分支  
5. 查看远程仓库  

    ```shell
    git remote show <remote>
    ```

6. 重命名远程仓库  

    ```shell
    git remote rename <old_remote_name> <new_remote_name>
    ```

7. 移除远程仓库  

    ```shell
    git remote remove <remote>
    ```

### 标签(NFY)  

可以给某一次提交打上标签  
标签名不可重复，标签与提交是多对一的关系  

1. 列出标签  

    ```shell
    #列出当前仓库所有的标签
    git tag
    #过滤特定标签
    git tag -l <glob匹配>
    ```

2. 创建标签  
    - **轻量标签**(lightweight)  
        轻量标签仅仅是对一次提交的引用，本质是将提交校验和存储到一个文件中  

        ```shell
        #不指定提交则默认为上一次提交
        git tag <tag> <commit>
        ```

    - **附注标签**(annotated)  
        附注标签是存储在 git 数据库中的一个完整对象，包含打标签者的名字、邮件地址、日期时间、标签信息、标签说明信息  

        ```shell
        #不指定提交则默认为上一次提交
        git tag -a <tag> -m <comment> <commit>
        ```

3. 推送标签  
    `git push` 不会推送标签信息，必须显式推送标签  

    ```shell
    #推送单个标签
    git push <remote> <tag>
    #推送多个标签，会推送所有远程仓库没有的标签
    git push <remote> --tags
    ```

4. 删除标签  

    ```shell
    #删除本地标签
    git tag -d  <tag>
    #删除远程标签
    git push <remote> ：refs/tags/<tag>
    #或
    git push <remote> --delete <tag>
    ```

5. 检出标签  

### git 别名  

```shell
git config --global alias.<alias> [!]<command>
```

则 `git <alias>` 将等价于 `git <command>`  

对于非 git 子命令，定义别名时在命令前加入 `!`，则 `<alias>` 等价于 `<command>`  

## git 分支模型  

git 的每一次提交对应着三种对象，三者分别对应文件快照、目录结构以及本次提交  
详见章节 [git 对象](#git%20对象)  

1. blob对象  
    暂存(stage)文件时，会为被暂存的文件计算校验和，并将其文件快照保存为 blob 对象  
2. **树**(tree)对象  
    提交(commit)时，git 会计算每一个子目录的校验和并将其保存为 tree 对象  
3. **提交**(commit)对象  
    创建树对象后，git 会创建提交对象，其中包含作者姓名和邮箱，提交说明信息，指向其父对象的指针(首次提交没有父对象，合并产生的有多个父对象)，指向 tree 对象的指针  
    所有的提交对象的关系构成了该项目的单向无环图  

对于每一个分支，都有其对应的指针，指向该分支当前所在的提交对象  
git 通过指向分支指针的指针 `HEAD` 来确定当前所在分支  

### 分支的新建与合并  

1. 新建分支  

    ```shell
    #创建分支并切换
    git checkout -b <new_branch>
    #等价于如下两条命令
    git branch <new_branch>
    git checkout <new_branch>
    ```

2. 合并分支  

    ```shell
    #将<branch>合并到当前分支
    git merge <branch>
    ```


> [!example] 快进式合并  
> 当 `<branch>` 分支指向的提交是当前分支指向的提交的直接后继时--称作**快进式**(fast-forward)--两者不会产生冲突，git 会直接移动当前分支的提交指针  
> ```mermaid  
> ---
> titile: "Git Merge Example"
> ---
> gitGraph  
> 	commit id: "C1"
> 	commit id: "C2" tag:"main pointer"
> 	branch "fix_branch"
> 	checkout "fix_branch"
> 	commit id: "C3" tag:"fix_branch pointer"
> 	checkout "main"
> ```
> 
> ```shell
> git checkout main
> git merge fix_branch  
> ```
> ```mermaid  
> ---
> titile: "Git Merge Example"
> ---
> gitGraph  
> 	commit id: "C1"
> 	commit id: "C2"
> 	branch "fix_branch"
> 	checkout "fix_branch"
> 	commit id: "C3" tag:"fix_branch pointer"  tag:"main pointer"
> 	checkout "main"
> ```

> [!example] 非快进式合并  
> 
> 当 `<branch>` 分支指向的提交不是当前分支指向的提交的直接后继时--称作**非快进式**(non-fast-forward)--git 会使用两个分支末端的快照以及两者的最后公共祖先做合并，产生一个新的快照并创建一个新的提交指向它，称作合并提交，合并提交有不止一个父提交  
> 
> ```mermaid  
> gitGraph  
> 	commit id: "C1"
> 	commit id: "C2"
> 	
> 	branch "fix_branch"
> 	commit id: "C3"
> 	
> 	checkout "main"
> 	commit id: "C4" tag:"main pointer"
> 	
> 	checkout "fix_branch"
> 	commit id: "C5" tag:"fix_branch pointer"
> ```
> ```shell
> git checkout main
> git merge fix_branch
> ```
> 
> ```mermaid  
> gitGraph  
> 	commit id: "C1"
> 	commit id: "C2"
> 	
> 	branch "fix_branch"
> 	commit id: "C3"
> 	
> 	checkout "main"
> 	commit id: "C4" 
> 	
> 	checkout "fix_branch"
> 	commit id: "C5" tag:"fix_branch pointer"
> 	
> 	checkout "main"
> 	merge "fix_branch" id:"C6" tag:"main pointer"
> ```
> 

3. 删除分支  
    对于不再使用的分支，可以删除  
    但若分支中包含尚未合并的部分则会失败，此时使用 `-D` 可以强制删除  

    ```shell
    git branch -d <branch>
    ```

    当两个分支中对同一个文件的同一个部分做了不同修改时，会产生冲突，需要解决冲突后暂存并提交  

### 分支管理  

```shell
#列出所有分支
git branch
#额外列出每个分支的最后一次提交
git branch -v
#过滤已经合并到当前分支的分支
git branch --merged
```

### 分支开发工作流  

1. 长期分支  
	一种做法是 `master` 分支用于保留稳定版，`develop` 分支用于开发，一旦 `develop` 达到稳定状态就可以将其合并入 `master` 分支  

2. 主题分支  
	按照需求的目标特性创建多个新的分支，当特性开发完毕后再将其合并入主分支  

### 远程分支

1. 远程跟踪分支  
	远程跟踪分支是对远程分支状态的本地引用，即最后一次连接到远程仓库时的状态，在本地以 `<remote>/<branch>` 的形式命名，其无法被移动  

2. 推送分支  
	本地的非远程跟踪分支不会自动与远程仓库同步，除非显式地推送  

    ```shell
    git push <remote> <branch>
    ```

    `<branch>` 会被自动展开为 `refs/heads/<branch>:refs/heads/<branch>`，即"推送本地的 `<branch>` 分支来更新远程的 `<branch>` 分支"  

    当本地分支与远程分支不同名时，可以使用如下语法  

    ```shell
    git push <remote> <local_branch>:<remote_branch>
    ```

3. 跟踪分支  
	从远程跟踪分支检出一个本地分支会自动创建跟踪分支，跟踪分支**拉取**(pull)，**获取**(fetch)远程数据时无需显式指定远程仓库的分支  

    ```shell
    #创建跟踪分支（创建本地分支并跟踪远程分支）
    git checkout -b <branch> <remote>/<branch>
    #简化版
    git checkout --track <remote>/<branch>
    #再简化版（当本地不存在<branch>同名分支，且远程只有一个名字匹配的分支）
    git checkout <branch>
    ```

4. 拉取  
    `git pull` 在大多数情况下等价于 `git fetch`+`git merge`  
    通常情况下还是使用 `git fetch`+`git merge`  

    `git fetch` 只会获取仓库数据，不会改动工作目录  
5. 删除远程分支  
	删除远程仓库的远程分支指针，但不会立即删除数据，数据将会保留一段时间直到垃圾回收运行  

	```shell
	git push <remote> --delete <branch>
	```

### 变基/衍合  

一个基本原则是**不要在协作的分支上使用变基，只对尚未推送或分享给别人的本地修改执行变基操作清理历史**，在协作分支上使用变基会导致与他人的提交混乱  

```shell
#将当前分支变基到目标分支
git rebase <branch>
```

> [!example] 变基  
> 变基会先找到两个分支的最近共同祖先，然后获取两个分支祖先之后提交的不同部分，将本次分支的提交在目标分支上依次应用，并移动当前分支指针指向新的提交  
> 
> ```mermaid
> gitGraph
> 	commit id: "C1"
> 	commit id: "C2"
> 
> 	branch "fix_branch"
> 	checkout "main"
> 	commit id:"C3" tag:"main pointer"
> 
> 	checkout "fix_branch"
> 	commit id: "C4" tag:"fix_branch pointer"
> ```
> ```shell
> git checkout fix_branch
> git rebase main
> ```
> ```mermaid
> gitGraph
> 	commit id: "C1"
> 	commit id: "C2"
> 
> 
> 	
> 	checkout "main"
> 	commit id:"C3" tag:"main pointer"
> 	branch "fix_branch"
> 	commit id: "C4'" tag:"fix_branch pointer"
> ```
> 
> 然后切回 `main` 分支，使用 merge 移动 `main` 指针，此时满足快进式提交的条件  
> ```shell
> git checkout main
> git merge branch
> ```
> ```mermaid
> gitGraph
> 	commit id: "C1"
> 	commit id: "C2"
> 
> 
> 	
> 	checkout "main"
> 	commit id:"C3" 
> 	branch "fix_branch"
> 	commit id: "C4'" tag:"fix_branch pointer" tag:"main pointer"
> ```

## 服务器上的 git（NFY）  

### 协议

git 支持四种协议来传输资料：本地协议(Local Protocol)，HTTP 协议，SSH(Secure Shell)协议，Git 协议  

#### 本地协议  

```shell
git clone <path>
#或者
git clone file://<path>
```

适用于远程仓库为同一主机上的另一个目录  
常见于团队每一个人都对一个共享的文件系统拥有访问权(如NFS,Network File System)  

对于直接指定路径，git 会尝试使用硬链接或直接复制  
若指定 `file://`，git 会触发用于网络传输资料的进程，传输效率更低；`file://` 主要用于取得没有外部引用或对象的干净版本库副本  

 > [!success] 优点  
> - 简单，直接使用了现有的共享文件系统  

> [!failure] 缺点  
> - 不方便从多个位置访问，必须先挂载远程磁盘且配置不方便  
> - 速度慢，速度不如网络连接访问  
> - 安全性不足，任意用户都拥有仓库所在目录的完整shell权限  

#### HTTP 协议  
详见[传输协议](#传输协议)  

```shell
git clone <url>
```

git 共有两种 HTTP 协议，旧版本-称为**哑 HTTP 协议**(Dumb HTTP Protocol)，1.6.6版本引入的更智能的协议-称为智能 HTTP 协议  
通常二者择一使用，极少会将二者混合使用  

1. 哑 HTTP 协议  
	将裸版本库视作普通文件，提供文件服务  
2. 智能 HTTP 协议  
	运行在标准 HTTP/S 端口，运行方式类似 SSH 与 git 协议  
	支持类似 git 协议的匿名服务、类似 SSH 协议的传输授权和加密  

 > [!success] 优点  
> - 智能 HTTP 协议，不同的访问方式只需要一个 URL 以及服务器只在需要授权时提示输入授权信息  
> - HTTP/S 协议广泛使用，在几乎所有系统上都可用  
> - 传输高效  

> [!failure] 缺点  
> - 某些服务器上架设 HTTP/S 服务比 SSH 服务棘手一些  
#### SSH协议  

```shell
git clone ssh://[<user>@]<server>/<project>.git
#SCP简化版命令
git clone [<user>@]<server>:<project>.git
```

架设 git 服务器时使用 SSH 协议  

> [!success] 优点  
> - SSH 架设相对简单  
> - 安全性，SSH 所有传输数据都要经过授权和加密  
> - 传输高效  

> [!failure] 缺点  
> - 不支持匿名访问git仓库
#### git协议  
包含在 git 里的一个守护进程，监听 9418 端口  

> [!success] 优点  
> - 传输速度快，git 协议传输速度是所有协议里最快的  

> [!failure] 缺点  
> - 没有授权机制，是否能读写对所有人都是一致的，一般只开放读权限，写权限通过其他协议  
> - 架设困难，要求守护进程，需要配置 xinetd、systemd 或其他程序  
> - 端口问题，9418 端口不常用，企业防火墙一般不开放甚至封锁该端口  

## 分布式 git  

### 分布式工作流程  

1. 集中式工作流  
    集中式系统中通常使用的单点协作模型  
    中心仓库可以接收代码，所有开发者将自己的工作与之同步  
    当提交的修改为非快进式时，必须将数据抓取下来并且合并后方能推送

2. 集成管理者工作流/多仓库工作流  
    GitHub 与 GitLab 常用的工作流程  
    每个开发者拥有自己仓库的写权限与其他开发者仓库的读权限  
    贡献者克隆主仓库，作出修改后，申请主仓库维护者拉取并合并自己的修改，通过后维护者将合并后的修改推送到主仓库  

3. 主管与副主管工作流  
    多仓库工作流的变种，一般用于超大型项目  
    **副总管**(lieutenant)负责项目中的特定部分，**主管**(dictator) 负责整体统筹  
    普通开发者在自己的主题分支上工作，副主管将普通开发者的主题分支合并到自己的 `master` 分支，主管将所有副主管的 `master` 分支合并到自己的 `master` 分支，最后将 `master` 分支推送到远程仓库  

### 向项目贡献(NFY)

1. 提交准则  
	- 提交不应包含空白错误  
		`git diff --check` 会找到可能的空白错误并将它们为你列出来  
	- 让每一个提交成为一个逻辑上的独立变更集  
	- 创建优质提交信息  
		一般情况下，信息应当以少于 50 个字符（25个汉字）的单行开始且简要地描述变更，接着是一个空白行，再接着是一个更详细的解释  
		使用指令式的语气来编写提交信息，比如使用 "Fix bug" 而非 "Fixed bug" 或 "Fixes bug"  

## git 工具

### 选择修订版本  

任意一个提交都可由 40 个字符的 SHA-1 散列值确定  
> SHA-1已被攻破，但对于git的非加密用途没有影响  
> 若SHA-1产生碰撞，git将认为该对象已经存在于仓库中，但这种情况概率是十分低，$2^{80}$个哈希对象才有50%概率出现一次碰撞  

1. 简短的SHA-1  
    对于没有歧义的 SHA-1 散列，只需要给出至少前4个字符即可获取指定的提交  

    ```shell
    #仅显示简短SHA-1而非完整SHA-1
    git log --abbrev-commit
    ```

2. 分支引用  
    对于分支的当前提交，可由分支名指代提交的 SHA-1 值  

    ```shell
    git show <branch>
    ```

3. 引用日志  
    git 会在后台保存**引用日志**(reflog)，记录了近几个月 `HEAD` 和分支引用的历史  
    引用日志只存在于本地仓库  

    ```shell
    #查看引用日志
    git reflog
    #通过分支指针(包括HEAD)指定提交，其中<n>为reflog输出的数字
    git show <branch>@{<n>}
    #查看分支在一定时间之前的位置
    git show <branch>@{yesterday}
    ```

4. 祖先引用  
    可以通过在引用尾部加上 `^` 来指定该引用的父提交  

    ```shell
    #<reference>可以为SHA-1，分支引用等
    git show <reference>^
    #指定第<n>个父提交，仅用于有多个父提交的合并提交
    git show <reference>^<n>
    #指定第<i>级祖先提交，例如i为2时表示祖父提交；i为2时可省略
    git show <reference>~<i>
    ```

> [!warning] Windows 的 CMD 中，`^` 被用作转义字符，需要双 `^` 转义  
5. 提交区间  
    1. `..`  
        指定在一个分支而不在另一个分支的提交，即分支1相对于分支2的差集  

        ```shell
        git log <branch_1>..<branch_2>
        ```

    2. `^` 或 `--not`  
        可以在引用前加上 `^` 或 `--not`，表明不在该分支引用的提交，即相对于该分支的差集  

        ```shell
        git log <branch_1> ^<branch_2> --not <branch_3>
        ```

    3. `...`  
        指定属于两个分支但不同时被包含的分支，即两者的补集  

### 交互式暂存  

将文件的特定部分组合成提交  
用于大量修改后拆分为若干次提交  

```shell
#进入交互式模式
git add -i
#或者
git add --interactive
#输出大致如下
           staged     unstaged path
  1:    unchanged        +0/-1 TODO
  2:    unchanged        +1/-1 index.html
  3:    unchanged        +5/-1 lib/simplegit.rb

*** Commands ***
  1: [s]tatus     2: [u]pdate      3: [r]evert     4: [a]dd untracked
  5: [p]atch      6: [d]iff        7: [q]uit       8: [h]elp
What now>
```

1. 暂存补丁  
    在交互式模式输入 `p`，git 会询问部分暂存哪些文件，然后对这些文件修改的部分，依次显示区别并询问是否暂存  
    也可以使用如下命令  

    ```shell
    git add -p
    #或者
    git add --patch
    ```

### 贮藏与清理  

1. **贮藏**(stash)  

    ```shell
    git stash [--include-untracked|-u] [--all|-a] [--patch]
    #或
    git stash push
    #查看贮藏
    git stash list
    #重新应用贮藏
    git stash apply [stash@{<n>}] [--index]
    #丢弃贮藏
    git stash drop [stash@{<n>}]
    ```

    用于切换分支时保留未完成的修改，即已修改和已暂存的已跟踪修改，若指定 `--include-untracked`，则也会贮藏未跟踪文件，但不会包含已忽略的文件，除非指定 `--all`  
    贮藏时指定 `--patch` 会交互式地提示哪些需要贮藏，哪些需要保存在工作目录中  
    git 使用栈来存储贮藏内容，故恢复栈顶贮藏时可省略 `stash@{<n>}`  
    重新应用贮藏后，原先的修改都会被移出暂存区，除非指定 `--index`  
    重新应用贮藏并不要求在原先的分支，若有冲突则需要解决  

2. 从贮藏创建分支  

    ```shell
    git stash branch <new_branch>
    ```

3. 清理工作目录  
    从工作目录移除未追踪的文件  

    ```shell
    git clean
    #交互模式运行clean
    git clean --interactive|-i
    ```

### 签名  

用于验证提交是否来自可信来源  

1. GPG  

    ```shell
    #生成密钥
    gpg --gen-key
    #列出密钥
    gpg --list-keys
    #签名密钥
    git config --global user.signingkey <key>
    ```

2. 签名（附注）标签  
    当对标签使用 `git show` 时会显示 gpg 签名

    ```shell
    # -s表示签名标签
    git tag -s <tag> -m <comment>
    ```

3. 验证标签  
    要求钥匙链中存储有签名者的公钥  

    ```shell
    git tag -v <tag>
    ```

4. 签名提交  

    ```shell
    git commit -a -S -m <comment>
    ```

### 搜索  

1. `git grep`  
    从提交历史、工作目录、甚至索引中查找一个字符串或者正则表达式  
    默认不带参数只会查找工作目录的文件  

    ```shell
    #--line-number：输出行数
    #--count：输出概述信息，仅包含匹配的文件以及文件中匹配数
    #--show-function：显示匹配字符串所在的函数
    #--and：多个字符串出现在同一行
    git grep [--line-number|-n] [--count|-c] [--show-function|-p] <keyword>
    ```

2. git 日志搜索  

    ```shell
    #显示修改内容包括该字符串的提交（源代码文件中的改动而非提交说明）
    git log -S <keyword>
    #显示代码中一行或一个函数的历史
    git log -L :<keyword>:<file>
    ```

### 重写历史(NFY)  

1. 修改最后一次提交  

    ```shell
    git commit --amend [-m <comment>|--noedit]
    ```

2. 修改多个提交信息  

### 重置  

1. 工作流程  
    `HEAD` 是当前分支引用的指针，它总是指向该分支上的最后一次提交  
    索引是预期的下一次提交，即 git 的暂存区  
    `HEAD` 和索引的内容存储在 .git 文件夹中，工作目录（或工作区）将其解包为实际的文件以供编辑  

    当初始化一个仓库时，`HEAD` 指向尚未创建的分支，索引为空，只有工作目录有内容  
    执行 `git add` 后，工作目录的内容被复制到索引中  
    执行 `git commit` 后，索引的内容被存储为快照，并创建指向快照的提交对象，以及更新分支指针  
    检出一个分支时，先修改 `HEAD` 使其指向新的分支引用，以新分支最后提交的快照填充索引，然后将索引的内容复制到工作目录  

2. 重置的作用  

    ```shell
    git reset [--soft|--mixed|--hard] <commit_reference>
    ```

    1. 回退分支指针  
        回退`HEAD`所指向的分支指针而非`HEAD`本身，使其指向指定的提交引用  
        若指定 `--soft`，则执行到这一步则停止  
    2. 更新索引  
        将当前分支指针指向的提交快照内容填充到索引  
        若指定 `--mixed`，则执行到这一步则停止  
        由于索引被覆盖，暂存的修改会被取消  
    3. 更新工作目录  
        将索引内容更新到工作目录  
        若指定 `--hard`，则执行到这一步  
        **会销毁掉未提交的修改且无法恢复**  
3. 部分重置  

```shell
git reset [<commit_reference>] <file> [--patch]
```

将指定的提交快照中的 `<file>` 内容填充到索引，不会应用到工作目录  
指定 `--patch` 可以选择 `<file>` 中的部分修改而非全部修改  
由于索引被覆盖，暂存的修改会被取消  

4. 重置与检出  
    - 不带路径的检出  
        `checkout` 对于工作目录是安全的，会检查以确保不会丢弃已更改的文件；`reset --hard`则会直接替换工作目录的文件  
        `checkout` 只会改变 `HEAD` 自身，而 `reset` 则是改变 `HEAD` 所指向的分支指针  
    - 带路径的检出  
        等价于 `git reset --hard [<branch>] <file>`  


### 高级合并(NFY)  

1. 中断合并    

    ```shell
    git merge --abort
    ```

    该命令会尝试恢复到合并前的状态  

2. 忽略空白  
    对于因修改空白字符（换行符、制表符、空格等）而产生的冲突，可以选择忽略空白修改  

    ```shell
    #将多个空白字符与单个空白字符视作等价
    git merge -Xignore-space-change <branch>
    #完全忽略空白修改
    git merge -Xignore-all-space <branch>
    ```

## 自定义 git(NFY)  

### 配置 git  

[简略版说明](#git%20配置)  

git 配置文件存储在三个位置，每一级的配置变量会覆盖上一级的配置变量  

| Position       | Description      |
| -------------- | ---------------- |
| /etc/gitconfig | 适用于系统上每一个用户的通用配置 |
| ~/.gitconfig   | 当前用户的配置          |
| .git/config    | 当前仓库的配置          |

可以直接编辑对应的配置文件，也可通过 `git config` 命令设定  

1. 客户端基本配置  
    - core.editor  
        默认情况下会调用环境变量 `$VISUAL` 或 `$EDITOR` 设置的文本编辑器，否则调用 `vi` 来创建和编辑提交以及标签信息  
        若设置了 `core.editor`，则会忽略其他设置，总是使用 `core.editor` 指定的文本编辑器  
    - commit.template  
        若设置了 `commit.template`，则 git 会使用指定文件的内容作为默认的提交注释  
    - core.pager  
        指定 `log` 和 `diff` 等命令时使用的分页器，默认为 `less`  
        若设置为空字符，则 git 会在一页内显示所有内容  
    - user.signingkey  
        详见[签名](#签名)  
        运行如下命令时会使用 `core.signingkey` 指定的密钥签名标签  

        ```shell
        # -s表示签名标签
        git tag -s <tag> -m <comment>
        ```

    - core.excludesfile  
        指定全局的 `.gitignore` 文件
    - help.autocorrect  
        接收一个单位为 0.1s 的整数，当输入错误的命令时，git 会尝试模糊匹配并在指定的时间后自动运行  
2. 着色  
    - color.ui  
        默认为 `auto`，自动着色大部分内容，当被重定向到管道或文件时，则忽略着色  
        设定为 `always` 时，在重定向到管道或文件中时也会着色输出  
        或者在重定向输出时指定 `--color`  
    - color.*  
        具体到哪些命令被着色以及如何着色，可选值为 `true`、`false`、`always`  
        `color.branch` `color.diff` `color.interactive` `color.status`  

        对于以上配置选项，其子选项用于设置如何着色  
        颜色可选项为 `normal`、`black`、`red`、`green`、`yellow`、`blue`、`magenta`、`cyan`、`white`  
        字体属性可选项为 `bold`、`dim`、`ul`、`blink`、`reverse`  
3. 外部合并与比较工具  
    默认情况下，git 传递给 `diff` 的参数如下  
    `path`、`old-file`、`old-hex`、`old-mode`、`new-file`、`new-hex`、`new-mode`  

    对于外部的合并与比较工具，可能需要脚本进行包装  

    - merge.tool  
        指定合并工具  
    - `merge.<tool>.cmd`  
        指定参数传递给外部工具的方式  
        `$BASE` 为一个临时文件的名称，包含合并的共同基础  
        `$LOCAL` 为一个临时文件的名称，包含当前分支上的文件内容  
        `$REMOTE` 为一个临时文件的名称，包含要合并的文件内容  
        `$MERGED` 为一个临时文件的名称，合并工具应该将合并结果写入  

        ```shell
        # 一个示例
        git config --global mergetool.<tool>.cmd '<tool> "$BASE" "$LOCAL" "$REMOTE" ""$MERGED'
        ```

    - `mergetool.<tool>.trustExitCode  [true|false]`  
        指定外部合并工具是否使用退出代码指示合并成功，否则 `git mergetool` 将在外部工具退出后表明解决成功  
4. 格式化多余空白字符  
    LF(Line Feed) 代表换行，即 `\n`，代表一行的结束  
    CR(Carriage Return) 代表回车，即 `\r`，代表将光标移动到当前行的开头  

    LF 和 CR 起源于打字机，完成一行后打字员需要手动将纸向上移动一段距离，即换行  
    同时需要重置打字机的**托架**(carriage)使下一个字符与纸张左侧对齐，即回车  

    MS-DOS 到Windows 使用 CRLF 代表行尾  
    电子文件没有打印机的物理限制，因而 Unix 系为了一致性和简单性使用 LF 代表行尾  

    - `core.autocrlf  [true|false|input]`
        设置为 `true` 时，在检出代码时，LF 会被转换为 CRLF，在提交代码时，CRLF 会被替换为 LF  
        设置为 `input` 时，在检出代码时不会执行操作，在提交时，CRLF 会被转换为 LF  
    - core.whitespace  
        可选项为  
        `blank-at-eol`：行尾空格  
        `blank-at-eof`：文件末尾空格  
        `space-before-tab`：行头tab之前的空格  
        `indent-with-non-tab`：空格而非tab开头的行  
        `tab-in-indent`：行头的tab  
        `cr-at-eol`：忽略行尾回车  

        可以一次列出多项以开启选项，以逗号分隔，前三个选项默认情况下开启  
        若要关闭选项，可以不列出或者在选项前增加 `-`  

        在执行 `git diff` 时会检测相关问题  
        在执行 `git apply` 和 `git rebase` 时可以通过指定 `--whitespace=warn` 在检测到问题时发出警告，或指定 `--whitespace=fix` 以自动修正  
5. 服务端配置  
    对于服务端，指定配置级别为 `--system`  
    - `receive.fsckObjects [true|false]`  
        设置为 `true` 时，git 在每次推送时都会检查每个对象的有效性和 SHA-1 校验和是否一致  
        该操作耗时较大，默认情况下为 `false`  
    - `receive.denyNonFastForwards [true|false]`  
        拒绝非快进式推送  
    - receive.denyDeletes  
        禁止通过推送删除分支和标签  

### git 属性(NFY)

针对于特定文件和路径的设定称作 git 属性，存储在工作目录 ==.gitattributes== 文件中

1. 二进制文件  
    1. 二进制文件识别  
        有些文件表面上是文本文件，但应该被当作二进制文件处理，则 git 不会处理 CRLF 问题，执行 `git show` 和 `git diff` 时也不会比较该文件的变化  

        在 ==.gitattributes== 文件中添加如下内容指示 git 将其视为二进制文件处理  

        ```shell
        <file> binary
        ```

    2. 二进制文件比较  
        一些特殊的二进制文件可以借助外部工具比较  
        以 word 文档为例，在 ==.gitattributes== 中添加如下入内以指示 docx 格式的比较过滤器  

        ```shell
        *.docx  diff=word
        ```

        然后通过如下命令指示 `word` 对应的外部工具  

        ```shell
        git config diff.word.textconv <program>
        ```

        对于部分图像文件，可以比较其 EXIF 信息  
2. 关键字展开(NFY)  
3. 导出版本库  
4. 合并策略  

### git 钩子  

git 钩子指能在特定的重要动作发生时触发的自定义脚本，分为客户端的和服务端的  
客户端钩子主要由提交、合并之类的动作触发  
服务端钩子主要由接收推送之类的动作触发  

1. 路径  
    钩子存储在 ==.git/hooks== 目录中，当初始化 git 仓库时该目录下会生成一些示例脚本  

    将一个正确命名且可执行的文件(任意编程语言)放入 ==.git/hooks== 目录即可激活该钩子  
2. 客户端钩子  
    客户端钩子不会随版本库被克隆  

    客户端钩子可分为提交工作流钩子、电子邮件工作流钩子和其它钩子  
    - 提交工作流钩子  
        - pre-commit  
            在输入提交注释前运行  

            用于检查即将提交的快照  

            如果以非0值退出，git 将放弃该次提交，除非指定了 `--no-verify`  

        - prepare-commit-msg  
            在准备输入提交注释信息，启动提交注释编辑器之前运行  

            可接收参数：存有当前提交信息的文件路径、提交类型、修补提交的提交的 SHA-1 校验  

            用于编辑默认提交注释；可与[提交注释模板](#配置%20git)结合使用  

        - commit-msg  
            在注释信息输入完成，准备提交之前运行  

            可接收参数：存有当前提交信息的文件路径  

            用于验证项目状态或提交信息  

            如果以非 0 值退出，git 将放弃该次提交  
        - post-commit  
            在提交流程完成后运行  

            用于通知  
    - 电子邮件工作流钩子  
        电子邮件工作流钩子由 `git am` 命令调用  
        - applypatch-msg  
            可接收参数：包含请求合并信息的临时文件的名字  
        - pre-applypatch  
        - posst-applypatch  
    - 其他客户端钩子  
        - pre-rebase  
        - post-rewrite  

### 强制策略

## git 内部原理

git 是一个**内容寻址**(content-addressable)文件系统，并在此之上提供了一个版本控制系统的用户界面  

### 底层命令与上层命令  

git 包含一部分用于完成底层工作的子命令，命令被设计成能以 UNIX 命令行风格连接在一起或被脚本调用，这部分命令被称作**底层**(plumbing)命令，更友好的命令被称为**上层**(procelain)命令  

```shell
#项目的git配置
config
#仅供gitweb使用
description
#指向被目前检出的分支
HEAD
#存储git hook脚本
hooks/
#存储全局性排除文件
info/
#存储数据内容
objects/
#存储指向数据的提交对象的指针
refs/
```

`git init` 初始化之后的 ==.git/== 目录如上所示，除此之外还有尚未创建的 ==index== 文件，保存暂存区信息  

### git 对象  

git 的核心部分是一个简单的键值对数据库，可以向 git 仓库插入任意类型的内容，其会返回一个唯一的键，通过该键可以取回内容  

对于 git 对象，git 存储其头部信息和原始数据  
头部信息首先是 git 识别出的类型：可以是 `blob`、`tree` 或 `commit`，然后是一个空格，再之后是原始数据的字节数，最后是一个空字节 `\0`  

```shell
#该命令用于取出文件内容并输出
# -p指示自动判断内容的类型，>指示输出至外部文件
git cat-file -p <file> [> <outputfile>]
```

1. **数据对象**(blob object)  

    ```shell
    #-w指示将该对象写入数据库中，--stdin指示从标准输入读取内容
    git hash-object -w <inputfile>|--stdin
    ```

    该命令会输出一个长度为 40 个字符的校验和，由待存储的数据和一个头部信息做 SHA-1 校验运算而得到  
    ==objects== 目录下会生成以校验和前两位命名的子目录以及其中以后 48 位命名的文件

2. **树对象**(tree object)  
    树对象对应工作目录的文件夹和文件，git 以类似于 UNIX 文件系统的方式存储内容，所有内容均以树对象和数据对象的形式存储，树对象对应 UNIX 中的目录项，数据对象大致对应文件内容  

    一个树对象包含了一条或多条**树记录**(tree entry)，每条记录含有一个指向数据对象或者子树对象的 SHA-1 指针以及相应的模式、类型、文件名信息  

    git 根据某一时刻暂存区所表示的状态记录创建并记录一个对应的树对象  

    ```shell
    #以下命令将文件加入暂存区
    #--add说明该文件此前不存在于暂存区
    #--cacheinfo指示将要添加的文件位于git数据库而非工作目录
    git update-index --add --cacheinfo <filemode> <SHA-1> <filename>
    ```

    对于数据对象，`<filemode>` 可选值为 100644-普通文件、100755-可执行文件、120000-符号链接  

    ```shell
    #以下命令将暂存区内容写入树对象
    #若<SHA-1>指定的树对象不存在则会自动创建一个新的树对象
    git write-tree <SHA-1>
    ```

3. **提交对象**(commit object)  
    提交对象包含一个树对象指针，代表快照；父提交指针；作者/提交者信息；提交注释  

    ```shell
    #-p指示其后的<SHA-1>为父提交
    git commit-tree <SHA-1> <comment> [-p <SHA-1>]
    ```

### git 引用

通过文件保存 SHA-1 值，通过文件的名字指针来替代原始的 SHA-1 值，称为**引用**(references，简写为 refs)，存储在 ==.git/refs/== 下  

```shell
#不推荐的用法
echo <SHA-1> > .git/refs/<file>
#git提供的命令
git update-ref refs/<file> <SHA-1>
```

当运行 `git branch` 之类的命令时，实际上会运行 `git update-ref`，取得提交对应的 SHA-1 值，用其更新分支的引用文件  

1. `HEAD` 引用  
    `HEAD` 文件是一个**符号引用**(symbolic reference)，即指向引用的指针，其指向目前所在的分支  

    ```shell
    #HEAD文件内容通常如下
    ref: refs/heads/master
    #若不指定分支引用，则仅会输出HEAD内容，否则改变HEAD指向
    #对于非法的<branch_ref_file>值，命令会报错
    git symbolic-ref HEAD [<branch_ref_file>]
    ```

2. **标签引用**(tag reference)  
    标签非常类似于提交对象，包含标签创建者信息、日期、注释信息以及指针，通常指向一个提交对象且不会改变指向  

    - 轻量标签  

        ```shell
        git update-ref refs/tags/<tag> <commit_SHA-1>
        ```

    - 附注标签  

        ```shell
        git tag -a <tag> <commit_SHA-1> -m <comment>
        ```

        对于附注标签，会创建一个标签对象，并使用引用来指向该标签对象而非直接指向提交对象  
3. **远程引用**(remote reference)  
    远程引用作为记录远程服务器上各分支最后已知位置状态的只读书签  
    添加远程版本库且执行过推送操作，git 会记录最后一次推送时每个分支多对应的值在 ==refs/remotes== 目录下  

### 包文件

初始情况下，对于同一份仅有细微差异的文件，git 在每次提交中都会完整保存一份快照，称为**松散**(loose)对象格式  

当满足以下三者之一时，git 会将这些文件打包成**包文件**(package)以及一个索引文件，即包文件中只完整保存一份，并记录其它版本的差异内容，索引文件记录包文件的偏移信息    
- 版本库中有太多松散对象  
- 向服务器执行推送  
- 手动执行 `git gc`  

git 打包时，会查找命名及大小相近的文件，并只保存文件不同版本之间的差异  
通常情况下，git 仅会保存最新版本的文件内容，而先前的版本通过差异方式保存  

### 引用规范  

添加远程仓库时会在本地 ==.git/config== 文件中添加如下内容，其包括远程版本库的名称、URL 以及用于 fetch 的**引用规范**(refspec)  

```shell
[remote "origin"]
    url = https://github.com/schacon/simplegit-progit
    fetch = +refs/heads/*:refs/remotes/origin/*
```

1. 格式  

    ```shell
    [+]<src>:<dst>
    ```

    其中，`+` 为可选项，表示即使不能快进的情况下强制更新引用，`<src>` 表示远程版本库中的引用，`<dst>` 是本地跟踪的远程引用的位置  
2. 引用规范推送  
    在上述引用规范位置可以指定push推送到的远程分支  

    ```shell
    push = <local_branch_ref>:<remote_branch_ref>
    ```

3. 删除引用  

    ```shell
    git push <remote> :<remote_branch>
    # 或
    git push <remote> --delete <remote_branch>
    ```

    将空值推送到远程分支，即删除该分支  

### 传输协议  

本章节概述两种基于 HTTP/S 协议的 git 仓库传输数据的方式  
其余协议见[协议](#协议)  

1. 哑协议 
    传输过程中，服务器不需要有 git 特有的代码，而是通过一系列 HTTP `GET` 请求，由客户端推断出服务端 git 仓库布局  

    一个典型的http-fetch流程如下：
    - 拉取 ==info/refs== 文件  
2. 智能协议
