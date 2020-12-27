## 文件状态转换概览

![image-20201217212016101](https://gitee.com/p8t/picbed/raw/master/imgs/20201217212017.png)

## 基本

```bash
git init [dir-name]
git add <==> git reset HEAD
git status
git commit -m 'message'
# 重新执行上一次提交. 用于一次提交后发现少添加了几个文件
git commit --amend -m 'message'
git log --oneline --graph --all
# 把暂存区的文件移回工作目录, 即取消暂存区的添加 【不会修改文件内容】
git reset HEAD <file>
# 把已修改的文件恢复到上一次提交的样子 【会修改文件内容】
git checkout -- <file>
```

## 标签

```bash
# 列出所有标签
git tag --list
# 打标签
git tag -a <tagname> -m 'message'
# 对指定版本打标签
git tag -a <tagname> <version> -m 'message'
# 删除标签
git tag -d <tagname>
# 查看标签对应的信息
git show <tagname>
```

## 回滚

```bash
# v1 --> v2 --> v3

# 从v3回滚到v2
git reset --hard <v2>
# 再从v2回到v3
git reflog
git reset --hard <v3>

# 上面是一个危险的操作, 一旦找不到v3版本号, 就再也回不去了
# 回到v2
git checkout v2
# 回到v3
git checkout master # 并不是真的回到v3, 而是回到v3所在分支最新那一个版本
```

## 分支

```bash
# 查看分支
git branch
# 创建分支
git branch <new-branch-name>
# 创建分支并同时切换过去
git checkout -b <new-branch-name>
# 切换分支
git checkout <branch-name>
# 合并分支
git merge <branch-name> # 如果有冲突需要手动commit
# 删除分支
git branch -d <branch-name>
# 修改分支名
git branch <old> --move <new>
```

## 远程仓库

```bash
# 添加远程仓库
git remote add <alias> <url>
# 列出所有远程仓库
git remote --verbose
# 推送代码
git push <alias> <branch>
# 拉取(更新)代码. 如果本地什么都没有, 需要先执行git clone
git pull <alias> <branch> # 等同于git fetch <> <> && git merge <>/<>
# 克隆代码
git clone <url> [dir] # 内部已执行(git remote add origin <remote-repository-uri>)
```

