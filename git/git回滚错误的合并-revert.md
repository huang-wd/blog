[TOC]
# revert 撤销
git revert 被用来撤销一个已经提交的快照
生成一个撤消了 引入的修改的新提交，然后应用到当前分支。而不是从项目历史中移除这个提交。
### 1. 单分支撤销

```
---o---o---o---M---x---x
```
在M引入了一个bug
我们可以通过新的提交来fix这个bug，也可以git revert, 来剔除这个bug
```
git revert M
```
得到结果
```
---o---o---o---M---x---x---W
```
这个时候bug的改动被撤销了，产生了一个新的commit W，但是commit M没有被清除。
### 2. 撤销合并
```
 ---o---o---'o---M---x---x
               /
       ---A---B
```
A和B是dev分支上有问题的提交，
M是master分支，合并之后包含了有问题的合并，
x是其他与有问题代码不相关的提交，
我们可以用git revert 来取消此次合并
该操作对应的git命令为：
```
$ git revert -m 1 M
```
得到结果

```
 ---o---o---'o---M---x---x---W
               /
       ---A---B
```
W是撤销M点之后的提交（revert of the merge M）

命令解释：
试图撤销两个分支的合并的时候Git不知道要保留哪一个分支上的修改。所以我们需要告诉git我们保留那个分支
-m后面带的参数值 可以是1或者2，对应着parent的顺序，
上面例子：1代表'o，2代表B 所以该操作会保留master分支的修改，而撤销dev分支合并过来的修改。

## 问题延伸：
因为我们抛弃过之前dev合并过来的commit，下次dev分支bug修复，再往master合并
#### 1. revert of the revert
修改bug之后，未合并之前（C和D为bug修复之后的提交）
```
---o---o---'o---M---x---x---W---x
               /
       ---A---B-------------------C---D
```
此时如果合并dev分支到master分支，A和B提交的代码不会出现在合并之后的master分支上，因为已经在W点被还原了。

我们可以还原上一次的revert （ revert the previous revert）
git命令
```
$ git revert W
```
执行之后的情况
```
 ---o---o---'o---M---x---x---W---x---Y
               /
       ---A---B-------------------C---D
```
Y点为还原W点之后的提交（revert of the revert）

之后在进行合并即可
```
git checkout master
git merge dev
```
之后的情况
```
 ---o---o---'o---M---x---x---W---x---Y---Z
               /                        /
       ---A---B-------------------C----D
```
#### 2.有些情况不能 revert of the revert
如果放弃A和B点的开发，并且在还原之后的W点之后，重新拉取分支来进行开发dev2
```
       ---A---B
               \
---o---o---o---M---x---x---W---x---x
                                \
                                 A'--B'--C'
```
此时在合并时进行"revert of revert"，之后在合并结果如下：
```
       ---A---B
               \
 ---o---o---o---M---x---x---W---x---x---Y---Z
                                 \         /
                                  A'--B'--C'
```
得到的结果为：在开发阶段dev2分支是在没有包含A、B点的情况下进行的开发，在合并阶段，又还原回来了A、B点进行合并，
会造成重复的改变，从而引发冲突。
```
不能盲目的、不加思考的
进行 "revert of revert"
```
#### 3.不用"revert of the revert"能不能合并
```
 P---o---o---M---x---x---W---x
  \         /
   A---B---C----------------D---E   <-- fixed-up topic branch
```
执行命令
```
$ git checkout E
$ git rebase --no-ff P
```
结果
```
 P---o---o---M---x---x---W---x
  \         /
   A'---B'---C'----------------D'---E' <-- recreated topic branch
```
之后我们就可以在不revert W点的情况下，合并重建之后的dev分支到master分支
```
 P---o---o---M---x---x---W---x---Z
  \                              /
   A'---B'---C'------------D'---E'
```
The natural thing to do in this case is to checkout the A-B-C branch and use
"rebase -i P" to change commit B.  However this does not rewrite commit A,
because "rebase -i" by default fast-forwards over any initial commits selected
with the "pick" command.

The --no-ff option creates a new branch A'-B'-C' with all-new commits (all the
SHA IDs will be different) even if in the interactive case you only actually
modify commit B.  You can then merge this new branch directly into the mainline
branch and be sure you'll get all of the branch's changes.

You can also use --no-ff in cases where you just add extra commits to the topic
to fix it up.

**参考资料：**
- [revert-a-faulty-merge](https://github.com/git/git/blob/master/Documentation/howto/revert-a-faulty-merge.txt)