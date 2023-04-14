---
title: hexo插件推荐
date: 2023-04-13 01:38:29
tags: [hexo]
---
:::tips
Hexo的插件真是个好东西！挑了些比较有用的插件，拿出来做个总结，同时也是为了方便以后使用做的一个简单记录。
:::

## 插件安装命令
```shell
npm install yourplugins --save
```

## 文章加密

插件是`hexo-blog-encrypt`，使用前须在站点配置文件加入以下内容：

```yaml
encrypt:
    enable: true
```

随后在文章的开头部分加入`password:`字段设置密码即可。

同时，配合`abstract:`字段和`message:`字段可以设置对无密码人的提示信息。同时注意不要设置toc。

## 中英文自动空格

插件为`hexo-filter-auto-spacing`，`npm install --save`之后就可以用，无需手动设置。

## 快速上标

插件为`hexo-filter-sup`，站点设置：

```yaml
sup:
    markup: '^'
```

实例：

```markdown
x^2^+5=10
```

## 文本提示

插件为`hexo-tag-hint`，实例：

```markdown
大家好，这个博客用了{% hint 'Hexo' '一个静态博客框架' %}。
```

注意字符串中有单引号的时候加反斜线转义。

## 下拉抽屉

插件为`hexo-tag-details`，实例：

```markdown
{% details mode:close 怎样才能订阅你博客的更新？ %}
    订阅RSS啊！
{% enddetails %}
```

## 脚注

插件为`hexo-footnotes`，实例：

```markdown
我真的喜欢读《三体》[^1]
[^1]: 作者为 刘慈欣。
```

支持多行注释和Markdown注释。

## PDF 文件

插件为`hexo-pdf`，实例：（Modeling Singing F0 With Neural Network Driven Transition-Sustain Models - By Kanru Hua）

```markdown
{% pdf./Modeling-Singing-F0-With-Neural-Network-Driven-Transition-Sustain-Models.pdf %}
```