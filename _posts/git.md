# 本地文件
仓库repository
不同机器通过仓库进行沟通
仓库下有不同分支（branch） 不同branch下的文件content可以不同
对location的文件content进行修改，可以commit到本地的repository
(commit相当于存档)
不同branch之间可以合并merger
可以把branch同步到其他机器（push）
1. 为一个文件夹下的内容提供版本管理，动作是新建一个repository
# Initialize the local Git repository
git init
# Add all (files and directories) to the Git repository
git add .

>其实 git add 的潜台词就是把目标文件快照放入暂存区域，也就是 add file into staged area，同时未曾跟踪过的文件标记为需要跟踪。这样就好理解后续 add 操作的实际意义了

# Make a commit of your file to the local repository
git commit -m "Initial commit"
# Show the log file
git log

常用这个：git log --graph --oneline

查看未来的 

git reflog --oneline 
#查看修改

###查看某一次commit修改了什么文件

	git show --pretty="format:" --name-only 0429aba
	
###查看某一次commit的具体修改

	git show 0429aba

##查看自上一次commit后的修改
git status（工作区和版本库的区别）

>Displays paths that have differences between the index file and the current HEAD commit, paths that have differences between the working tree and the index file, and paths in the working tree that are not tracked by Git (and are not ignored by gitignore(5)). The first are what you would commit by running git commit; the second and third are what you could commit by running git add before running git commit.


##查看某文件具体被修改什么
###查看文件工作区和上次commit的区别
git diff readme.txt (查看工作区和版本库的区别)
###查看两个commit之间同一个文件的区别
 git diff $start_commit..$end_commit -- path/to/file

对于git add，其实是把**修改**放到暂存区，而git commit是把暂存区的，提交到仓库，回滚，是回滚仓库的。具体看

[工作区和暂存区](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013745374151782eb658c5a5ca454eaa451661275886c6000)

[管理修改](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374829472990293f16b45df14f35b94b3e8a026220c5000)

###两个branch的同一个文件区别


git diff can show you the difference between two commits:

git diff mybranch master -- myfile.cs


2. 新建branch
git branch branchname

3. 切换branch(切换branch后，location的内容就会是切换后branch的内容)
git checkout branch


通过git log 可以看到当前branch的操作，git status git diff可以看到自上次提交后的修改

我们将创建一个远端的Git仓库。这个仓库可以存储在本地或者是网络上。

远端Git仓库和标准的Git仓库有如下差别：一个标准的Git仓库包括了源代码和历史信息记录。我们可以直接在这个基础上修改代码，因为它已经包含了一个工作副本。但是远端仓库没有包括工作副本，只包括了历史信息。
除了通过完整的URL来访问Git仓库外，还可以通过git remote add命令为仓库添加一个短名称。当你克隆了一个仓库以后，origin表示所克隆的原始仓库。即使我们从零开始，这个名称也存在。 

#还原

常用 git reset --hard c63b6
git checkout commit_name  git revert commit_name  git reset
git checkout c63b6 (5位就可以了)

The git index defined in 2 sentences
The git “index” is where you place files you want committed to the git repository.
Before you “commit” (checkin) files to the git repository, you need to first place the files in the git “index”.
An index by any other name
The git index goes by many names. But they all refer to the same thing. Some of the names you may have heard:
Index
Cache
Directory cache
Current directory cache
Staging area
Staged files

git add xxx
首先要知道有一个object database或叫object store的东西，就是用来存储git add 的对象副本
这个指令是：当你在工作目录做出修改后，修改是没有提交到object database的，这个指令就是用来提交的
然后用git commit 指令  可以为object database最近一次提交创建一个树对象，并与这个commit关联起来，这个树对象就是用来版本管理的
关于index git add  git commit详细见这几个东西  看http://www.gitguys.com/topics/whats-the-deal-with-the-git-index/

###当文件版本修改

	git checkout 0429aba paht/to/file

##删除原来tracked的
（后来添加上.gitignore的）
git rm --cached filename
注意，这个push后，服务器上相对应的文件会被删掉

###删除branch

##关于ignore

gitignore - Specifies intentionally **untracked** files to ignore

>The purpose of gitignore files is to ensure that certain files not tracked by Git remain untracked.

>To ignore uncommitted changes in a file that is already tracked, use git update-index {litdd}assume-unchanged.

>To stop tracking a file that is currently tracked, use git rm --cached.

ignore只保证原来没有track的继续untrack，但是已经track的，后来加入.gitignore的，就无能为力了，只能用`git update-index`或`git rm --cached`

###context
一开始创建git的.gitignore文件时，有些文件忘记.gitignore
如果简单的在.gitignore文件中加上这个文件，其实是没有效果的！
需要`git rm --cached FILE_NAME`,但是如果只在本机`git rm --cached FILE_NAME`,push上服务器，其他机器在pull时，会发现这个文件在本地被！删！除。所以必须在所有开发机`git rm --cached FILE_NAME`

见下面：


Solution

1. Add the filename to your .gitignore file (if you don’t have one, create it). This will make sure git ignores changes to this file(s) for all future commits.

2. Remove the HISTORY of the config file(s) not the file itself. This causes git to have amnesia about the existence of your config file(s)


    	echo "FILE_NAME" >> .gitignore
    	git rm --cached FILE_NAME
    	git add -u
    	git commit -m "removing files from version control"
    
    	git pull
    	git push

3. Repeat these steps on all your machines.

重点是，每台机器在git pull之前都要

    	echo "FILE_NAME" >> .gitignore
    	git rm --cached FILE_NAME
    	git add -u
    	git commit -m "removing files from version control"

##添加到远程仓库
git remote add origin git@server-name:path/repo-name.git

###设置本地和远程的分支相关联
设置dev和origin/dev的链接：

$ git branch --set-upstream dev origin/dev
Branch dev set up to track remote branch dev from origin.

##查看
###List files in local git repo
git ls-tree --full-tree -r HEAD


查看ignore哪些文件
git ls-files --others --exclude-from=.git/info/exclude

查看哪些是tracked的

If you want to list all the files currently being tracked under the branch master, you could use this command:

    git ls-tree -r master --name-only
If you want a list of files that ever existed (i.e. including deleted files):

    git log --pretty=format: --name-only --diff-filter=A | sort - | sed '/^$/d'


###关于track
http://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository


###对比两个不同
1. 对比两个commit的不同：git diff hashcode1 hashcode2 如：git diff 

###子模块
git submodule

https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97