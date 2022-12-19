commitlint 推荐我们使用 config-conventional 配置去写 commit，以便于清晰的查看每一次代码提交记录

- build  发布版本

- chore 改变构建流程、或者增加依赖库、工具等

- ci 持续集成修改

- docs  文档修改

- feat  新特性

- fix  修改问题

- perf  优化相关，比如提升性能、体验

- refactor  代码重构

- revert  回滚到上一个版本

- style  代码格式修改

- test  测试用例修改

例如

```
git commit -m 'fix: fix ie6 margin bug'
```