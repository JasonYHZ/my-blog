---
title: Git 协作规范
date: 2023-03-02 11:07:00
comment: true
toc: true
cover: https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171259572.webp
tags:
    - Git
    - 提交规范
    - Git Commit
categories:
    - 教程
    - 规范规则
description: 当多人协作开发一个项目时，使用 Git 进行版本控制是非常重要的。为了保证团队协作的顺利进行，需要遵循一些 Git 规范。下面是一些 Git 规范的建议，供你参考。
---

当多人协作开发一个项目时，使用 Git 进行版本控制是非常重要的。为了保证团队协作的顺利进行，需要遵循一些 Git 规范。下面是一些 Git 规范的建议，供你参考。

## 模型分支图

依据 [GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html) 模型修改简化。若想深入了解相关模型，可点击链接阅读详情。

![git flow模型](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171259572.webp)

## 分支定义

### 1.master 分支

1. 长期存在于项目的开发分支，是所有分支的上游分支，禁止删除
2. 遵循“上游优先”原则，只有上游分支采纳的代码变化，才能应用到其他分支
3. 禁止在此分支中进行 push 推送，只能进行 Meger Request 合并
4. 由项目负责人对此分支进行管理
5. 代码部署上线后应当进行 Tag 版本标记,遵循语义化版本号规则

### 2.Pre-Production 分支

1. 此分支为预发分支，禁止删除
2. 开发分支测试通过的代码，经过 MR 合并到上游 Master 分支后，可将代码从上游分支部署至此分支
3. 此分支代码对应的是应用的 release 状态，即所有经过测试的可发布的新功能代码都将从上沉淀到此
4. 此分支所操作的数据库应当是与线上环境为同一套数据库
5. 禁止在此分支中进行 push 推送，只能从上游分支（Master）进行 Meger Request 合并
6. 由项目管理人员进行发布管理
7. 代码部署上线后应当进行 release Tag 版本标记，遵循语义化版本号规则

### 3.Production 分支

1. 长期存在于项目的正式应用分支，此分支代码与正式应用保持强一致性，禁止删除
2. 禁止在此分支中进行 push 推送，只能从Pre-Production分支进行 Meger Request 合并
3. 由项目管理人员进行发布管理

### 4.Feature 分支

1. 此分支为功能开发分支
2. 一般根据相关需求，由开发人员自行从 master 创建
3. 分支名规则为 feat/_xxxx-date_
4. 提交MR之前应当本地同步上游分支最新代码，以确保解决了冲突
5. 此分支通过测试后通过 Merger Request 来向 Master 分支进行合并
6. MR 应当修剪提交commit，MR 结束时此分支应当立即删除

### 5.HotFix 分支

1. 此分支可以视为比较特殊的 Feateure 分支
2. 此分支明明规则为 fix/_xxxx-date_
3. 若为非紧急 bug，则按照一般 Feature 分支研发流程规则进行对待
4. 若为紧急 bug, 则可跳过上游分支，直接对 Production 分支进行合并，之后再回归到 master 分支

### 6.Test分支

1. 此分支对应的是测试环境，对此分支进行push后将会自动触发 ci/cd 构建部署
2. 此分支的上游就是每个 feature/hotfix 分支，此分支可以被删除重建
3. 开发环节的提测也是在此分支进行

## 版本号语义化规则

版本格式：主版本号.次版本号.修订号，版本号递增规则如下：

主版本号：当你做了不兼容的 API 修改，
次版本号：当你做了向下兼容的功能性新增，
修订号：当你做了向下兼容的问题修正。先行版本号及版本编译元数据可以加到“主版本号.次版本号.修订号”的后面，作为延伸。

主版本号为 0，代表还未发布正式版本。

## commit 规范

每次提交 commit 之前一定要编写具有完整语义化的 commit message，否则会拒绝合并。

```bash
git commit -m "fix:fix #1"

# 如果一行不够写，就可以只执行 git commit 这时就会跳出文本编辑器可以写多行
git commit
```

### commit message 格式

每次提交的 message 都包括三个部分 Header，Body 和 Footer，格式如下

```shell
<type>(<scope>):<subject>
// 空行
<body>
// 空行
<footer>
```

其中 Header 是必需的，Body 和 Footer 可省略。

无论是哪一个部分，任何一行都不能超过 72 个字符（或者100个字符）避免自动换行。

#### 1. Header

Header 部分只有一行，包括三个字段：type（必须）、scope（可选）、subject（必须）。

(1) `type` 用于说明 commit 的类别，只允许使用下面的关键字

| 关键字   | 说明                                                                            |
| -------- | ------------------------------------------------------------------------------- |
| init     | 项目初始化                                                                      |
| feat     | 添加新特性                                                                      |
| fix      | 修复 bug                                                                        |
| docs     | 文档改进，包括代码注释                                                          |
| style    | 仅仅修改了空格、格式缩进、逗号等等，不改变代码逻辑（对结构有影响的用 refactor） |
| refactor | 代码重构，没有加新功能或者修复 bug                                              |
| perf     | 优化相关，比如提升性能、体验                                                    |
| test     | 增加测试用例                                                                    |
| build    | 依赖相关的内容                                                                  |
| ci       | ci 配置相关例如对 k8s，docker 的配置文件的修改                                  |
| chore    | 改变构建流程、或者增加依赖库、工具等                                            |
| revert   | 回滚到上一个版本                                                                |

