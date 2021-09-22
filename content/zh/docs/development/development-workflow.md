---
title: "开发流程"
linkTitle: "开发流程"
weight: 10
description: >
    本文件将向您介绍项目开发的详细过程。
---

## 步骤1. Fork

1. 访问 https://github.com/OpenFunction/OpenFunction
2. 点击 “Fork” 按钮，在你的 GitHub 账户中创建一个 fork 项目。

## 步骤2. 克隆 fork 项目至本地

Per Go's [workspace instructions](https://golang.org/doc/code.html#Workspaces), place OpenFunction code on your `GOPATH` using the following cloning procedure.

根据 Go 的 [workspace instructions](https://golang.org/doc/code.html#Workspaces)，使用以下克隆工序将 OpenFunction 代码放在你的 `GOPATH` 上。

定义一个本地目录：

```
export working_dir=$GOPATH/src/openfunction.io
export user={your github profile name}
```

将代码克隆到本地：

```
mkdir -p $working_dir
cd $working_dir
git clone https://github.com/$user/OpenFunction.git
cd $working_dir/OpenFunction
git remote add upstream https://github.com/OpenFunction/OpenFunction.git

# Never push to upstream master
git remote set-url --push upstream no_push

# Confirm your remotes make sense:
git remote -v
```

## 步骤3. 保持与上游分支的同步

```
git fetch upstream
git checkout main
git rebase upstream/main
```

## 步骤4. 增加新特性或补丁

从 master 中创建一个新分支：

```
git checkout -b myfeature
```

然后在 `myfeature` 分支上开发代码。参考 [effective_go](https://golang.org/doc/effective_go.html) 有助于你的编码。

### 测试与构建

目前，make 规则只包含简单的检查，如 vet、unit 测试，很快会加入 e2e 测试。

### 使用 KubeBuilder

- 对于 Linux 操作系统，你可以下载并执行这个 [KubeBuilder脚本](https://raw.githubusercontent.com/kubesphere/kubesphere/master/hack/install_kubebuilder.sh)。
- 对于 MacOS，你可以按照这个 [指南](https://book.kubebuilder.io/quick-start.html) 来安装 KubeBuilder。

### 运行与测试

```
make all
# Run every unit test
make test
```

运行 `make help` 以获得关于 make 的额外信息。

## 步骤5. 开新分支开发

### 与上游分支同步

测试完成后，让你本地的代码与上游分支保持同步有助于避免冲突。

```
# Rebase your master branch of your local repo.
git checkout main
git rebase upstream/main

# Then make your development branch in sync with master branch
git checkout new_feature
git rebase -i main
```

### 提交本地变化

```
git add <file>
git commit -s -m "add your description"
```

## 步骤6. 推送至你的 fork 仓库

当准备好审查时（或者只是为了建立一个异地的工作备份），把你的本地分支推送到 GitHub 上的  fork 仓库。

```
git push -f ${your_remote_name} myfeature
```

## 步骤7. 创建一个 PR

- 访问你在 GitHub 上的 fork 仓库 https://github.com/$user/OpenFunction
- 点击你的 `myfeature` 分支旁的 ` Compare & Pull Request` 按钮
- 请查看 [pull requests 流程](pull-requests.md)，了解更多细节和建议

