# 盐 碱 地

使用 Hexo 8、NexT 8 和 Node.js 26。构建使用的 Node.js 补丁版本见 `.nvmrc`；主题由 npm 安装，无需初始化 Git 子模块。

```sh
npm ci
npm run server
```

在 `source/_posts/` 编写文章，配图放在同名目录。文章链接由发布日期和文件名生成；修改文章标题时可只改 Markdown 的 `title`，保留已有 URL。

站点配置位于 `_config.yml`，主题覆盖配置位于 `_config.next.yml`。头像与自定义样式分别放在 `source/images/` 和 `source/_data/`。

GitHub Actions 会在源码分支 `hexo` 上验证安装和构建。发布前先本地预览，再按现有流程手动部署：

```sh
npm run clean
npm run build
npm run deploy
```

部署命令会把静态文件推送到 GitHub Pages 使用的 `master` 分支。
