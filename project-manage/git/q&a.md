#### 文件名大小写问题

如果你的文件名前后大小写发生了变化，git 是无法感知变更的，需要特殊处理

方法一：添加配置

```bash
git config core.ignorecase false
```

方法二：手动删除 git 记录再重新提交（建议），步骤如下

1. 假设 Button.js 改成了 button.js，先删除 git 本地仓库的文件

```bash
git rm Button.js -f
```

2. 修改文件名，然后提交

#### pull request

用来执行分支合并，github 称为 pull request，gitlab 称为 merge request，其实是执行了以下的命令

```bash
git pull
git merge
```

流程如下

1. 先 fork 别人的仓库，相当于拷贝一份，相信我，不会有人直接让你改修原仓库的
2. clone 到本地分支，做一些 bug fix
3. 发起 pull request 给原仓库，让他看到你修改的 bug
4. 原仓库 review 这个 bug，如果是正确的话，就会 merge 到他自己的项目中

解决冲突的方式如下

1. 分支 A 向 master 分支合并 pr 时冲突，先在分支 A git merge origin master，解决冲突后 push 到 A 分支
2. 分支 A 向 B 分支合并 pr 时冲突，且 B 分支不能直接提交 commit，只能 merge，先基于 B 分支拉取分支 C，在分支 C git merge origin A，解决冲突后再 merge 到分支 B

#### commitlint

commitlint 推荐我们使用 config-conventional 配置去写 commit，以便于清晰的查看每一次代码提交记录

- build 发布版本
- chore 改变构建流程、或者增加依赖库、工具等
- ci 持续集成修改
- docs 文档修改
- feat 新特性
- fix 修改问题
- perf 优化相关，比如提升性能、体验
- refactor 代码重构
- revert 回滚到上一个版本
- style 代码格式修改
- test 测试用例修改

例如

```bash
git commit -m 'fix: fix ie6 margin bug'
```
