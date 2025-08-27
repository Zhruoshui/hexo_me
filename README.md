## ruoshui · 个人博客

写点日常、技术与碎碎念。欢迎来聊。

### 在线地址
- https://blog.aruoshui.fun/

### 效果
![主页](https://image.aruoshui.fun/i/2025/08/27/n721um-0.webp)


### 交流方式
- 在博客文章下留言即可；
- 或在本仓库提交 Issue，一起讨论想法和问题。

## 主题与插件

- 主题：hexo-theme-butterfly（已配置 _config.butterfly.yml）
- 常用插件（已在 package.json 中声明）：
  - hexo-wordcount：字数/阅读时长
  - hexo-abbrlink：固定链接优化
  - hexo-algoliasearch：站内搜索（需配置 Algolia）
  - katex：数学公式渲染
  - 以及部分 Butterfly 生态增强插件（douban、swiper、envelope、tag-plugins-plus 等）

如需个性化外观，请主要修改：
- _config.butterfly.yml（主题外观、布局、组件开关）
- source 下的页面与资源（关于页、友链、图片等）

---

## 目录结构（简版）

```
.
├─ source/            写作素材（文章、页面、图片）
│  ├─ _posts/         文章（Markdown）
│  ├─ about/          关于页等自定义页面
│  └─ img/            静态图片资源
├─ themes/            主题目录（Butterfly）
├─ public/            生成后的静态文件（build 后出现）
├─ scaffolds/         模板（post/page/draft）
├─ _config.yml        Hexo 主配置
├─ _config.butterfly.yml  主题配置
└─ package.json       脚本与依赖
```

## 关于部署
- 本仓库使用 GitHub Actions 自动构建与发布；
- 构建产物会部署到腾讯云服务器，方便在云上编辑，push 即自动触发部署。

---

## 参考与致谢
- Hexo 中文文档：https://hexo.bootcss.com/docs/configuration.html
- Butterfly 主题文档：https://butterfly.js.org/
- 以及相关的 Hexo 主题文档、Butterfly 主题文档与店长的教程等

—— 欢迎来我的博客交流，也欢迎点个 Star。
