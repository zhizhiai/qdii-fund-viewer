# QDII 基金查询工具

一个纯前端的 QDII 基金查询和排行榜展示工具，无需后端服务器。

## 功能特性

- 📊 **QDII 近一年业绩前 20** 基金排行榜
- 💰 **QDII 限购额度前 20** 基金排行榜
- 🔍 **按基金代码查询** 详细信息
- 🔄 **实时数据** 通过天天基金和东方财富 API 获取
- 📱 **响应式设计** 支持移动端访问

## 技术栈

- **纯静态 HTML/CSS/JavaScript**（单文件应用）
- **数据来源**：
  - 天天基金实时净值接口（JSONP）
  - 东方财富移动端 API
  - 天天基金档案数据（pingzhongdata）

## 本地开发

直接在浏览器中打开 `index.html` 即可使用。

## 部署

### CloudStudio 部署

```bash
# 使用 WorkBuddy 的 CloudStudio 部署功能
# 部署后的访问地址：
# https://61f0f3e4f1a54521986346db9515cc42.app.codebuddy.work
```

### GitHub Pages 部署

1. 推送代码到 GitHub 仓库
2. 在仓库设置中启用 GitHub Pages
3. 选择 `main` 分支作为源

## 项目结构

```
qdii-site/
├── index.html          # 主页面（包含全部代码）
├── README.md          # 项目说明
├── .gitignore         # Git 忽略配置
└── .workbuddy/        # WorkBuddy 项目文件（不提交）
```

## 数据更新

- **榜单数据**：定期手动更新 `FUND_DB` 数组中的数据
- **实时净值**：通过 JSONP 接口实时获取
- **基金详情**：通过 API 实时查询

## 许可证

MIT License

## 贡献

欢迎提交 Issue 和 Pull Request！

---

**部署地址**：https://61f0f3e4f1a54521986346db9515cc42.app.codebuddy.work
