
### github

1. 设置传输方式为ssh
> git remote set-url

2. git status中文文件名乱码
> git config --global core.quotepath false

### [git cheat sheet](https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf)

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