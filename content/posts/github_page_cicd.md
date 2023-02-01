---
title: "个人博客的CICD(基于Github)实践"
date: 2023-02-01T14:28:13+08:00
summary: "用Github workflow打造快速博客构建部署工作流"
tags: ["github pages", "cicd", "blog", "hugo"]
---

[![Github Actions](/img/github-actions.png)](https://docs.github.com/en/actions)

博客的构建和部署受更新影响，一般不会很频繁，但枯燥、乏味、一成不变.

那是否有"谁"可以代劳呢, 让我们专注于文字价值的创造?

`Github Actions` 正因此而生。

## Github Actions 为何物?

---

官方这么说:

> Automate, customize, and execute your software development workflows right in your repository with GitHub Actions. You can discover, create, and share actions to perform any job you'd like, including CI/CD, and combine actions in a completely customized workflow.

简而言之:

> Github Actions可以自动化、自定义并执行软件开发工作流。

## 流程

---

听起来，似乎可以帮我们完成`博客源码`->`构建`->`部署`的枯燥工作， 不错!

我的基本想法是，`全托管`:

- 编写: 源码托管(主分支: `main` 或者 `master`)
- 构建: Github Actions
- 部署: Github Pages, 编译后的静态文件(gh-pages分支)

### 新建发布分支

将主分支作为源码分支, **新建空分支** `gh-pages` 作为编译后的静态文件托管分支，即 Pages 的发布分支

```bash
> git checkout --orphan gh-pages # 新建空分支
> git rm -rf . # 清空文件
> git add . 
> git commit -m "init gh-pages"
> git push origin gh-pages # 提交分支
```

这里的 **空分支** 的做法非常漂亮，但不是必须. 它将脱离任何分支，独立地重头开始自己的"历史"(`git log`查看)。

### 配置Github Pages

#### 进入博客仓库的设置页面

 ![github setting](/img/gh-pages-setting.png)

#### 发布分支和目录

在 `Pages` 选项中，将发布分支选择 `gh-pages`, 并将 `/(root)` 作为发布目录

 ![github page deploy](/img/gh-pages-deploy-setting.png)

#### 强烈建议开启 `HTTPS`

 ![github page https](/img/gh-pages-https.png)

### 编写Workflow文件

#### 新建 workflow

 ![github new workflow](/img/gh-actions-new-workflow.png)

 并搜索 `Hugo` 模板:

 ![github hugo template](/img/workflow-template.png)

#### 修改workflow文件

新增构建产物的推送, 将`hugo.yml`最后的`deploy`job删除后，在`build` job 末尾追加如下step:

```yaml
- name: Deploy
  uses: JamesIves/github-pages-deploy-action@v4
  with:
    folder: public
    token: ${{ secrets.ACCESS_TOKEN }}
    branch: gh-pages
    git-config-name: gh-action
    git-config-email: gh-action@github.com
```

触发条件修改成特定tag(如: `v0.0.1`)触发, 这样我们可以控制博客何时发布，有时一篇文章处于`draft`状态会很久:

```yaml
on:
  push:
    tags:
      - "v*"
```

保存，提交改动。

#### 配置环境变量ACCESS_TOKEN

因为要推送代码到仓库分支，所以需要我们提供 `ACCESS_TOKEN`([这里生成](https://github.com/settings/tokens)) 给这个action, 设置方法也很简单:

 ![repo-secrets](/img/repo-secrets.png)

## 效果

---

至此，所有配置工作已经完成。
现在，我们来试试是否正常工作吧!

### 修改README.md

```bash
> git checkout origin main # 或者 master
> # 修改 README.md
> git add .
> git commit -m 'Modify README'
> git push origin main
> git tag v0.0.1 && git push origin v0.0.1
```

### 查看构建情况

- 切到 `Actions` 标签页，看到 `v0.0.1` 已经运行成功。

 ![workflow-run](/img/workflow-run.png)

- 点击可以看到整个workflow运行的全部step:

 ![workflow-job-run](/img/workflow-job-run.png)

- 来看看`gh-pages`的commit

 ![gh-pages-auto-push](/img/gh-pages-auto-push.png)

## 结语

通过使用 Github CI/CD服务，我们可以轻松的达到自动化构建部署博客的能力，
除此之外我们还免费享受到了**内容托管**, **HTTPS**, **域名服务**, 让我们排除一切干扰, 专注于价值创造吧!
