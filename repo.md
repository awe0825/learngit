#####repo xml文件
Android使用repo来管理多个git项目。它需要一个manifest XML文件来指示这些git项目的属性。
*****
在本地服务器创建一个manifest的文件夹
通过git clone到该文件夹中
*****

EXAMPLE:
(```)
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
	<remote fetch="ssh://git.example.com"(从哪里去下载代码) name="test" review="gerrit.example.com"（Gerrit/Git服务器的地址）/>
	<defaut remote="test"(需要和remote中的name一致) revision="master" syn-j="1"/>
	<project path="project/name" name="name" revision="branches"/>
</manifest>
(```)

######xml文件包括下面的元素：
**remote标签**
	name：在每一个.git/config文件的remote项中用到这个name，即表示每个git的远程服务器的名字(这个名字很关键，如果多个remote属性的话，default属性中需要指定default remote)。git pull、get fetch的时候会用到这个remote name。
	alias ：可以覆盖之前定义的remote name，name必须是固定的，但是alias可以不同，可以用来指向不同的remote url
	fetch ：所有git url真正路径的前缀，所有git 的project name加上这个前缀，就是git url的真正路径
	review ：指定Gerrit的服务器名，用于repo upload操作。如果没有指定，则repo upload没有效果
	
**default标签**
	remote ：远程服务器的名字（上面remote属性中提到过，多个remote的时候需要指定default remote，就是这里设置了）
	revision ：所有git的默认branch，后面project没有特殊指出revision的话，就用这个branch
	sync_j ： 在repo sync中默认并行的数目
	sync_c ：如果设置为true，则只同步指定的分支(revision 属性指定)，而不是所有的ref内容
	sync_s ： 如果设置为true，则会同步git的子项目

**project标签**
	name ：git 的名称，用于生成git url。URL格式是：${remote fetch}/${project name}.git 其中的 fetch就是上面提到的remote 中的fetch元素，name 就是此处的name
	path ：clone到本地的git的工作目录，如果没有配置的话，跟name一样
	remote ：定义remote name，如果没有定义的话就用default中定义的remote name
	revision ：指定需要获取的git提交点，可以定义成固定的branch，或者是明确的commit 哈希值
	groups ：列出project所属的组，以空格或者逗号分隔多个组名。所有的project都自动属于"all"组。每一个project自动属于name:'name' 和path:'path'组。例如，它自动属于default, name:monkeys, and path:barrel-of组。如果一个project属于notdefault组，则，repo sync时不会下载
	sync_c ：如果设置为true，则只同步指定的分支(revision 属性指定)，而不是所有的ref内容。
	sync_s ： 如果设置为true，则会同步git的子项目
	upstream ：在哪个git分支可以找到一个SHA1。用于同步revision锁定的manifest(-c 模式)。该模式可以避免同步整个ref空间
	annotation ：可以有0个或多个annotation，格式是name-value，repo forall命令是会用来定义环境变量
	
#####本地repo init，代码仓库的初始化
创建一个文件夹并进入
repo init -u ssh://username@服务器：端口号/manifest -b BranchName -m xmlFile进行代码仓库的初始化
执行repo sync（-c -j16） 进行代码同步
//-c表示当前分支，-j（1,2,4,8,16）表示线程。
在完成repo sync的动作后一定需要进行 repo start branchName --all 的动作。
该动作会对于每个仓库进行分支的创建， 其中branchName是自己命名的。
//可用repo forall -c "git branch"来查看本地是否成功建立分支。
1.repo forall命令
 # repo forall -help
 # repo forall -c: 此命令遍历所有的git仓库，并在每个仓库执行-c所指定的命令，被执行的命令不限于git命令，而是任何被系统支持的命令，比如：ls, git log, git status等
2.repo forall -c使用
  # 切换分支
  # repo formal -c git checkout dev_test
  # 删除分支
  # repo forall -c git branch -D dev_test
  # 丢弃分支
  # repo forall -c git git reset —hard 提交ID(或最原始:HEAD)
  # repo forall -r framework/base/core -c git reset —hard 提交ID(或最原始HEAD)

#####将本地代码仓库上传提交至另一远程服务器上
创建工程
repo forall -c 'ssh -p 端口号 username@服务器 gerrit create-project $REPO_PROJECT --empty-commit(会创建一个master的分支，一次空提交) --description $REPO_PROJECT'
repo forall -c 'git push ssh://username@服务器:端口号/$REPO_PROJECT 本地分支名:远程服务器的需创建分支名'
//repo forall -c "git push origin 本地分支名:远程服务器的需创建分支名"
创建manifest仓库并提交