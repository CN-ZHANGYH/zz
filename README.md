[English](./README.en.md) | 中文

# W1sh-Sec知识库

<a href="http://creativecommons.org/licenses/by-sa/4.0/" target="_blank">
    <img src="https://img.shields.io/badge/文章%20License-CC%204.0%20BY--SA-blue.svg">
</a>
<a href="https://gitlab.com/sewiki/w1sh-sec/main" target="_blank">
    <img src="https://img.shields.io/badge/源码%20License-MIT-blue.svg">
</a>
<a href="https://gitlab.com/sewiki/w1sh-sec/main" target="_blank">
    <img src="https://github.com/Charles7c/charles7c.github.io/actions/workflows/deploy-pages.yml/badge.svg">
</a>


📝 **这个项目是网络安全的技术知识库，用于记录和分享个人在安全领域的碎片化、结构化、体系化的技术知识内容。主要内容涵盖了 Web 渗透测试、逆向安全等多个方面的内容，旨在为安全从业者、爱好者、学生和研究者提供丰富的学习资源和知识分享平台。**

🐢 [GitHub Pages（完整体验）](https://w1sh.drill.wiki)

## 开始

```bash
# 1.克隆本仓库
git clone https://gitlab.com/sewiki/w1sh-sec.git
# 2.安装 PNPM
npm install pnpm -g
# 3.设置淘宝镜像源
pnpm config set registry https://registry.npmmirror.com/
# 4.安装依赖
pnpm install
# 5.dev 运行，访问：http://localhost:5173
pnpm dev
# 6.打包，文件存放位置：docs/.vitepress/dist
# 如果是部署到 GitHub Pages，可以利用 GitHub Actions，在 push 到 GitHub 后自动部署打包
# 详情见：.github/workflows/deploy-pages.yml，根据个人需要删减工作流配置
pnpm build
# 7.部署
# 7.1 push 到 GitHub 仓库，部署到 GitHub Pages：需要在仓库设置中启用 GitHub Pages（本仓库采用此种部署方式）
# 7.2 在其他平台部署, 例如：Gitee Pages、Vercel、Netlify、个人虚拟主机、个人服务器等
```



## 已扩展功能（持续优化细节）

**已扩展功能（持续优化细节）**

- **拆分配置文件：**  解决“大”配置文件问题，提取公有配置选项进行复用，方便维护。
- **GitHub Actions：**  自动进行项目打包及 GitHub Pages 部署，并同步到 Gitee Pages。
- **自动生成侧边栏：** 将文章按规律性目录存放后，侧边栏将自动生成，支持文章置顶🔝。
- **主页美化：** 参照 vite 文档主页进行美化。
- **自定义页脚：** 支持ICP备案号、公安备案号、版权信息配置。
- **文章元数据信息显示：**  显示是否原创、作者、发布时间、所属分类、标签列表等信息。
- **已扩展文章阅读数信息：**  默认已启用，可在配置中关闭。
- **《我的标签》：** 模仿语雀标签页风格，另有标签云展示。
- **《我的归档》：** 自定义时间轴，展示历史文章数据。
- **文章评论：** 目前仅支持Gitalk。
- **版权声明：** 文末显示原创或转载文章的版权声明，可自由配置采用的版权协议。
- **徽章：** 标题后可显示徽章。
- **本地文档搜索支持：** 支持本地文档搜索插件。
- **Mermaid 流程图：** 在 Markdown 中绘制流程图、状态图、时序图、甘特图、饼图等。
- **Markdown 脚注、Markdown 公式支持**。
- **更多细节优化。**




## 首页

### 主页页面

![主页](./docs/public/screenshot/主页.jpg)


## 特别鸣谢
- Mu-cream 作者贡献
- Seso 作者贡献

## License

- 文章遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和声明
- 源码遵循 [MIT](https://github.com/Charles7c/charles7c.github.io/blob/main/LICENSE) 许可协议
- Copyright © 2019-2022 Charles7c