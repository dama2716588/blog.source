title: 常用的SVN命令小结
tags:
  - svn
categories:
  - svn
date: 2015-11-10 20:06:00
---
### 1，基础命令

``` bash
svn co repository_url // check out respoitory
svn ci -m "your comments" // commit files
svn up repository_url // update files
```

### 2，branch & tag

所谓打tag，要从[SVN](https://www.baidu.com/s?wd=SVN&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1Y3PA7bP1mvrj99uH61rHT10ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6K1TL0qnfK1TL0z5HD0IgF_5y9YIZ0lQzqlpA-bmyt8mh7GuZR8mvqVQL7dugPYpyq8Q1n1Pjc1nWRvPs)官方推荐的目录结构说起了。[SVN](https://www.baidu.com/s?wd=SVN&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1Y3PA7bP1mvrj99uH61rHT10ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6K1TL0qnfK1TL0z5HD0IgF_5y9YIZ0lQzqlpA-bmyt8mh7GuZR8mvqVQL7dugPYpyq8Q1n1Pjc1nWRvPs)官方推荐在一个版本库的根目录下先建立trunk、branches、tags这三个文件夹，其中trunk是开发主干，存放日常开发的内容；branches存放各分支的内容，比如为不同客户定制的不同版本；tags存放某个版本状态的标签，比如验收测试版、1.0.3版等。branhces和tags本质没有区别，都是通过[svn](https://www.baidu.com/s?wd=svn&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1Y3PA7bP1mvrj99uH61rHT10ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6K1TL0qnfK1TL0z5HD0IgF_5y9YIZ0lQzqlpA-bmyt8mh7GuZR8mvqVQL7dugPYpyq8Q1n1Pjc1nWRvPs) copy方式建立的，差异在于通常branches中的内容是需要继续修改或开发的，tags中的内容是存放不再修改的，这一般通过权限设置来解决，tags通常只给管理员开放写权限。

``` bash
// 新建分支
svn copy master_repository_url branch_repository_url -m "your comments"

// 新建空白分支
svn mkdir branch_repository_url

// 删除分支
svn rm branch_repository_url -m "your comments"

// 新建tag
svn copy master_repository_url tag_repository_url -m "your comments"

// 删除tag
svn rm tag_repository_url -m "your comments"

// 查看branches
svn ls ^/branches --verbose
```

### 3，回滚

3-1），改动没有被提交

这种情况下，使用svn revert就能取消之前的修改。svn revert用法如下：当something为单个文件时，直接svn revert something就行了；当something为目录时，需要加上参数-R(Recursive,递归)，否则只会将something这个目录的改动。

3-2），改动已经被提交

这种情况下，用svn merge命令来进行回滚。先运行svn up保证拿到最新的版本，然后svn log查看并找到要回滚的版本号，如果想要更详细的了解情况，可以使用svn diff -r HEAD:2500 [something]，此处的something可以是文件、目录或整个项目。

如果需要回滚到版本号2500：

``` bash
svn merge -r HEAD:2500 something
```

为了保险起见，再次确认回滚的结果：

``` bash
svn diff -r HEAD:2500 [something]
```

发现无误，提交即可。

3-3），分支与主干的合并：

``` bash
# 分支合到主干 cd trunk
svn merge -r <revision where branch was cut>:<revision of trunk> svn://branch/path

# 分支当前版本为4847，想把4825到4847间的改动merge到主干
# cd trunk
svn merge -r 4825:4847 svn://branch/path
svn ci -m "merge branch changes r4835:4847 into trunk"

# 主干合到分支 cd branch
# 在r23创建了一个分支，trunk版本号更新到了25，想把23-25之间的改动merge到分支
svn merge -r 23:25 svn://trunk/path
svn ci -m "merge trunk changes r23:25 into my branch"

# cd trunk
# 查看当前Branch中已经有那些改动已经被合并到Trunk中
svn mergeinfo svn://branch/path

# cd trunk
# 查看Branch中那些改动还未合并
svn merginfo svn://branch/path --show-revs eligible
```

3-4），merge分支B到分支A

``` bash
step1: Checkout URL A
       # cd branch A
step2: merge URL B to your working copy of A
       svn merge -r 10:HEAD http://branch-b .
step3: Commit A
```

### 4，冲突提示

``` bash
(p) postpone          暂时推后处理，我可能要和那个和我冲突的家伙商量一番
(df) diff-full        把所有的修改列出来，比比看
(e) edit              直接编辑冲突的文件
(mc) mine-conflict    如果你很有自信可以只用你的修改，把别人的修改干掉
(tc) theirs-conflict  底气不足，还是用别人修改的吧
(s) show all options  显示其他可用的命令
```

### 5，svn符号

``` bash
U:表示从服务器收到文件更新了
G:表示本地文件以及服务器文件都已更新,而且成功的合并了 
其他的如下:
A:表示有文件或者目录添加到工作目录
R:表示文件或者目录被替换了.
C:表示文件的本地修改和服务器修改发生冲突
```

### 6，patch
有时同事A做的修改需要同事B去Review，同事C去提交。使用patch工具可以很好的决代码传递。
6-1).生成patch:
同事 A 运行如下命令生成 patch:
``` bash 
svn diff > aaa.patch
```

6-2).应用patch:
同事 B 运行如下命令应用 patch:
``` bash 
patch –p0 < aaa.patch
```

6-3).去除patch，恢复旧版本
当他 review 完代码，想删除该 patch 时， 可运行：
``` bash 
 patch -RE -p0 < aaa.patch
 ```
-p0，是“当前路径”
-p1，是“上一级路径”


参考：http://stereointeractive.com/blog/2009/02/17/svn-merge-trunk-changes-to-your-branch/