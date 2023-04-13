## 河山的技术指南网页源代码
>该项目参考修改自 ```hexo``` 与```orange```主题，用于渲染生成[```wrm244.github.io```](https://wrm244.github.io)页面静态代码。这个仓库主要目的是备份，也可以进行拉取修改。

[![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme) ![NPM version](https://badge.fury.io/js/hexo.svg)
## Install
该仓库已配置好Github Action 会渲染部署同步到github与gitee静态代码仓库，以下是Action配置文件详细内容：
> This repository has been configured hexo with Github Action to render, deploy and sync to github and gitee static code repositories. Here are the details of the Action configuration file: 
```yml
name: Deploy

on:   
  push:
    branches:
      - main # default branch
jobs:
  build:
    runs-on: ubuntu-latest
    name: A job to deploy blog.
    steps:
    - name: Checkout
      uses: actions/checkout@v1
      with:
        submodules: true # Checkout private submodules(themes or something else).
    
    # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
    - name: Cache node modules
      uses: actions/cache@v1
      id: cache
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - name: Install Dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
    
    # Deploy hexo blog website.
    - name: Deploy
      id: deploy
      uses: sma11black/hexo-action@v1.0.3
      with:
        deploy_key: ${{ secrets.HEXO_DEPLOY_KEY}}
        user_name: wrm244  # (or delete this input setting to use bot account)
        user_email: wrm244@outlook.com  # (or delete this input setting to use bot account)
        commit_msg: ${{ github.event.head_commit.message }}  # (or delete this input setting to use hexo default settings)
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

## English
> This project is modified from hexo and orange themes, and is used to render and generate static code for wrm244.github.io page. The main purpose of this repository is backup, and it can also be pulled and modified.
## License

MIT © 河山 100%