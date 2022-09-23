#### 流程

1. 先 fork 别人的仓库，相当于拷贝一份，相信我，不会有人直接让你改修原仓库的

2. clone 到本地分支，做一些 bug fix

3. 发起 pull request 给原仓库，让他看到你修改的 bug

4. 原仓库 review 这个 bug，如果是正确的话，就会 merge 到他自己的项目中

#### 解决冲突

1. 分支A向master分支合并pr时冲突，先在分支A git merge origin master，解决冲突后push到A分支

2. 分支A向B分支合并pr时冲突，且B分支不能直接提交commit，只能merge，先基于B分支拉取分支C，在分支C git merge origin A，解决冲突后再merge到分支B