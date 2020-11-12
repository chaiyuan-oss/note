# git status命令
## 简介
* `git status` 命令用于显示工作目录和暂存区的状态
* 使用此命令能看到那些修改被暂存到了, 哪些没有, 哪些文件没有被Git tracked到
* `git status` 不显示已经 `commit` 到项目历史中去的信息
* 看项目历史的信息要使用 `git log`
```git
git status [<options>…​] [--] [<pathspec>…​]
```
## 描述
显示索引文件和当前HEAD提交之间的差异，在工作树和索引文件之间有差异的路径以及工作树中没有被Git跟踪的路径。
* 第一个是通过运行 `git commit` 来提交的
* 第二个和第三个是你可以通过在运行 `git commit` 之前运行 `git add` 来提交的

git status相对来说是一个简单的命令，它简单的展示状态信息。输出的内容分为3个分类/组。
```git
# On branch master
# Changes to be committed:  (已经在stage区, 等待添加到HEAD中的文件)
# (use "git reset HEAD <file>..." to unstage)
#
#modified: index.html
#
# Changes not staged for commit: (有修改, 但是没有被添加到stage区的文件)
# (use "git add <file>..." to update what will be committed)
# (use "git checkout -- <file>..." to discard changes in working directory)
#
#modified: main.html
#
# Untracked files:(没有tracked过的文件, 即从没有add过的文件)
# (use "git add <file>..." to include in what will be committed)
#
```
### 忽略文件(untracked文件)
- 没有 `tracked` 的文件分为两类
    - 一是已经被放在工作目录下但是还没有执行 `git add` 的
    - 另一类是一些编译了的程序文件(如`.pyc`, `.obj`, `.exe`等)
    - 当这些不想 `add` 的文件一多起来, `git status` 的输出简直没法看
    - 基于这个原因。 `Git` 让我们能在一个特殊的文件 `.gitignore` 中把要忽略的文件放在其中
    - 每一个想忽略的文件应该独占一行, `*`这个符号可以作为通配符使用
    - 例如在项目根目录下的 `.gitignore` 文件中加入下面内容能阻止 `.pyc` 和 `.tmp` 文件出现在 `git status` 中
    ```
    *.pyc
    *.tmp
    ```
- 通过 `git status -uno` 可以只列出所有已经被git管理的且被修改但没提交的文件。
- 一个好的习惯就是每次提交前运行一次 `git status` 命令，这样会避免一些不必要发生的问题