用来执行分支合并，github 称为 pull request，gitlab 称为 merge request，其实是执行了以下的命令

```bash
git pull
git merge
```

#### 流程

1. 先 fork 别人的仓库，相当于拷贝一份，相信我，不会有人直接让你改修原仓库的

2. clone 到本地分支，做一些 bug fix

3. 发起 pull request 给原仓库，让他看到你修改的 bug

4. 原仓库 review 这个 bug，如果是正确的话，就会 merge 到他自己的项目中

#### 解决冲突

1. 分支 A 向 master 分支合并 pr 时冲突，先在分支 A git merge origin master，解决冲突后 push 到 A 分支

2. 分支 A 向 B 分支合并 pr 时冲突，且 B 分支不能直接提交 commit，只能 merge，先基于 B 分支拉取分支 C，在分支 C git merge origin A，解决冲突后再 merge 到分支 B
