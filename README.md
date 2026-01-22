# Franklin爱家个人博客

基于 Hexo + Butterfly 主题的 GitHub Pages 个人博客。

## 技术栈

- **Hexo**: 快速、简洁且高效的静态站点生成器
- **Butterfly**: 优雅美观的 Hexo 主题
- **GitHub Pages**: 免费网站托管服务

## 功能特性

- 文章分类和标签
- 友情链接页面
- 关于我页面
- 时间线功能
- 评论系统（Twikoo）
- 搜索功能
- MacOS 风格代码高亮
- Mermaid 图表支持

## 本地开发

### 安装依赖

```bash
npm install
```

### 启动本地服务器

```bash
npm run server
```

访问 http://localhost:4000

### 构建静态网站

```bash
npm run build
```

## 部署

### 本地一键部署

```bash
npm run deploy
```

### GitHub Actions 自动部署

推送到 main 分支后，GitHub Actions 会自动构建并部署到 GitHub Pages。

## 后续配置

- [ ] 申请 Twikoo 腾讯云开发环境并配置环境 ID
- [ ] 添加自定义头像和封面图片
- [ ] 配置社交链接
- [ ] 配置搜索功能

## 项目结构

```
├── source/                    # 博客源文件
│   ├── _posts/               # Markdown 文章
│   ├── about/                # 关于我页面
│   ├── links/                # 友情链接页面
│   ├── timeline/             # 时间线页面
│   ├── categories/           # 分类页面
│   └── tags/                 # 标签页面
├── themes/butterfly/         # Butterfly 主题
├── _config.yml               # Hexo 主配置
├── _config.butterfly.yml     # Butterfly 主题配置
├── .github/workflows/        # GitHub Actions 工作流
└── package.json              # 依赖管理
```

## 相关链接

- [Hexo 文档](https://hexo.io/zh-cn/docs/)
- [Butterfly 主题文档](https://butterfly.js.org/)
- [GitHub Pages](https://pages.github.com/)

## 许可证

[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
