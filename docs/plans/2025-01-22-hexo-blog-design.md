# Hexo 博客设计文档

**日期**: 2025-01-22
**项目**: Franklin爱家个人博客
**技术栈**: Hexo + Butterfly 主题 + GitHub Pages

## 1. 项目架构

博客采用 Hexo 静态站点生成器 + Butterfly 主题的架构。

### 项目结构
```
wufulin.github.com/
├── source/                    # 博客源文件
│   ├── _posts/               # Markdown 文章
│   ├── categories/           # 分类页
│   ├── tags/                 # 标签页
│   ├── links/                # 友情链接页
│   ├── about/                # 关于我页
│   └── timeline/             # 时间线页
├── themes/                    # 主题目录
│   └── butterfly/            # Butterfly 主题
├── _config.yml               # Hexo 主配置
├── _config.butterfly.yml     # Butterfly 主题配置
├── .github/workflows/        # GitHub Actions 工作流
├── package.json              # 依赖管理
└── public/                   # 生成的静态网站（不提交）
```

### 部署架构
- **源代码仓库**: main 分支
- **静态网站**: gh-pages 分支（由部署工具自动管理）
- **发布地址**: https://wufulin.github.io
- **远程仓库**: https://github.com/wufulin/wufulin.github.com

## 2. 核心配置

### Hexo 主配置 (_config.yml)
```yaml
# 网站信息
title: Franklin爱家
author: Franklin
language: zh-CN
timezone: Asia/Shanghai

# URL
url: https://wufulin.github.io
permalink: :year/:month/:day/:title/

# 部署配置
deploy:
  type: git
  repo: https://github.com/wufulin/wufulin.github.com.git
  branch: gh-pages
```

### Butterfly 主题配置 (_config.butterfly.yml)

#### 核心功能
- **主题版本**: latest
- **代码高亮**: MacOS 风格 (`highlight_theme: mac`)
- **Mermaid 支持**: 启用 (`mermaid: enable: true`)
- **Timeline 支持**: 启用 (`timeline: enable: true`)

#### 菜单结构
```yaml
menu:
  首页: / || fas fa-home
  文章: /archives/ || fas fa-archive
  分类: /categories/ || fas fa-folder-open
  标签: /tags/ || fas fa-tags
  时间线: /timeline/ || fas fa-clock
  友情链接: /links/ || fas fa-link
  关于: /about/ || fas fa-heart
```

#### 评论系统
- **系统**: Twikoo
- **配置**: 需后续申请腾讯云开发环境 ID

#### 搜索功能
- **方案**: 本地搜索（使用 hexo-generator-search）

#### 侧边栏小部件
- 公告
- 最近文章
- 分类列表
- 标签云

## 3. 页面结构

### 首页
- 顶部导航栏（Logo + 菜单 + 搜索框）
- Hero 区域（个人简介/欢迎语）
- 文章卡片列表（封面、标题、摘要、日期、分类、标签）
- 分页导航
- 页脚（版权、社交链接）

### 独立页面
1. **分类页**: 展示所有分类及文章数量
2. **标签页**: 标签云展示
3. **友情链接页**: 卡片式展示（网站名、描述、头像）
4. **关于我页**: 个人信息、头像、简介、社交链接
5. **时间线页**: 按时间轴展示个人经历或重要事件

### 文章详情页
- 面包屑导航
- 文章标题和元信息（日期、分类、标签、字数、阅读时长）
- 文章内容（支持 Markdown、Mermaid 图表）
- MacOS 风格代码块（带复制按钮）
- 版权声明
- 上一篇/下一篇导航
- 相关文章推荐
- Twikoo 评论区

## 4. 部署方案

### 本地一键部署
```bash
hexo clean && hexo generate && hexo deploy
```

**流程**:
1. 清理缓存和生成文件
2. 生成静态网站到 public 目录
3. 推送到 gh-pages 分支
4. GitHub Pages 自动发布

### GitHub Actions 自动部署

**工作流文件**: `.github/workflows/deploy.yml`

**触发条件**: Push 到 main 分支

**步骤**:
1. 检出代码
2. 安装 Node.js 18.x
3. 安装依赖
4. 生成静态网站
5. 部署到 gh-pages 分支

**所需 Secrets**:
- `GITHUB_TOKEN`: 自动提供
- `GH_TOKEN`: Personal Access Token（需手动配置）

## 5. 必要插件

```json
{
  "hexo-deployer-git": "^4.0.0",
  "hexo-generator-search": "^2.4.3",
  "hexo-generator-feed": "^3.0.0",
  "hexo-generator-sitemap": "^3.0.1",
  "hexo-generator-category": "^1.0.0",
  "hexo-generator-tag": "^2.0.0",
  "hexo-renderer-marked": "^6.0.0",
  "hexo-renderer-stylus": "^2.1.0",
  "hexo-renderer-ejs": "^2.0.0"
}
```

## 6. 初始化步骤

1. 初始化 Git 仓库
2. 安装 Hexo CLI: `npm install -g hexo-cli`
3. 初始化 Hexo 项目
4. 安装 Butterfly 主题
5. 安装必要插件
6. 创建主题配置文件
7. 配置 _config.yml 和 _config.butterfly.yml
8. 创建必要的页面
9. 推送到 GitHub

## 7. 后续配置清单

- [ ] 申请 Twikoo 腾讯云开发环境并配置环境 ID
- [ ] 配置搜索功能索引
- [ ] 创建 GitHub Personal Access Token（用于 Actions）
- [ ] 自定义主题配色（可选）
- [ ] 添加社交链接
- [ ] 编写第一篇博文
- [ ] 配置自定义域名（可选）

## 8. 设计原则

- **YAGNI**: 只实现必要功能，避免过度设计
- **简洁优先**: 使用默认主题风格，减少定制化
- **自动化**: 配置 GitHub Actions 实现自动部署
- **文档化**: 记录所有配置和步骤
