
### github

testtest

1. 设置传输方式为ssh
> git remote set-url

2. git status中文文件名乱码
> git config --global core.quotepath false

#### [git cheat sheet](https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf)

- 比较两个分支的异同

```shell
// 差集
git log dev..alpha	// alpha中有，dev中没有
git log alpha..dev	// dev中有，alpha中没有
git log origin/master..HEAD	//本地分支与远程分支的不同

git log ^dev alpha	// 同dev..alpha
git log HEAD --not dev // 同dev..alpha

// 全差（左差集和右差集的并集）
git log dev...alpha	// dev中有,alpha中没有和alpha中有,dev中没有
git log --left-right dev...alpha	// 列出两个分支的差异，同时指明是哪个分支
```

git difftool


#### git merge 与 git rebase

[rebase](https://www.zhihu.com/search?q=rebase&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1990894567}) 和 merge 不是二选一的关系，要协同使用，毕竟作者设计出两个命令不是让你挑一个来用的。

一个最简单的模型，从 master 分支 checkout 出几个本地 feature 分支，你或者你的团队在协同开发某个 feature-a 时，可能别人已把 feature-b 的代码 merge 回 master 了，所以应该及时将 master 的改动 rebase 到你的本地分支，顺便 fix conflicts。即：

```bash
$ git switch feature-a
$ git rebase master
fix conflicts...
$ git rebase --continue
```

当你开发完成 feature-a 时，应该将改动 merge 回 master。即：

```bash
$ git switch master
$ git merge --no-ff -m "Merge branch 'feature-a'" feature-a
```

在本地分支中使用 rebase 来合并主分支的改动，是为了让你的本地提交记录清晰可读。（当然， rebase 不只用来合并 master 的改动，还可以在协同开发时 rebase 队友的改动。）

在主分支中使用 merge 来把 feature 分支的改动合并进来，是为了保留分支信息。

如果全使用 merge 就会导致提交历史繁复交叉，错综复杂。如果全使用 rebase 就会让你的 commits history 变成一条光秃秃的直线。

一个好的 commits history，应该是这样的：

```text
*   e2e6451 (HEAD -> master) feture-c finished
|\
| * 516fc18 C.2
| * 09112f5 C.1
|/
*   c6667ab feture-a finished
|\
| * e64c4b6 A.2
| * 6058323 A.1
|/
*   2b24281 feture-b finished
|\
| * c354401 B.4
| * 4bfefb8 B.3
| * eb13f72 B.2
| * c2c62b9 B.1
|/
* bbbba82 init
```

而不是这样的：

```text
*   9f0c13b (HEAD -> master) feture-c finished
|\
| * 55be61c C.2
| *   e18b5c5 merge master
| |\
| |/
|/|
* |   ee549c2 feture-a finished
|\ \
| * | 51f2126 A.3
| * |   72118e2 merge master
| |\ \
| |/ /
|/| |
* | |   6cb16a0 feture-b finished
|\ \ \
| * | | 7b27b77 B.3
| * | | 3aac8a2 B.2
| * | | 2259a21 B.1
|/ / /
| * | 785fab7 A.2
| * | 2b2b664 A.1
|/ /
| * bf9e77f C.1
|/
* 188abf9 init
```

也不是这样的：

```text
* b8902ed (HEAD -> master) C.2
* a4d4e33 C.1
* 7e63b80 A.3
* 760224c A.2
* 84b2500 A.1
* cb4c4cb B.3
* 2ea8f0d B.2
* df97f39 B.1
* 838f514 init
```

就这么简单。

#### 小技巧总结

- git 删除某个中间提交的记录

  1. git log获取commit信息  
  2. git rebase -i (commit-id)   commit-id 为要删除的commit的下一个commit号
  3. 编辑文件，将要删除的commit之前的单词改为drop  
  4. 保存文件退出  
  5. git log查看

- 查看分支，按更新时间排序

  ```bash
  git branch -a --sort=-committerdate // 降序
  git branch -a --sort=committerdate // 升序
  ```

