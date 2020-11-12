### 什么是版本库
* 版本库又名仓库，英文名`repository`
    - 你可以简单理解成一个目录，这个目录里面的所有文件都可以被`Git`管理起来
    - 每个文件的修改、删除，`Git`都能跟踪
    - 以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”

### git 能做什么&需要注意的地方
* git 只能跟踪文本文件的改动（如TXT文件）
    - Microsoft 的 Word 格式是二进制格式，因此，版本控制系统是没法跟踪Word文件的改动的
* 图片、视频等二进制文件只能进行版本控制，但不能追踪其变化
* 文本编码强烈建议使用标准的UTF-8编码
    - 中文有常用的GBK编码，日文有Shift_JIS编码 不会被所有平台支持
    - Windows 自带的记事本编辑任何文本文件（保存为UTF-8），会在每个文件开头添加了0xefbbbf（十六进制）的字符

### 创建一个版本库
* 可以创建一个空目录，也可以在已有的目录进行创建 git 仓库
* 为了避免不必要的麻烦，请确保所要创建的仓库路径中全部为英文字母
#### 开始创建
```
mkdir blog
cd blog
git init
```
* 经过上述三个命令后，我们的 git 仓库就已经建立好了
* 现在当前目录下多了一个 .git 的目录
    - 不要试图手动更改 .git 的目录下的文件，这样会造成 git 仓库的损坏
* 如果你没有看到 .git 目录，则可以通过命令 `ls -ah` 查看
* 初始化仓库命令 `git init`
#### 添加文件
``` 
touch index.html
vim index.html
<span>hello git </span>
:wq
git add index.html
git commit -m 'hello git'
```
* 创建一个 `index.html` 文件
* 通过 vim 进行编辑
* 键入 `<span>hello git </span>`
* 保存 index.html 文件
* 通过 `git add` 命令添加 `index.html` 文件
* 通过 `git commit` 命令提交 `index.html` 文件
    - -m 后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。
继续编辑 index.html 文件
```
vim index.html
<span> goodbye git </span>
:wq
git add index.html
git status
git diff index.html
```
* `git status` 命令会提示我们 添加、更改、删除了哪些文件
    - [git status 命令详情请点击](gitstatus.md)
* `git diff` 命令可以查看我们更改了什么
    - `git diff` 顾名思义就是查看 `difference`
    - 显示的格式正是 `Unix` 通用的 `diff` 格式
### 本文所用 git 命令如下
* `git init` 初始化一个Git仓库
* `git add <file>` 可反复多次使用，添加多个文件
* `git commit -m` 向版本库进行提交
* `git status` 显示工作目录和暂存区的状态   
* `git diff` 显示提交和工作树等之间的更改