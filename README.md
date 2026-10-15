# 紫野的笔记本

紫野的线上笔记本，基于 [Hugo](https://gohugo.io/) + [Hudocs](https://github.com/zkreations/hudocs) 主题构建，使用 GitHub Actions 自动部署到 GitHub Pages。

## 目录结构

```
content/docs/     笔记内容（Markdown，按分类目录组织，自动生成侧边栏）
  Linux/
  ROS/
  3d打印/
  嵌入式Linux/
  嵌入式硬件/
  嵌入式软件/
  杂项/
  编程语言与算法/
  软路由折腾/
static/           静态资源
  images/         笔记图片
  media/          站点媒体文件
i18n/zh.toml      中文本地化
themes/hudocs/    Hudocs 主题
hugo.yaml         站点配置
.github/workflows/hugo.yaml   自动部署工作流
```

## 本地预览

```shell
hugo server
```

浏览器打开 http://localhost:1313/nobody_notebook/ 即可预览。

## 撰写新笔记

在 `content/docs/` 对应分类目录下新建 Markdown 文件，文件头部需包含以下 frontmatter（标题会自动提取自第一个一级标题，也可手动指定）：

```yaml
---
title: '笔记标题'
---
```

- **新分类**：新建目录并添加 `_index.md`（含 `title` 和 `weight`，`weight` 控制侧边栏排序）
- **图片引用**：将图片放入 `static/images/`，在笔记中用 `img` 短代码引用（会自动带上站点子路径，兼容本地预览和 Pages 部署）：

```markdown
{{< img "/images/图片文件名.png" "图片描述" >}}
```

推送 `main` 分支后，GitHub Actions 会自动构建并部署。