---
title: hexo利用github action进行部署
date: 2023-04-14 01:38:29
tags: [hexo]
---
## GitHub Actions

GitHub Actions 的工作原理：我们提前设置好需要自动化执行的任务，GitHub Actions 监控当前仓库的某一个操作（如：push），一旦有此操作，就自动化执行这些任务。

所以我们希望使用 GitHub Actions 后，只需要往源文件仓库 push 更新源文件，GitHub Actions 监控到 push 操作时，就自动化执行 `hexo clean`、`hexo g -d` 操作，部署到自己github pages上面，完成博文发布。

Action 存放在项目根目录的 `.github/workflows` 下，后缀为 `.yml`。一个 Action 相当于一个工作流 workflow，一个工作流可以有多个任务 job，每个任务可以分为几步 step。任务、步骤依次执行。

每个 Action 是一个独立脚本，所以可以作为代码仓库。

-   `actions/setup-node` 就表示 `github.com/actions/setup-node` 这个 [仓库](https://github.com/actions/setup-node)，代表安装 node.js。Action 为 action.yml

可以通过下面这种格式来使用别人写好的 action，@借用了指针的概念：

```bash
actions/setup-node@74bc508 # 指向一个 commit
actions/setup-node@v1.0    # 指向一个标签
actions/setup-node@master  # 指向一个分支
```

现在需要实现一个 Action，使其能够执行 `hexo clean`、`hexo g -d` 操作。

以下是我个人使用的配置文件
```yml
name: Deploy

on:   
  push:
    branches:
      - main # default branch
jobs:
  build:
    runs-on: ubuntu-latest
    name: deploy hexo blog.
    steps:
    - name: Checkout
      uses: actions/checkout@v1
      with:
        submodules: true # Checkout private submodules(themes or something else).
    
    # Deploy hexo blog website.
    - name: Deploy
      id: deploy
      uses: wrm244/hexo-deploy-action@main
      with:
        deploy_key: ${{ secrets.HEXO_DEPLOY_KEY}}
        user_name: wrm244  # (or delete this input setting to use bot account)
        user_email: wrm244@outlook.com  # (or delete this input setting to use bot account)
       
    # Use the output from the `deploy` step(use for test action)
    - name: Get the output
      run: |
        echo "${{ steps.deploy.outputs.notify }}"
  
  sync-2-gitee:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Sync to Gitee
        uses: wearerequired/git-mirror-action@master
        env:
          # 注意在 Settings->Secrets 配置 GITEE_RSA_PRIVATE_KEY
          SSH_PRIVATE_KEY: ${{ secrets.GITEE_PRIVATE_KEY }}
        with:
          # 注意替换为你的 GitHub 源仓库地址
          source-repo: git@github.com:wrm244/wrm244.github.io.git
          # 注意替换为你的 Gitee 目标仓库地址
          destination-repo: git@gitee.com:wrm244/wrm244.git
```