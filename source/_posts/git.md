---
title: git学习过程
date: 2020-01-19 11:04:42
tags: git
categories: git
---

学习git的记录
<!-- more -->

## 合并到master分支

```bash
m commits behind master , n commits ahead 
m | n
```

#### 先变成 m | 1

```bash
git pull
git reset --soft HEAD~n
git status
git commit -m ""
git push -f
```

#####再将m | 1 变为 0 | 1

```bash
git checkout master
git pull origin master
git checkout wh
git rebase master
git push -f
```


git 回退打破上一次分支合并的merge

1.找到上一次提交到master分支的版本号

```bash
git log --pretty=oneline
```

2.回退到指定版本

```bash
git reset --hard [merge前的版本号]
```

merge过程

当git pull时出现冲突问题时，如下：

```bash
warning: Cannot merge binary files: regionminute10_15.xlsx (HEAD vs. 03930ecfcd8a9c20eebe2d7dcd62e830455f1642)
warning: Cannot merge binary files: data3.xlsx (HEAD vs. 03930ecfcd8a9c20eebe2d7dcd62e830455f1642)
Auto-merging users3_15.py
Auto-merging regionminute10_15.xlsx
CONFLICT (content): Merge conflict in regionminute10_15.xlsx
Auto-merging data3.xlsx
CONFLICT (content): Merge conflict in data3.xlsx
Automatic merge failed; fix conflicts and then commit the result.
```

右下角则会出现 merging 状态，此时pull push等都不能用，只能见解决冲突或者取消merge

```bash
git merge --abort #取消merge
```

解决冲突过程

![merge](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/merge.png?raw=true)

选择Resolve Conflicts

![resolve](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/resolve.png?raw=true)

如图可以发现冲突的文件，如果你的版本低于服务器的版本直接Accept Theirs,如果你的是最新的版本直接accept yours，如果不确定的话点击merge，逐行代码进行更新替换。

最后结束merge

```bash
git merge --continue
```

## git提交每次都要输入用户名和密码

问题:使用git提交到github,每次都要输入用户名和密码，操作起来很麻烦。

原因:在clone项目或上传项目的时候，使用的是https方式，而不是ssh方式，切换一下即可。

![git](https://github.com/wentianhao/wentianhao.github.io/blob/master/images/git.png?raw=true)

选择ssh方式，复制url

解决办法：

1.查看是https方式还是ssh方式

```bash
git remote -v   
```

2.移除https的方式，换成ssh方式

```bash
git remote rm origin
```

3.添加新的git方式：ssh方式

```bash
git remote add origin git@github.com:xxxxxx.git
```

4.查看push方式是否修改成功

```bash
git remote -v
```

5.push测试是否需要用户名和密码

```bash
git push origin master
```

## github删除一个commit(回滚到某个commit)的方法

- 获取提交的历史

```bash
git log
```

找到提交的commit_id

- 回滚

```bash
git reset --hard commit_id
```

- 提交,回滚至之前的代码

```bash
git push origin HEAD --force
```

