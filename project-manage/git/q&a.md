#### 文件名大小写问题

如果你的文件名前后大小写发生了变化，git是无法感知变更的，需要特殊处理

方法一：添加配置

```bash
git config core.ignorecase false
```

方法二：手动删除git记录再重新提交（建议），步骤如下

1. 假设Button.js改成了button.js，先删除git本地仓库的文件

```bash
git rm Button.js -f
```

2. 修改文件名，然后提交