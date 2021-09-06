---
title: "Pull Request 流程"
linkTitle: "Pull Request 流程"
weight: 20
description: >
    本文档将帮助你学习如何提交 pull requests。
---

本文档演示了向 [OpenFunction 项目](https://github.com/OpenFunction/OpenFunction) 提交 PR 的过程和最佳实践。它可以为所有贡献者（尤其是新接触或不经常提交 PR 的开发者）提供有价值的参考。

- [在提交 PR 之前](#在提交-pr-之前)
  - [运行本地验证](#运行本地验证)
- [PR 的提交流程](#pr-的提交流程)
  - [如有需要，编写 Release notes ](#如有需要编写-release-notes)
  - [测试与合并工作流](#测试与合并工作流)
  - [标记未完成的 Pull Requests](#标记未完成的-pull-requests)

## 在提交 PR 之前

本指南是为已经有 PR 要提交的贡献者准备的。如果你正在寻找关于如何搭建你的开发环境以及如何为 OpenFunction 贡献代码的信息，请参见 [开发指南](development-workflow.md) 。

**确保你的 PR 符合我们的最佳实践。这包括遵循项目惯例，提交小型 PR，以及全面地进行评论。请阅读本文件末尾关于[加快评审的最佳实践](#best-practices-for-faster-reviews)的更详细章节。**

### 运行本地验证

你可以在提交 PR 之前运行这些本地验证，以预测持续集成的通过或失败。

## PR 的提交流程

合并一个 PR 需要在自动合并 PR 之前完成以下步骤。关于每个步骤的细节，请参见下面的[测试和合并工作流程](#the-testing-and-merge-workflow)部分。

- 生成 PR
- Release notes -- 包含以下一项：
  - 在 release notes 中添加说明，或
  - 上传 release notes 的标签
- 通过所有测试
- 从 reviewer 处获得一个 `/lgtm` 标签
- 从 owner 处获得一个 `approval`

如果你的 PR 符合上述所有步骤，它将进入提交队列被合并。当它成为下一个被合并的队列时，测试将运行第二次。如果所有的测试都通过了，PR 将被自动合并。

### 如有需要，编写 Release notes 

任何带有用户可见变化的 PR 都需要 release notes，例如错误修复、功能添加和输出格式变化。

如果你不在 PR 模板中添加 release notes，那么在你创建 PR 后，“do-not-merge/release-note-label-needed” 的标签会自动添加到你的 PR 中。有几种方法可以更新它。

在 PR 描述中增加一个 release-note 部分。

对于有 release notes 的 PR：

    ```release-note
    Your release note here
    ```

对于需要用户在切换到新版本时采取额外操作的 PR，请在发布说明中加入 “action required” 的字符串（不区分大小写）：

    ```release-note
    action required: your release note here
    ```

对于不需要在发布时提及的 PR，只需写上 “NONE”（不区分大小写）：

    ```release-note
    NONE
    ```

`/release-note-none` 注释命令仍然可以用来替代在 release-note 块中写 “NONE”，如果它留空的话。

要了解如何格式化您的 release notes，请查看 [PR 模板](https://github.com/) 以了解一个简单的例子。PR 的标题和正文注释可以在发布前的任何时候进行修改，以使其对发布注释友好。
// PR template TODO

Release notes 适用于主分支上的 PR 。对于 cherry-pick PR，请参阅 [cherry-pick instructions](cherry-picks.md) 。这些规则的唯一例外是，当一个 PR 不是 cherry-pick，而是直接针对非主干分支时。 在这种情况下，非主分支 PR 需要一个 `release-note-*` 标签。

// cherry-pick TODO

现在你的 release notes 已经成型了，让我们看看 PR 是如何被测试和合并的。

### 测试与合并工作流

OpenFunction 的合并工作流程使用注释来运行测试和标签来合并 PR。

注意：对于正在进行但尚未准备好接受审查的拉动请求，在 PR 标题前加上 “WIP” 或 “[WIP]”，并在 pull requests 描述中的检查清单中跟踪任何剩余的 TODO。

以下是 PR 从提交到合并的过程：

1. 生成 pull request
2. `@o8x-merge-robot` 分配 reviewer //TODO

如果你 **不是** 的 OpenFunction 组织的成员：

1. Reviewer/OpenFunction 成员检查 PR 是否可以安全测试。如果是的话，他们会评论 `/ok-to-test`。
2. Reviewer 建议编辑
3. 提交编辑至你的 PR 分支
4. 重复最先的两个步骤
5. （可选）一些 reviewer 希望你在这一步合并（squash）提交的内容
6. Owner 被安排处理这次合并，并将在 PR 上添加 `/approve` 标签

如果你是一个项目的成员，或者一个评论了 `/ok-to-test` 的成员，该 PR 将被认为是可信的。然后提交前的测试就会运行：

1. 自动测试运行。参见 PR 上的当前测试列表
2. 如果测试失败，通过向你的 PR 分支推送编辑来解决问题
3. 如果失败是偶然的，任何受信任的 PR 上的人都可以评论 `/retest` 来重新运行失败的测试

一旦测试通过，所有的失败都被评论为偶然的，或者评审员添加了 `lgtm` 和 `approved` 标签，PR 就进入了最后的合并队列。合并队列是必须的，以确保在你的 PR 上最后一次运行测试后，其他 PR 没有引入不兼容的变化。

1. 该 PR 进入合并队列
2. 合并队列触发了一个测试的重新运行，注释为 `/test all [submit-queue 正在验证这个 PR 是否可以安全合并]`。
    2.1. 作者已经签署了 CLA（`cncf-cla: yes` 标签添加到 PR）。
    2.2. 自上次应用 `lgtm` 标签以来，没有做出任何改变
3. 如果测试失败，通过向你的 PR 分支推送编辑来解决问题
4. 如果失败是偶然的，任何受信任的 PR 上的人都可以评论 `/retest` 来重新运行失败的测试
5. 如果测试通过，合并队列将自动合并 PR

这就是最后一步了。你的 PR 现在被合并了。

### 标记未完成的 Pull Requests

如果你想在你的 pull requests 执行完成之前征求评论，你应该保留你的 pull requests，以确保合并队列不会试图合并它。有两种方法可以实现这一点。

1. 你可以添加 `/hold` 或 `/hold cancel` 的注释命令
2. 你可以在你的 pull requests 标题中添加或删除 `WIP` 或 `[WIP]` 前缀

当你使用评论命令时，GitHub 机器人会添加和删除 `do-not-merge/hold` 标签，当你编辑标题时，会添加和删除 `do-not-merge/work in-progress` 标签。当这两个标签存在时，你的 pull requests 将不被考虑合并。
