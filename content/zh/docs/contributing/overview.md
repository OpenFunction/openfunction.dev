---
type: docs
title: "概述"
linkTitle: "概述"
weight: 7100
description:
---

本文档提供了如何通过问题和拉取请求向 [OpenFunction](https://github.com/OpenFunction) 做出贡献的指南。贡献也可以通过其他方式进行，例如在社区电话会议中与社区进行互动，在问题或拉取请求上发表评论等。

请参阅 [OpenFunction 社区仓库](https://github.com/OpenFunction/community) 以获取有关社区参与和社区成员资格的更多信息。

## 问题

### 问题类型

在大多数 OpenFunction 仓库中，通常有 4 种类型的问题：

- 问题/错误报告：您发现了一个错误，并希望报告并跟踪它。
- 问题/功能请求：您希望使用一个功能，但它尚未得到支持。
- 问题/提案：用于提出新的想法或功能的项目。这允许在编写代码之前从他人那里获得反馈。

### 提交前的检查

在您提交问题之前，请确保您已经检查了以下内容：

1. 这是正确的仓库吗？
    - OpenFunction 项目分布在多个仓库中。如果您不确定哪个仓库是正确的，请检查 [仓库列表](https://github.com/OpenFunction)。
2. 检查现有的问题
    - 在您创建新问题之前，请在 [开放问题](https://github.com/OpenFunction/OpenFunction/issues) 中搜索，看看问题或功能请求是否已经被提交。
    - 如果您发现您的问题已经存在，请发表相关评论并添加您的 [反应](https://github.com/blog/2119-add-reaction-to-pull-requests-issues-and-comments)。使用反应：
        - 👍 赞成
        - 👎 反对

## 拉取请求

所有的贡献都通过拉取请求来进行。要提交建议的更改，请遵循以下工作流程：

1. 确保已经提出了问题，这为您即将做出的贡献设定了期望。
2. 分叉相关的仓库并创建一个新的分支
3. 创建您的更改
    - 代码更改需要测试
4. 更新相关的文档以进行更改
5. 带有 [DCO 签名]({{< ref "overview.md#developer-certificate-of-origin-signing-your-work" >}}) 的提交并打开一个 PR
6. 等待 CI 过程完成并确保所有检查都是绿色的
7. 您可以在几天内期待一次审查

### 使用工作进度中的 PR 获取早期反馈

在投入太多时间之前，一种进行沟通的好方法是创建一个 "工作进度中" 的 PR 并与您的审查者分享。做到这一点的标准方式是在您的 PR 的标题中添加 "[WIP]" 前缀并分配 **do-not-merge** 标签。这将让查看您的 PR 的人知道它还没有准备好。

## 开发者原创证书：签署您的工作
### 每个提交都需要签署

开发者原创证书 (DCO) 是一种轻量级的方式，供贡献者证明他们编写或者以其他方式有权提交他们正在向项目贡献的代码。这里是 [DCO](https://developercertificate.org/) 的全文。

贡献者通过在提交消息中添加 `Signed-off-by` 行来签署他们遵守这些要求。

```
这是我的提交消息
Signed-off-by: 随机 J 开发者 <random@developer.example.org>
```

Git 有一个 `-s` 命令行选项，可以自动将此内容添加到您的提交消息中：
```
git commit -s -m '这是我的提交消息'
```

每个拉取请求都会检查拉取请求中的提交是否包含有效的 Signed-off-by 行。

### 我没有签署我的提交，现在该怎么办？！

别担心 - 您可以轻松地重播您的更改，签署它们并强制推送它们！

```
git checkout <branch-name>
git commit --amend --no-edit --signoff
git push --force-with-lease <remote-name> <branch-name>
```

## 开发指南

[这里](https://github.com/OpenFunction/OpenFunction/tree/main/docs/development) 您可以找到一个开发指南，它将指导您如何开始在您的本地环境中构建 OpenFunction。

## 行为准则

请参阅 [OpenFunction 社区行为准则](https://github.com/OpenFunction/OpenFunction/blob/main/code-of-conduct.md)。
