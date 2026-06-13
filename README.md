# QDII 基金查询工具

纯前端 QDII 基金排行榜与查询工具，无需后端服务器，单个 HTML 文件即可运行。

## 功能特性

- **QDII 近一年业绩前 20**：按近一年收益率降序排列，筛选纳指、标普、海外科技、全球成长、新兴市场、AI/硬科技等相关 QDII 产品
- **QDII 限购额度前 20**：按单日购买上限从大到小排列，方便了解哪些基金限购较松
- **按基金代码查询**：输入任意 6 位基金代码，实时获取基金规模、业绩基准、限购金额、收益率等信息
- **响应式设计**：适配桌面端和移动端

## 技术栈

| 类别 | 技术 |
|------|------|
| 前端框架 | 无（原生 HTML/CSS/JS） |
| 构建工具 | 无（直接打开 HTML） |
| 数据来源 | 天天基金 JSONP + 东方财富移动端 API |
| 部署方式 | CloudStudio 静态站点 |

## 项目启动

```bash
# 方式一：直接在浏览器打开
open index.html

# 方式二：本地 HTTP 服务（可选，JSONP 在 file:// 协议下也可正常工作）
python3 -m http.server 8080
# 然后访问 http://localhost:8080
```

## 部署

### CloudStudio（当前使用）

通过 WorkBuddy 的 CloudStudio 部署工具一键部署：

- **线上地址**：https://61f0f3e4f1a54521986346db9515cc42.app.codebuddy.work
- **部署方式**：`workbuddy_cloudstudio_deploy`（指定 `qdii-site/` 目录）

### GitHub Pages（可选）

1. 推送代码到 GitHub 仓库 `zhizhiai/qdii-fund-viewer`
2. 在仓库 Settings → Pages 中启用，选择 `main` 分支
3. 访问地址将为 `https://zhizhiai.github.io/qdii-fund-viewer/`

## 数据来源与更新

| 数据类型 | 更新频率 | 来源 |
|----------|----------|------|
| 榜单排名 | 每周手动更新 `FUND_DB` | 东方财富 QDII 排行 + 天天基金档案 |
| 基金规模 | 每周手动更新 `FUND_DB` | 天天基金档案（季度报告） |
| 业绩基准 | 手动更新 `FUND_DB` | 天天基金档案 |
| 限购金额 | 每周手动更新 `FUND_DB` | 基金公司公告 |
| 非清单基金查询 | 实时 | 东方财富移动端 API（`FundMNNBasicInformation`）+ `pingzhongdata` |

> **注意**：榜单数据截止日期标注在页面底部。QDII 额度和限购变化很快，最终以基金公司公告、交易软件和销售平台为准。

## 项目结构

```
qdii-fund-viewer/
├── index.html              # 主页面（含全部 HTML/CSS/JS，684 行）
├── README.md               # 本文件
├── AGENTS.md               # Codex 接手开发指南
├── PROJECT_STRUCTURE.md    # 项目结构与架构说明
├── .gitignore              # Git 忽略配置
└── .workbuddy/             # WorkBuddy 项目文件（不提交到 Git）
```

## GitHub 仓库

- **仓库地址**：https://github.com/zhizhiai/qdii-fund-viewer
- **默认分支**：`main`
- **推送命令**（需绕过本机代理）：
  ```bash
  cd qdii-site
  git add .
  git commit -m "描述你的修改"
  ALL_PROXY="" HTTPS_PROXY="" HTTP_PROXY="" https_proxy="" http_proxy="" git push
  ```

## 许可证

MIT License
