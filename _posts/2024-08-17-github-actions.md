---
layout: post
title: "自定义GitHub Actions"
date: 2024-08-17 13:32:38 +0800
comments: true
tags: Note
---

[GitHub Actions](https://github.com/features/actions) 是 GitHub 官方推出的持续集成和持续交付(CI/CD)平台，可以让用户便捷实现自动化构建、测试和部署流程。

![github-actions](/images/github-actions/github-actions.png)

### 关于 GitHub Actions

GitHub Actions 有几个概念（workflow、event、job、action、step、runner)，下面会简单介绍常用的workflow、job、step的使用。

![overview-actions-simple](/images/github-actions/overview-actions-simple.webp)

#### workflow (工作流程)

workflow 是可配置的自动化流程，可以运行一个或多个job。workflow 由仓库 `.github\workflows` 目录下的 yaml 文件定义。每个项目可以有一个或多个workflow。下面是一个简单的webpack构建的流程：

```
name: webpack

on:
  workflow_dispatch:
  push:
    tags:
      - '*'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        id: checkout
        uses: actions/checkout@v4

      - name: Build
        id: build
        run: |
          npm install
          npm run build
```

Workflow文件语法特别多也比较复杂，详细参考官方文档 [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions)，一般我们需要使用下面的几个字段的定义。

#### (1) name

`name` workflow的名称，可为空默认是workflow文件的名称，配置示例如下：

```
name: 'workflow name'
```

#### (2) on

`on` 指定触发workflow的条件，配置示例如下：

```
on:
  workflow_dispatch:
  push:
    tags:
      - '*'
```

上面的示例表示 手动触发(workflow_dispatch) 和 任意tag push的时候触发，更多触发条件，可以参考官方的文档 [Triggering a workflow](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/triggering-a-workflow)

#### (3) jobs

##### 1) jobs.<job_id>.name

`name` 表示任务的名称，可以不填

```
jobs:
    job1:
        name: job1
    job2:
        name: job2
```

##### 2) jobs.<job_id>.needs

needs 指定当前任务的依赖关系，可以通过设置needs来控制job的运行顺序。示例代码如下：

```
jobs:
    job1:
        name: job1
    job2:
        name: job2
        needs: job1
    job3:
        name: job3
        needs: [job1, job2]
```

上面代码表示任务按照 job1 > job2 > job3 的顺序执行。

##### 3) jobs.<job_id>.runs-on

runs-on 必填字段，表示当前指定的运行环境。目前支持的环境如下：

- ubuntu-latest 或  ubuntu-20.04、ubuntu-22.04
- windows-latest 或  windows-2019、windows-2022
- macos-latest 或  macos-14、macos-13

更多配置和环境说明可以参考官方文档。[About GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners)

不同的任务可以使用不同环境，比如部分跨平台的项目可以在各自平台的宿主容器上面单独编译，示例代码如下：

```
jobs:
  job1:
    runs-on: ubuntu-latest

  job2:
    runs-on: windows-latest

  job3:
    runs-on: macos-latest
```

#####  4) jobs.<job_id>.steps

steps 指定每个job下运行的步骤，可以有一个或多个step。step参数定义：

- jobs.<job_id>.steps.name: 步骤名称
- jobs.<job_id>.steps.id: 步骤ID
- jobs.<job_id>.steps.uses: 使用的action，如actions/checkout@v4，表示用户actions的checkout 库 的v4 tag
- jobs.<job_id>.steps.env: 步骤需要的环境变量
- jobs.<job_id>.steps.run: 步骤上运行的命令
- jobs.<job_id>.steps.with: action需要的参数

示例代码如下：

```
jobs:
  build:
    runs-on: ubuntu-latest

    env:
      FIRST_NAME: Mona

    steps:
      - name: Checkout
        id: checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        id: setup-node
        uses: actions/setup-node@v4
        with:
          node-version-file: .node-version
          cache: npm

      - name: My first step
        uses: actions/hello_world@main
        with:
          first_name: Mona
          middle_name: The
          last_name: Octocat

      - name: Print a greeting
        env:
          MY_VAR: Hi there! My name is
          FIRST_NAME: Mona
          MIDDLE_NAME: The
          LAST_NAME: Octocat
        run: |
          echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```

### 定制 GitHub Actions

