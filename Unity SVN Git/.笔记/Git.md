# 关于版本控制

## 本地版本控制系统

​	采用某种简单的数据库来记录文件的历次更新差异

​	如 RCS：工作原理是在硬盘上保存补丁集（补丁是指文件修订前后的变化）；通过应用所有的补丁，可以重新计算出各个版本的文件内容

![本地版本控制图解](https://git-scm.com/book/en/v2/images/local.png)



## 集中化的版本控制系统

集中化的版本控制系统（Centralized Version Control Systems，简称 CVCS）

如 SVN

单一的集中管理的服务器，保存所有文件的修订版本

客户端连到这台服务器，取出最新的文件或者提交更新

缺点是

​	1、中央服务器的单点故障，宕机客户端无法提交更新，

​	2、中心数据库所在的磁盘发生损坏，又没有做恰当备份，毫无疑问你将丢失所有数据

![集中化的版本控制图解](https://git-scm.com/book/en/v2/images/centralized.png)

## 分布式版本控制系统

分布式版本控制系统（Distributed Version Control System，简称 DVCS）

如 Git

客户端并不只提取最新版本的文件快照， 而是把代码仓库完整地镜像下来，包括完整的历史记录

优点：服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。每一次的克隆操作，实际上都是一次对代码仓库的完整备份。

![分布式版本控制图解](https://git-scm.com/book/en/v2/images/distributed.png)

# Git 是什么



## 直接记录快照，而非差异比较

SVN 其它大部分系统以文件变更列表的方式存储信息，基于差异（delta-based）的版本控制，将它们存储的信息看作是一组基本文件和每个文件随时间逐步累积的差异（存储每个文件与初始版本的差异）

![存储每个文件与初始版本的差异。](https://git-scm.com/book/en/v2/images/deltas.png)



Git 把数据看作是对 **小型文件系统** 的一系列快照 - 存储项目随时间改变的快照.（ Git 与几乎所有其它版本控制系统的重要区别）

​	每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引

​	为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。

Git 对待数据更像是一个 **快照流**。

![Git 存储项目随时间改变的快照。](https://git-scm.com/book/en/v2/images/snapshots.png)



## 近乎所有操作都是本地执行

不需要来自网络上其它计算机的信息 - 浏览项目的历史，Git 不需外连到服务器去获取历史 只需直接从本地数据库中读取

离线或者没有 VPN 时，几乎可以进行任何操作

在本地磁盘上就有项目的完整历史，所以大部分操作看起来瞬间完成。

SVN 能修改文件，但不能向数据库提交修改（因为你的本地数据库离线了）



## Git 保证完整性

Git 中所有的数据在存储前都计算校验和，然后以校验和来引用

Git 用以计算校验和的机制叫做 SHA-1 散列（hash，哈希）

```
24b9da6552252987aa493b52f8696cd6d3b00373
```

Git 数据库中保存的信息都是以文件内容的哈希值来索引，而不是文件名



## Git 一般只添加数据

Git 几乎不会执行任何可能导致文件不可恢复的操作

一旦你提交快照到 Git 中， 就难以再丢失数据，特别是定期的推送数据库到其它仓库



## 三种状态

**已提交（committed）**、**已修改（modified）** 和 **已暂存（staged）**

- 已修改表示修改了文件，但还没保存到数据库中。
- 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。- 还没有真正提交进git系统当中，把所有的改动都记录下来了，在所有的改动都暂存的情况下，git status是不会出现红色的提示的，只会有绿色的提示信息
- 已提交表示数据已经安全地保存在本地数据库中。- 只有通过命令git commit之后，**才算是真正把暂存区的代码提交了**



Git 项目拥有三个阶段：工作区、暂存区以及 Git 目录

![工作区、暂存区以及 Git 目录。](https://git-scm.com/book/en/v2/images/areas.png)

工作区是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。

暂存区是一个文件，保存了下次将要提交的文件列表信息，一般在 Git 仓库目录中。 按照 Git 的术语叫做“索引”，不过一般说法还是叫“暂存区”。

仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，复制的就是这里的数据。



基本的 Git 工作流程如下：

1. 在工作区中修改文件。
2. 将你想要下次提交的更改选择性地暂存，这样只会将更改的部分添加到暂存区。
3. 提交更新，找到暂存区的文件，将快照永久性存储到 Git 目录。

如果 Git 目录中保存着特定版本的文件，就属于 **已提交** 状态。 如果文件已修改并放入暂存区，就属于 **已暂存** 状态。 如果自上次检出后，作了修改但还没有放到暂存区域，就是 **已修改** 状态。 



# 起步 - 初次运行 Git 前的配置

定制你的 Git 环境

每台计算机上只需要配置一次

程序升级时会保留配置信息

 Git 自带一个 `git config` 的工具来帮助设置控制 Git 外观和行为的配置变量



三个 config 文件

在 Windows 系统中，Git 会查找 `$HOME` 目录下（一般情况下是 `C:\Users\$USER` ）的 `.gitconfig` 文件。 

Git 同样也会寻找 `安装 Git 时所选的目标位置/etc/gitconfig` 文件

系统级的配置文件： `C:\ProgramData\Git\config` 。此文件只能以管理员权限通过 `git config -f <file>` 来修改。



每一个级别会覆盖上一级别的配置，所以 `.git/config` 的配置变量会覆盖 `/etc/gitconfig` 中的配置变量。

```console
//查看所有的配置以及它们所在的文件
$ git config --list --show-origin

//设置你的用户名和邮件地址
//使用了 --global 选项，那么该命令只需要运行一次 之后无论你在该系统上做任何事情， Git 都会使用那些信息
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com

//针对特定项目使用不同的用户名称与邮件地址
//在那个项目目录下运行没有 --global 选项的命令来配置。
$ git config user.name "John Doe"
$ git config user.email johndoe@example.com

//配置默认文本编辑器
//使用不同的文本编辑器，例如 Notepad++ - 使用 32 位的 Windows 系统(64位的可能不支持)
$ git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"


//检查配置信息
//git config --list 命令来列出所有 Git 当时能找到的配置
$ git config --list
user.name=John Doe
user.email=johndoe@example.com
color.status=auto
color.branch=auto
color.interactive=auto
color.diff=auto
...


//输入 git config <key>： 来检查 Git 的某一项配置
$ git config user.name
John Doe
//查询 Git 中该变量的 原始 值 会告诉你哪一个配置文件最后设置了该值：
$ git config --show-origin rerere.autoUpdate
file:/home/johndoe/.gitconfig	false

//帮助 - 全面的手册
//到 Git 命令的综合手册
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
//获得 git config 命令的手册
$ git help config


//帮助 - 快速参考 如 add 的使用快速参考
$ git add -h
```



# Git 基础



## 获取 Git 仓库

两种获取 Git 项目仓库的方式：

1. 将尚未进行版本控制的本地目录转换为 Git 仓库；

   ```
   //在一个空文件夹中进行版本控制
   //首先需要进入该项目目录中
   $ cd /c/user/my_project
   //初始化 但 项目里的文件还没有被跟踪
   $ git init
   
   //在一个已存在文件的文件夹（而非空文件夹）中进行版本控制
   //开始追踪这些文件并进行初始提交
   //git add 命令使用文件或目录的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件
   $ git add *.c
   $ git add LICENSE
   $ git commit -m 'initial project version'
   ```

   

2. 从其它服务器 **克隆** 一个已存在的 Git 仓库。

Git 克隆的是该 Git 仓库服务器上的几乎所有数据，而不是仅仅复制完成你的工作所需要文件

远程 Git 仓库中的每一个文件的每一个版本都将被拉取下来

服务器的磁盘坏掉可以使用任何一个克隆下来的用户端来重建服务器上的仓库 （虽然可能会丢失某些服务器端的钩子（hook）设置，但是所有版本的数据仍在）

使用 `clone` 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写

```console
//git clone <url>
$ git clone https://github.com/libgit2/libgit2

//在克隆远程仓库的时候，自定义本地仓库的名字 额外的参数指定新的目录名
$ git clone https://github.com/libgit2/libgit2 mylibgit
```



Git 支持多种数据传输协议

`https://` 协议

`git://` 协议

SSH 传输协议，比如 `user@server:path/to/repo.git` 



## 记录每次更新到仓库

工作目录下的每一个文件都不外乎这两种状态：**已跟踪** 或 **未跟踪**



1、

已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后， 它们的状态可能是未修改，已修改或已放入暂存区

已跟踪的文件就是 Git 已经知道的文件



2、

未跟踪文件：既不存在于上次快照的记录中，也没有被放入暂存区



编辑过某些文件之后，由于自上次提交后你对它们做了修改，Git 将它们标记为已修改文件。 

在工作时，你可以选择性地将这些修改过的文件放入暂存区，然后提交所有已暂存的修改，如此反复。

![Git 下文件生命周期图。](https://git-scm.com/book/en/v2/images/lifecycle.png)



### 检查当前文件状态

```
//查看哪些文件处于什么状态 git status 
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working directory clean

//在项目下创建一个新的 README 文件
//新建的 README 文件出现在 Untracked files 下面 =  Git 在之前的快照（提交）中没有这些文件，Git 不会自动将之纳入跟踪范围
$ echo 'My Project' > README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

nothing added to commit but untracked files present (use "git add" to track)


```

### 跟踪新文件

```
//使用命令 git add 开始跟踪 README 文件
$ git add README

// README 文件已被跟踪，并处于暂存状态
//在 Changes to be committed 这行下面的，就说明是已暂存状态
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)

    new file:   README

```



### 暂存已修改的文件

```
//暂存已修改的文件 再运行 git status 命令
//Changes not staged for commit 这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区
//暂存这次更新，需要运行 git add 命令
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md

//git add 命令 是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。
//“精确地将内容添加到下一次提交中” 而不是 “将一个文件添加到项目中”
$ git add CONTRIBUTING.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
    
    
//对已经暂存的文件修改编辑后 git status 查看状态后 文件同时出现在暂存区和非暂存区
$ vim CONTRIBUTING.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md

//运行了 git add 之后又作了修订的文件，需要重新运行 git add 把最新版本重新暂存起来
$ git add CONTRIBUTING.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
    
    
```



### 状态简览

缩短状态命令的输出

 `git status -s` 命令或 `git status --short` 命令

```console
//?? = 新添加的未跟踪文件
//A  = 新添加到暂存区中的文件
//M  = 修改过的文件
//输出中有两栏，左栏指明了暂存区的状态，右栏指明了工作区的状态
//README 文件在工作区已修改但尚未暂存  lib/simplegit.rb 文件已修改且已暂存 Rakefile 文件已修改，暂存后又作了修改(文件的修改中既有已暂存的部分，又有未暂存的部分)
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```



### 忽略文件

些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表（如日志文件、临时文件）

`.gitignore` 的文件，列出要忽略的文件的模式

```console
$ cat .gitignore
*.[oa]		// 忽略所有以 .o 或 .a 结尾的文件
*~			// 忽略所有名字以波浪符（~）结尾的文件
```

一开始就为你的新仓库设置好 .gitignore 文件

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `#` 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
- 匹配模式可以以（`/`）开头防止递归。
- 匹配模式可以以（`/`）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反。



glob 模式是指 shell 所使用的简化了的正则表达式

```
星号（*）匹配零个或多个任意字符；
[abc] 匹配任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）； 
问号（?）只匹配一个任意字符；
如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。 
使用两个星号（）表示匹配任意中间目录，比如 a//z 可以匹配 a/z 、 a/b/z 或 a/b/c/z 等。


# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```



### 查看已暂存和未暂存的修改

`git diff` 命令：比较的是工作目录中当前文件和暂存区域快照之间的差异。 也就是修改之后还没有暂存起来的变化内容

 `git diff --staged` 命令：比对已暂存文件与最后一次提交的文件差异



### 提交更新

每次准备提交前，先用 `git status` 看下，你所需要的文件是不是都已暂存起来了， 然后再运行提交命令 `git commit`

执行后会启动选择的文本编辑器来输入提交说明



在 `commit` 命令后添加 `-m` 选项，将提交信息与命令放在同一行

```console
$ git commit -m "Story 182: Fix benchmarks for speed"

//在 master 提交 + 完整的 SHA-1 校验和是什么（463dc4f）
[master 463dc4f] Story 182: Fix benchmarks for speed、
//在本次提交中，有多少文件修订过，多少行添加和删改过
 2 files changed, 2 insertions(+)
 create mode 100644 README
```



**请记住，提交时记录的是放在暂存区域的快照**。 任何还未暂存文件的仍然保持已修改状态，可以在下次提交时纳入版本管理。 每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较。



### 跳过使用暂存区域

在提交的时候，给 `git commit` 加上 `-a` 选项

提交之前不再需要 `git add` 文件 了。  `-a` 选项使本次提交包含了所有修改过的文件

```console
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md

no changes added to commit (use "git add" and/or "git commit -a")
$ git commit -a -m 'added new benchmarks'
[master 83e38c7] added new benchmarks
 1 file changed, 5 insertions(+), 0 deletions(-)
```



### 移除文件

从 Git 中移除某个文件 = 从暂存区域移除 再 提交

`git rm` 命令，连带从工作目录中删除指定的文件，不会出现在未跟踪文件清单中

 如果要删除之前修改过或已经放到暂存区的文件，则必须使用强制删除选项 `-f`（译注：即 force 的首字母）



 `--cached` 选项：把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中

```console
//git rm 命令后面可以列出文件或者目录的名字
$ git rm --cached README

//可以使用 glob 模式 注意 星号 * 之前的反斜杠 \
$ git rm log/\*.log
```



### 移动文件

在 Git 中对文件改名

```console
$ git mv file_from file_to

$ git mv README.md README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
    
    
//git mv 就相当于运行了下面三条命令
$ mv README.md README
$ git rm README.md
$ git add README

```



## 查看提交历史

 `git log`  会按时间先后顺序列出所有的提交，最近的更新排在最上面

比较有用的选项是 `-p` 或 `--patch ` 会显示每次提交所引入的差异（按 **补丁** 的格式输出）

使用 `-2` 选项来只显示最近的两次提交

```console
$ git log -p -2
```

想看到每次提交的简略统计信息，可以使用 `--stat` 选项

```console
$ git log --stat
```

 `--pretty` 使用不同于默认格式的方式展示提交历史 `oneline` 会将每个提交放在一行显示， `short`，`full` 和 `fuller` 选项，它们展示信息的格式基本一致，但是详尽程度不一

```console
$ git log --pretty=oneline
ca82a6dff817ec66f44342007202690a93763949 changed the version number
085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test
a11bef06a3f659402fe7563abf99ad00de2209e6 first commit
```

 `format` ，可以定制记录的显示格式

```console
$ git log --pretty=format:"%h - %an, %ar : %s"
ca82a6d - Scott Chacon, 6 years ago : changed the version number
085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
a11bef0 - Scott Chacon, 6 years ago : first commit


//git log --pretty=format 常用的选项
%H		提交的完整哈希值
%h		提交的简写哈希值
%T		树的完整哈希值
%t		树的简写哈希值
%P		父提交的完整哈希值
%p		父提交的简写哈希值
%an		作者名字
%ae		作者的电子邮件地址
%ad		作者修订日期（可以用 --date=选项 来定制格式）
%ar		作者修订日期，按多久以前的方式显示
%cn		提交者的名字
%ce		提交者的电子邮件地址
%cd		提交日期
%cr		提交日期（距今多长时间）
%s		提交说明

```



 *作者* 和 *提交者* 之间的差别：

作者指的是实际作出修改的人

提交者指的是最后将此工作成果提交到仓库的人

当你为某个项目发布补丁，然后某个核心成员将你的补丁并入项目时，你就是作者，而那个核心成员就是提交者



当 `oneline` 或 `format` 与另一个 `log` 选项 `--graph` 结合使用时尤其有用

展示分支、合并历史

```console
$ git log --pretty=format:"%h %s" --graph
* 2d3acf9 ignore errors from SIGCHLD on trap
*  5e3ee11 Merge branch 'master' of git://github.com/dustin/grit
|\
| * 420eac9 Added a method for getting the current branch.
* | 30e367c timeout code and tests
* | 5a09431 add timeout protection to grit
* | e1193f8 support for heads with slashes in them
|/
* d6016bc require time for xmlschema
*  11d191e Merge branch 'defunkt' into local
```



```
//git log 的常用选项

-p
按补丁格式显示每个提交引入的差异。

--stat
显示每次提交的文件修改统计信息。

--shortstat
只显示 --stat 中最后的行数修改添加移除统计。

--name-only
仅在提交信息后显示已修改的文件清单。

--name-status
显示新增、修改、删除的文件清单。

--abbrev-commit
仅显示 SHA-1 校验和所有 40 个字符中的前几个字符。

--relative-date
使用较短的相对时间而不是完整格式显示日期（比如“2 weeks ago”）。

--graph
在日志旁以 ASCII 图形显示分支与合并历史。

--pretty
使用其他格式显示历史提交信息。可用的选项包括 oneline、short、full、fuller 和 format（用来定义自己的格式）。

--oneline
--pretty=oneline --abbrev-commit 合用的简写。
```



`--since` 和 `--until` 这种按照时间作限制的选项

列出最近两周的所有提交：

```console
$ git log --since=2.weeks
```

过滤出匹配指定条件的提交。 用 `--author` 选项显示指定作者的提交，用 `--grep` 选项搜索提交说明中的关键字。

过滤器 `-S`

```console
//找出添加或删除了对某一个特定函数的引用的提交
$ git log -S function_name
```



路径（path）：只关心某些文件或者目录的历史提交

用两个短划线（--）隔开之前的选项和后面限定的路径名

```
//限制 git log 输出的选项
-<n>
仅显示最近的 n 条提交。

--since, --after
仅显示指定时间之后的提交。

--until, --before
仅显示指定时间之前的提交。

--author
仅显示作者匹配指定字符串的提交。

--committer
仅显示提交者匹配指定字符串的提交。

--grep
仅显示提交说明中包含指定字符串的提交。

-S
仅显示添加或删除内容匹配指定字符串的提交

```



在 Git 源码库中查看 Junio Hamano 在 2008 年 10 月其间， 除了合并提交之外的哪一个提交修改了测试文件，可以使用下面的命令：

```console
// 避免显示的合并提交弄乱历史记录，可以为 log 加上 --no-merges 选项。
$ git log --pretty="%h - %s" --author='Junio C Hamano' --since="2008-10-01" \
   --before="2008-11-01" --no-merges -- t/
5610e3b - Fix testcase failure when extended attributes are in use
acd3b9e - Enhance hold_lock_file_for_{update,append}() API
f563754 - demonstrate breakage of detached checkout with symbolic link HEAD
d1a43f2 - reset --hard/read-tree --reset -u: remove unmerged new paths
51a94af - Fix "checkout --track -b newbranch" on detached HEAD
b0ad11e - pull: allow "git pull origin $something:$current_branch" into an unborn branch
```



## 撤消操作

注意，有些撤消操作是不可逆的



### 重新提交

比如 有一次提交，然后提交之后评审发现代码有问题，我们没有进行和入，需要重新修改，但是我们又不能产生新的commit

提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了：`--amend` 选项的提交命令来重新提交

```console
$ git commit --amend
```

这个命令会将暂存区中的文件提交。 

如果自上次提交以来你还未做任何修改（例如，在上次提交后马上执行了此命令）， 那么快照会保持不变，而你所修改的只是提交信息。

文本编辑器启动后，可以看到之前的提交信息。 编辑后保存会覆盖原来的提交信息。



提交后发现忘记了暂存某些需要的修改：

最终你只会有一个提交——第二次提交将代替第一次提交的结果，旧有的提交从未存在过一样，它并不会出现在仓库的历史中

```console
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```



### 取消暂存的文件

使用 `git reset HEAD <file>…` 来取消暂存

```console
$ git add *
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
    modified:   CONTRIBUTING.md
```

`git reset` 确实是个危险的命令，如果加上了 `--hard` 选项则更是如此

```console
// 取消暂存 CONTRIBUTING.md 文件
$ git reset HEAD CONTRIBUTING.md
Unstaged changes after reset:
M	CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```



### 撤消对文件的修改

将修改的文件还原成上次提交时的样子

一个危险的命令

```console
git checkout -- <file>...
```

```console
$ git checkout -- CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```



## 远程仓库的使用

### 查看远程仓库

 `git remote` 命令：列出你指定的每一个远程服务器的简写

 origin ——这是 Git 给你克隆的仓库服务器的默认名字



指定选项 `-v`，会显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL

```console
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
```



### 添加远程仓库

运行 `git remote add <shortname> <url>` 添加一个新的远程 Git 仓库，同时指定一个方便使用的简写

简写 可以用来代替整个 URL

```console
$ git remote
origin
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)

//使用简写 来拉取
$ git fetch pb
remote: Counting objects: 43, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 43 (delta 10), reused 31 (delta 5)
Unpacking objects: 100% (43/43), done.
From https://github.com/paulboone/ticgit
 * [new branch]      master     -> pb/master
 * [new branch]      ticgit     -> pb/ticgit
```



### 从远程仓库中抓取与拉取

从远程仓库中拉取所有你还没有的数据

执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。

```console
$ git fetch <remote>
```

`git fetch origin` 会抓取克隆（或上一次抓取）后新推送的所有工作。 必须注意 `git fetch` 命令只会将数据下载到你的本地仓库——它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。

如果你的当前分支设置了跟踪远程分支， 那么可以用 `git pull` 命令来自动抓取后合并该远程分支到当前分支。默认情况下，`git clone` 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 `master` 分支（或其它名字的默认分支）。 运行 `git pull` 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。



### 推送到远程仓库

`git push <remote> <branch>`

将 `master` 分支推送到 `origin` 服务器时

```console
$ git push origin master
```

只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效

当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 

你必须先抓取他们的工作并将其合并进你的工作后才能推送。



### 查看某个远程仓库

git remote show <remote>

列出远程仓库的 URL 与跟踪分支的信息

```
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/schacon/ticgit
  Push  URL: https://github.com/schacon/ticgit
  //正处于 master 分支
  HEAD branch: master
  Remote branches:
    master                               tracked
    dev-branch                           tracked
   //如果运行 git pull， 就会抓取所有的远程引用
  Local branch configured for 'git pull':
    //然后将远程 master 分支合并到本地 master 分支。 它也会列出拉取到的所有远程引用。
    master merges with remote master
   //当你在特定的分支上执行 git push 会自动地推送到哪一个远程分支
  Local ref configured for 'git push':
    master pushes to master (up to date)
```



### 远程仓库的重命名与移除

修改一个远程仓库的简写名

```
git remote rename
```

将 `pb` 重命名为 `paul`

```console
$ git remote rename pb paul
$ git remote
origin
paul
```

会修改你所有远程跟踪的分支名字



想要移除一个远程仓库

```
git remote remove 或 git remote rm
```

```console
$ git remote remove paul
$ git remote
origin
```

所有和这个远程仓库相关的远程跟踪分支以及配置信息也会一起被删除



## 打标签

给仓库历史中的某一个提交打上标签，以示重要

标记发布结点（ `v1.0` 、 `v2.0` 等等）



### 列出标签

列出已有的标签

 `git tag` （可带上可选的 `-l` 选项 `--list`）

```console
$ git tag
v1.0
v2.0
```



按照特定的模式查找标签

```console
$ git tag -l "v1.8.5*"
v1.8.5
v1.8.5-rc0
v1.8.5-rc1
v1.8.5-rc2
v1.8.5-rc3
v1.8.5.1
v1.8.5.2
v1.8.5.3
v1.8.5.4
v1.8.5.5
```



### 创建标签

轻量标签（lightweight）与附注标签（annotated）

轻量标签很像一个不会改变的分支——它只是某个特定提交的引用。用在 只是想用一个临时的标签， 或者因为某些原因不想要保存这些信息

附注标签是存储在 Git 数据库中的一个完整对象，可以被校验的，其中包含打标签者的名字、电子邮件地址、日期时间， 此外还有一个标签信息



### 附注标签

创建附注标签

在运行 `tag` 命令时指定 `-a` 选项

`-m` 选项指定了一条将会存储在标签中的信息

```console
$ git tag -a v1.4 -m "my version 1.4"
$ git tag
v0.1
v1.3
v1.4
```



 `git show` 命令可以看到标签信息和与之对应的提交信息

```console
$ git show v1.4
tag v1.4
Tagger: Ben Straub <ben@straub.cc>
Date:   Sat May 3 20:19:12 2014 -0700

my version 1.4

commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```



### 轻量标签

本质上是将提交校验和存储到一个文件中——没有保存任何其他信息

 创建轻量标签，只需要提供标签名字

```console
$ git tag v1.4-lw
$ git tag
v0.1
v1.3
v1.4
v1.4-lw
v1.5
```

在标签上运行 `git show`只会显示出提交信息

```console
$ git show v1.4-lw
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```



### 后期打标签

对过去的提交打标签

要在那个提交上打标签，你需要在命令的末尾指定提交的校验和（或部分校验和）

```console
$ git log --pretty=oneline
15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function
4682c3261057305bdd616e23b64b0857d832627b added a todo file
166ae0c4d3f420721acbb115cc33848dfcc2121a started write support
9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo
8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme

$ git tag -a v1.2 9fceb02

$ git tag
v0.1
v1.2
v1.3
v1.4
v1.4-lw
v1.5

$ git show v1.2
tag v1.2
Tagger: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Feb 9 15:32:16 2009 -0800

version 1.2
commit 9fceb02d0ae598e95dc970b74767f19372d61af8
Author: Magnus Chacon <mchacon@gee-mail.com>
Date:   Sun Apr 27 20:43:35 2008 -0700

    updated rakefile
...
```



### 共享标签

默认情况下，`git push` 命令并不会传送标签到远程仓库服务器上

显式地推送标签到共享服务器上： `git push origin <tagname>`

```console
$ git push origin v1.5
Counting objects: 14, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (12/12), done.
Writing objects: 100% (14/14), 2.05 KiB | 0 bytes/s, done.
Total 14 (delta 3), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.5 -> v1.5
```



 `--tags` 选项的 `git push` 命令：一次性推送很多标签（轻量标签 和 附注标签）

把所有不在远程仓库服务器上的标签全部传送到那里，让其他人从仓库中克隆（clone）或拉取（fetch），他们也能得到你的那些标签

```console
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 160 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.4 -> v1.4
 * [new tag]         v1.4-lw -> v1.4-lw
```



### 删除标签

 `git tag -d <tagname>`

```
//删除一个轻量标签
$ git tag -d v1.4-lw
Deleted tag 'v1.4-lw' (was e7d5add)
```

不会从任何远程仓库中移除这个标签，`git push <remote> :refs/tags/<tagname>` 来更新你的远程仓库



变体一： `git push <remote> :refs/tags/<tagname>` 

将冒号前面的空值推送到远程标签名，从而高效地删除它

```console
$ git push origin :refs/tags/v1.4-lw
To /git@github.com:schacon/simplegit.git
 - [deleted]         v1.4-lw
```



变体二：更直观

```console
$ git push origin --delete <tagname>
```



### 检出标签

 `git checkout` 命令：查看某个标签所指向的文件版本

注意：这样会使你的仓库处于“分离头指针（detached HEAD）”的状态

在“分离头指针”状态下，如果你做了某些更改然后提交它们，标签不会发生变化， 但你的新提交将不属于任何分支，并且将无法访问，除非通过确切的提交哈希才能访问。

因此，如果你需要进行更改，比如你要修复旧版本中的错误，那么通常需要创建一个新分支：

```console
$ git checkout -b version2 v2.0.0
Switched to a new branch 'version2'
```

如果在这之后又进行了一次提交，`version2` 分支就会因为这个改动向前移动， 此时它就会和 `v2.0.0` 标签稍微有些不同，这时就要当心了。



## Git 别名

如果不想每次都输入完整的 Git 命令，可以通过 `git config` 文件来轻松地为每一个命令设置一个别名

```console
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status

//这意味着，当要输入 git commit 时，只需要输入 git ci。

```

取消暂存别名

```console
$ git config --global alias.unstage 'reset HEAD --'

//会使下面的两个命令等价
$ git unstage fileA
$ git reset HEAD -- fileA

//last 命令看起来更清楚一些
$ git config --global alias.last 'log -1 HEAD'
//使用别名后 可以轻松地看到最后一次提交
$ git last
commit 66938dae3329c7aebe598c2246a8e6af90d04646
Author: Josh Goebel <dreamer3@example.com>
Date:   Tue Aug 26 19:48:51 2008 +0800

    test for current head

    Signed-off-by: Scott Chacon <schacon@example.com>
```



在命令前面加入 `!` 符号：执行外部命令

写一些与 Git 仓库协作的工具

```console
//将 git visual 定义为 gitk 的别名
$ git config --global alias.visual '!gitk'
```



# Git 分支

使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线（**影响开发主线才需要使用分支**）

Git 的分支模型称为它的“必杀技特性”：难以置信的轻量，瞬间创建新分支，在不同分支之间的便捷切换

**Git 鼓励在工作流程中频繁地使用分支与合并，哪怕一天之内进行许多次**



Git 是如何保存数据：

Git 保存的不是文件的变化或者差异，而是一系列不同时刻的 **快照** 

在进行提交操作时，Git 会保存一个**提交对象（commit object）**，

提交对象：包含一个指向暂存内容快照的指针，还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。

首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象， 而由多个分支合并产生的提交对象有多个父对象



假设现在有一个工作目录，里面包含了三个将要被暂存和提交的文件：

当使用 `git commit` 进行提交操作时，**Git 会先计算每一个子目录的校验和， 然后在 Git 仓库中这些校验和保存为树对象。**随后，Git 便会创建一个提交对象， 它除了包含上面提到的那些信息外，还包含指向这个树对象（项目根目录）的指针。 如此一来，Git 就可以在需要的时候重现此次保存的快照。

1、首次提交对象及其树结构，Git 仓库中有五个对象：

​	三个 *blob* 对象（保存着文件快照）、

​	一个 **树** 对象 （记录着目录结构和 blob 对象索引）

​	一个 **提交** 对象（包含着指向前述树对象的指针和所有提交信息）

![首次提交对象及其树结构。](https://git-scm.com/book/en/v2/images/commit-and-tree.png)

2、做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针

![提交对象及其父对象。](https://git-scm.com/book/en/v2/images/commits-and-parents.png)



**Git 的分支，其实本质上仅仅是指向提交对象的可变指针。** 

Git 的默认分支名字是 `master`。 在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 `master` 分支。 `master` 分支会在每次提交时自动向前移动。

![分支及其提交历史。](https://git-scm.com/book/en/v2/images/branch-and-history.png)



## 分支创建

只是为你创建了一个可以移动的新的指针

```console
//创建一个 testing 分支 
//会在当前所在的提交对象上创建一个指针
$ git branch testing
```

![两个指向相同提交历史的分支。](https://git-scm.com/book/en/v2/images/two-branches.png)

 `HEAD` 的特殊指针： 知道当前在哪一个分支上

​		在 Git 中， `HEAD` 是一个指针，将 `HEAD` 想象为当前分支的别名，指向当前所在的本地分支



 `git branch` 命令仅仅 **创建** 一个新分支，并不会自动切换到新分支中去

![HEAD 指向当前所在的分支。](https://git-scm.com/book/en/v2/images/head-to-master.png)

使用 `git log` 命令查看各个分支当前所指的对象。 提供这一功能的参数是 `--decorate`

```console
$ git log --oneline --decorate
// 当前 master 和 testing 分支均指向校验和以 f30ab 开头的提交对象
f30ab (HEAD -> master, testing) add feature #32 - ability to add new formats to the central interface
34ac2 Fixed bug #1328 - stack overflow under certain conditions
98ca9 The initial commit of my project
```



## 分支切换

`git checkout` 命令

```console
//让 HEAD 指向 testing 分支
$ git checkout testing
```

![HEAD 指向当前所在的分支。](https://git-scm.com/book/en/v2/images/head-to-testing.png)



 HEAD 分支随着提交操作自动向前移动

![HEAD 分支随着提交操作自动向前移动。](https://git-scm.com/book/en/v2/images/advance-testing.png)

```console
//检出时 HEAD 随之移动
$ git checkout master
```

检出时做了两件事：

一是使 HEAD 指回 `master` 分支，二是将工作目录恢复成 `master` 分支所指向的快照内容



![检出时 HEAD 随之移动。](https://git-scm.com/book/en/v2/images/checkout-master.png)





切换分之后进行提交，会产生项目分叉历史

![项目分叉历史。](https://git-scm.com/book/en/v2/images/advance-master.png)

使用 `git log` 命令查看分叉历史

运行 `git log --oneline --decorate --graph --all` ，它会输出你的提交历史、各个分支的指向以及项目的分支分叉情况

```console
$ git log --oneline --decorate --graph --all
* c2b9e (HEAD, master) made other changes
| * 87ab2 (testing) made a change
|/
* f30ab add feature #32 - ability to add new formats to the
* 34ac2 fixed bug #1328 - stack overflow under certain conditions
* 98ca9 initial commit of my project
```



**过去大多数版本控制系统：在创建分支时，将所有的项目文件都复制一遍，并保存到一个特定的目录**



**Git 的分支实质上仅是包含所指对象校验和（长度为 40 的 SHA-1 值字符串）的文件，所以它的创建和销毁都异常高效。 **

**创建一个新分支就相当于往一个文件中写入 41 个字节（40 个字符和 1 个换行符）**



`git checkout -b <newbranchname>` ：创建新分支的同时切换过去



## 分支新建与合并

**使用场景：为紧急任务新建一个分支，并在其中修复它**



### 新建分支（指针）

![一个简单的提交历史。](https://git-scm.com/book/en/v2/images/basic-branching-1.png)



如：要解决你的公司使用的问题追踪系统中的 #53 问题，

新建一个分支并同时切换到那个分支上

运行一个带有 `-b` 参数的 `git checkout` 命令：

```console
$ git checkout -b iss53
Switched to a new branch "iss53"
```

简写：

```console
$ git branch iss53
$ git checkout iss53
```

![创建一个新分支指针。](https://git-scm.com/book/en/v2/images/basic-branching-2.png)

 `iss53` 分支随着工作的进展向前推进

![`iss53` 分支随着工作的进展向前推进。](https://git-scm.com/book/en/v2/images/basic-branching-3.png)



切换回 `master` 分支之前，要留意你的工作目录和暂存区里那些还没有被提交的修改， 它可能会和你即将检出的分支产生冲突从而阻止 Git 切换到该分支

最好的方法是，**在你切换分支之前，保持好一个干净的状态**。 有一些方法可以绕过这个问题（即，暂存（stashing） 和 修补提交（commit amending））

```console
$ git checkout master
Switched to branch 'master'
```



请牢记：当你切换分支的时候，Git 会重置你的工作目录，使其看起来像回到了你在那个分支上最后一次提交的样子。 Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样。













