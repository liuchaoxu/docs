# git stash
常用`git stash`命令：

`git stash`: 备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中。

`git stash save ‘message’`：备份工作区内容，同时添加备注信息。

`git stash save -a “messeag” `：没有加 -a 这个option选项，代码开发可能是在原代码上进行修改的。而对于在项目里加入了代码新文件的开发来说，-a选项才会将新加入的代码文件同时放入暂存区。

`git stash apply`: 从Git栈中读取最近一次保存的内容，恢复工作区的相关内容。但是不会将该stash记录删除

`git stash drop`: 把最近的一条stash记录删除。

`git stash pop`: 从Git栈中读取最近一次保存的内容，恢复工作区的相关内容。由于可能存在多个Stash的内容，所以用栈来管理，pop会从最近的一个stash中读取内容并恢复，同时会删除这条stash记录，相当于`git stash apply`和`git stash drop`一起执行了。

`git stash list`: 显示Git栈内的所有备份，可以利用这个列表来决定从那个地方恢复。

`git stash clear`: 清空Git栈，原来存储的所以stash的节点都消失了。

## 在提交修改功能代码到 Git 之前，通常需要遵循一定的提交信息规范，以便更好地记录和追踪代码变更。以下是一些常见的 Git 提交信息前缀和建议：
`feat`: 新增一个功能。
示例：feat: 添加用户注册功能
`fix`: 修复一个 bug。
示例：fix: 修复登录页面的显示问题
`docs`: 只改动了文档相关的内容。
示例：docs: 更新 README 文件
`style`: 改动代码格式，不影响代码逻辑。
示例：style: 格式化代码
`refactor`: 重构代码，既不是新增功能，也不是修复 bug 的代码变动。
示例：refactor: 优化用户服务类
`perf`: 改进性能的代码变动。
示例：perf: 提高查询响应速度
`test`: 添加或修改测试代码。
示例：test: 增加用户模块的单元测试
`chore`: 构建过程或辅助工具的变动。
示例：chore: 升级 Maven 版本
`ci`: 持续集成相关的变动。
示例：ci: 配置 Jenkins 构建流程
`revert`: 回滚某个提交。
示例：revert: 撤销上次的错误提交
假设你修改的是原本的功能代码，可以使用 fix 或 refactor 前缀