如果 `type` 为 `feat、fix`，则此次 commit 将必须出现在 change log 中。其他类型可视情况而定。

(2) `scope` 用于说明本次 commit 影响的范围，比如数据层、控制器层等。

(3) `subject` 是对 commit 的简短描述，尽量不超过 50 个字符

- 以动词开头，使用第一人称现在时，比如change，而不是 changed 或 changes
- 第一个字母小写
- 结尾不加句号

#### 2. Body

Body 是对本次 commit 的详细描述，可以分成多行。例如：

```bash
如有必要，更详细的解释性文本。 把它包裹起来
大约72个字符左右。

更多的段落写在空行之后。

- Bullet points are okay, too
- Use a hanging indent
```

注意点：

- 以动词开头，使用第一人称现在时，比如change，而不是 changed 或 changes
- 应该说明代码变动的动机，以及与以前行为的对比。

#### 3. Footer

Footer 只适用于两种情况。

**(1) 不兼容变动**

如果当前代码与上一个版本不兼容，则 Footer 部分以 `BREAKING CHANGE` 开头,后面是对变动的描述，以及变动理由和迁移方案。

```bash
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

**(2) 关闭 Issue**

如果当前 commit 针对某个 issue，那么可以在 Footer 部分关闭这个 issue 。

```bash
Closes #123
# 或者
Closes  #123, #245, #992
```

#### 4.Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以 `revert:` 开头，后面紧跟被撤销的 Commit 的 Header。

```bash
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body 部分的格式是固定的，必须写成 `This reverts commit &lt;hash>.`，其中的 `hash` 是被撤销 commit 的 SHA 标识符

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的 `Reverts` 小标题下面。

### 辅助工具

#### 1. Commitizen

Commitizen 是一个辅助编写 Commit message 的工具。

安装命令 `npm install -g commitizen`

然后在项目目录里运行

```
commitizen init cz-conventional-changelog --save --save-exact
```

以后凡是用到 `git commit` 的地方可以全部使用  `git cz` 来代替。

注意此工具会在你的项目根目录中初始化 node 模块。如果不是 node 类的项目会造成项目干扰。

#### 2. IDE Plugins - Git Commit Template （推荐）

插件主页：[https://plugins.jetbrains.com/plugin/9861-git-commit-template](https://plugins.jetbrains.com/plugin/9861-git-commit-template)

插件使用截图：

![插件使用截图](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171301801.webp)

![插件使用截图](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171302762.webp)

#### 3. 生成 Change Log
如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成。
生成的文档包括以下三个部分。

- New features
- Bug fixes
- Breaking changes

每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，你还可以添加其他内容。
[conventional-changelog](https://github.com/conventional-changelog/conventional-changelog) 就是生成 Change log 的工具，运行下面的命令即可。
```bash
npm install -g conventional-changelog-cli

conventional-changelog -p angular -i CHANGELOG.md -w
```
上面命令不会覆盖以前的 Change log，只会在 `CHANGELOG.md` 的头部加上自从上次发布以来的变动。
如果你想生成所有发布的 Change log，要改为运行下面的命令。

```bash
conventional-changelog -p angular -i CHANGELOG.md -w -r 0
```

为了方便使用，可以将其写入 `package.json` 的 `scripts` 字段。

```json
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -w -r 0"
  }
}
```

以后可以直接运行下面的命令

```json
npm run changelog
```
## 注意事项

1. 不要对已经推送过的提交（pushed commits）进行变基（rebase）

> 当你推送（push）到公共分支后，你就不该对其变基，因为这样会很难跟进你正在改进的内容，很难跟进测试结果是什么，而且它打破了 cherry-picking。有时我们在审查流程的后期要求代码贡献者合并及变基（`git merge --squash`），以使一些东西容易还原时，我们也会违背这一规则。但一般而言，准则是：代码应干净，修改历史应真实。


2. 请确保每一个feature分支的 commit 都是有意义的，若有比较乱的commit 应在提交前进行 rebase 修剪。
3. 再向 Master 分支合并时，一律使用 --no-ff 参数强行关闭 fast-forward 方式合并
4. 成功部署到 Production 分支的commit节点必须打上 tag
5. 部署到 pre-production 分支的节点commit 节点打 tag 时必须以 release-前缀
6. Master 分支仅允许 PR/MR 合并，禁止 push,  Pre-production,Production 分支禁止推送代码，仅仅允许Master或hotfix分支部署

## 操作实例

### 1.Feature 开发流程图

#### 示例一：有序单线路开发流

![有序单线路开发流](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171302705.webp)

#### 示例二：无序多线路开发流

![无序多线路开发流](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171303134.webp)

### 2. Test 分支使用示例

![Test 分支使用示例](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171303747.webp)

### 3. Pre-Production、Production 上线示例

#### 1.Pre-Production 预发流

![Pre-Production 预发流](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171304499.webp)

#### 2. Production 正式发布流

![Production 正式发布流](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171304287.webp)

### HotFix 修复流示例

![HotFix 修复流示例](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202303171304496.webp)