GitHub 官方提供了一个插件市场(Marketplace)，可以在里面找到其他人开发的actions，基本上你需要的功能都可以在上面找到。

![github-marketplace](/images/github-actions/github-marketplace.png)

但是有些时候没有合适的action我们可以考虑创建自己的action。GitHub官方也非常贴心提供了一个体验课程[《Creating actions》](https://docs.github.com/en/actions/sharing-automations/creating-actions)开发一个专属的action。

目前支持创建 Docker/Javascript/Composite Actions 三种 action, 下面以一个实际的例子来介绍如何开发一个Javascript action。

#### 1、创建

GitHub 官方提供两个模板工程 [typescript-action](https://github.com/actions/typescript-action)、[javascript-action](https://github.com/actions/javascript-action)，在创建的时候可以选择。

![action-template](/images/github-actions/action-template.png)

点右上角的 `Use this template` 可以快速创建一个项目仓库

![create-template-repo](/images/github-actions/create-template-repo.png)

#### 2、开发

需要开发一个action实现将编译产物上传到upyun的功能，要求支持上传多个文件。使用upyun官方提供的npm库实现。

```
npm install upyun --save
npm install @types/upyun --save-dev
```

(1) 定义 action 描述文件

每个 action 的输入输出定义在 action.yml 文件中，下面是示例代码：

```
name: 'upyun-deploy-action'
description: 'GitHub Actions upyun deploy'
author: 'zengjing'

# Define your inputs here.
inputs:
  path:
    description: 'input path parameters'
    required: true

# Define your outputs here.
outputs:
  result:
    description: 'output result'

runs:
  using: node20
  main: dist/index.js
```

(2) 获取输入参数

```
import * as core from '@actions/core'

export interface UploadInputs {
  srcPath: string
  destPath: string
}

export function getInputs(): UploadInputs[] {
  const path = core.getInput(Inputs.Path, { required: true })
  const lines = path.split(/[\s\n]/)
  const inputs: UploadInputs[] = []
  for (const line of lines) {
    const items = line.split('->')
    if (items.length === 2) {
      const srcPath: string = items[0].trim()
      const destPath: string = items[1].trim()
      const input = { srcPath: srcPath, destPath: destPath } as UploadInputs
      inputs.push(input)
    }
  }
  return inputs
}
```

(3) 上传文件

```
import * as core from '@actions/core'

export interface UploadOutputs {
  srcPath: string
  destPath: string
  result: boolean
}

export function uploadArtifacts(inputs: UploadInputs[]): UploadOutputs[] {
    const results: UploadOutputs[] = []
    for (const input of inputs) {
        core.info(`Start to upload artifact from ${input.srcPath} to ${input.destPath}`)
        try {
            const result = await uploadArtifact(input.srcPath, input.destPath)
            results.push({ srcPath: input.srcPath, destPath: input.destPath, result: result })
        } catch (e) {
            results.push({ srcPath: input.srcPath, destPath: input.destPath, result: false })
        }
    }
    return results
}
```

(4) 输出结果

```
import * as core from '@actions/core'

// Set outputs for other workflow steps to use
core.setOutput('result', JSON.stringify(results))
```

（5）发布

```
npm run bundle
```

打包完成后提交代码，项目根目录下面有个 `script\release` 这个脚本可以创建tag并发布。

#### 3、使用

因为上传到upyun需要三个配置 service/operator/password，在代码里面是通过环境变量获取的

需要先在项目设置里面配置 （Settings -> Security -> Secrets and variables -> Actions），点 `New repository secret`添加Secrets。

![github-secrets](/images/github-actions//github-secrets.png)

在workflow添加step：

```
- name: Upload
  uses: hhtczengjing/upyun-deploy@v1
  env:
   UPYUN_SERVICE_NAME: ${{ secrets.UPYUN_SERVICE_NAME }}
   UPYUN_OPERATOR: ${{ secrets.UPYUN_OPERATOR }}
   UPYUN_PASSWORD: ${{ secrets.UPYUN_PASSWORD }}
  with:
   path: |
      version.json->/versions/products
      package.json->/versions/products
```

最终的代码在：https://github.com/hhtczengjing/upyun-deploy.git

### 参考资料

- [1、GitHub Actions 入门教程](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

- [2、Quickstart for GitHub Actions](https://docs.github.com/en/actions/writing-workflows/quickstart)