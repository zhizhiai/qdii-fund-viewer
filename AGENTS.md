# AGENTS.md — Codex 接手开发指南

## 项目概述

这是一个纯前端 QDII 基金查询工具，所有功能集中在单个 `index.html` 文件中（HTML + CSS + JS），无构建步骤、无后端、无框架依赖。

## 技术约束

- **无构建工具**：不使用 Webpack/Vite/Babel，代码必须是浏览器原生可运行的 ES5/ES6
- **无框架**：不使用 React/Vue/Angular，使用原生 DOM 操作
- **单文件架构**：`index.html` 包含全部 HTML 结构、`<style>` 样式、`<script>` 逻辑
- **JSONP 而非 fetch**：天天基金实时净值接口（`fundgz.1234567.com.cn`）使用 JSONP，回调名固定为 `jsonpgz`，不可自定义
- **跨域限制**：东方财富 PC 端接口（`rankhandler.aspx`、`FundArchivesDataService.aspx`）有 Referer 校验，浏览器无法直接调用；移动端 API（`fundmobapi.eastmoney.com`）可 fetch 调用

## 核心数据结构

### FUND_DB（硬编码基金数据库，第 257-280 行）

```javascript
{
  code: "012920",           // 6位基金代码
  short_name: "...",        // 短名称（用于榜单卡片）
  full_name: "...",         // 基金全称（用于查询详情）
  fund_type: "QDII-混合偏股",  // 基金类型
  one_year_return: "200.25",    // 近一年收益率（字符串，百分比数值）
  limit_summary: "单日累计购买上限20元",  // 限购摘要（用于卡片展示）
  limit_amount: 20,         // 限购金额数值（-1表示开放申购，用于排序）
  trading_status: "...",    // 交易状态
  scale: "37.05亿元（2026-03-31）",  // 基金规模（含日期）
  benchmark: "...",          // 业绩基准
  fund_url: "https://..."   // 天天基金档案链接
}
```

- 数组按 `one_year_return` 降序排列
- 共 22 只基金，筛选条件：QDII 类型 + 名称含纳指/标普/海外科技/全球成长/新兴市场/AI/硬科技/芯片/互联网
- **需定期手动更新**（建议每周）

### navCache（实时净值缓存）

```javascript
{
  "012920": {
    fundcode: "012920",
    name: "...",
    dwjz: "1.234",    // 单位净值
    gsz: "1.235",     // 估算净值
    gszzl: "0.08",    // 估算涨跌幅(%)
    gztime: "2026-06-13 15:00"  // 估算时间
  }
}
```

## 外部接口

| 接口 | 用途 | 调用方式 | 回调/字段 |
|------|------|----------|-----------|
| `fundgz.1234567.com.cn/js/{code}.js` | 实时净值 | JSONP（动态 `<script>`） | 固定回调 `jsonpgz(data)` |
| `fund.eastmoney.com/pingzhongdata/{code}.js` | 基金详情（名称、收益率、规模） | `<script>` 注入 | 全局变量 `fS_name`、`syl_1n`、`Data_fluctuationScale` |
| `fundmobapi.eastmoney.com/FundMNewApi/FundMNNBasicInformation` | 基金基本信息（业绩基准、限购） | `fetch()` | JSON: `Datas.BENCH`、`Datas.MAXSG`、`Datas.ENDNAV`、`Datas.SGZT` |

## 功能模块

### 1. 榜单渲染（第 408-467 行）

- `renderTopList()`：按近一年收益排序，渲染全部 22 只
- `renderLimitTopList()`：按限购金额排序，`.slice(0, 20)` 截取前 20
- 卡片包含：排名序号、基金名、代码/类型、业绩基准、近一年收益、限购标签

### 2. 基金代码查询（第 553-645 行）

- 清单内基金：直接读取 `FUND_DB` 硬编码数据
- 非清单基金：并行调用 `fetchPingZhongData()` + `fetchFundBasicInfo()`，合并展示

### 3. 实时净值加载（第 344-406 行）

- `fetchNavJsonp(code)`：单只基金 JSONP 加载
- `fetchAllNav(codes, onProgress)`：串行批量加载（每只间隔 300ms 防限流）
- 当前榜单未使用实时净值显示，但接口保留备用

## 代码源规则

**GitHub main 分支是唯一可信代码源，优先于本地文件。**

1. 所有人工修改优先提交到 GitHub
2. 若 GitHub 出现新提交，必须先同步到当前项目再继续开发
3. 不得覆盖 GitHub 中较新的代码
4. 若检测到冲突，提示冲突文件及解决建议
5. 发布（CloudStudio 部署）前以 GitHub 最新版本为准

同步流程：`git fetch origin` → 对比差异 → `git rebase origin/main` → 再开始修改

## 修改注意事项

1. **FUND_DB 更新**：修改基金数据后需同步更新 `DATA_VERSION`（第 282 行）和底部"榜单数据截止日期"
2. **新增筛选关键词**：在页面说明文字（第 215 行）和此文档中同步更新
3. **CSS 变量**：所有颜色通过 `:root` 变量定义（第 8-19 行），修改时保持一致性
4. **响应式断点**：`@media (max-width: 780px)`（第 182 行）
5. **escapeHtml 函数**：必须使用 `function(ch)` 语法而非箭头函数，因为模板字符串中的箭头函数会导致 HTML 属性上下文解析错误
6. **Git 推送**：本机代理无法访问 GitHub，推送时需清空代理环境变量：`ALL_PROXY="" HTTPS_PROXY="" HTTP_PROXY="" git push`

## 部署

- **CloudStudio**：通过 WorkBuddy 的 `workbuddy_cloudstudio_deploy` 工具部署，URL 为 `https://61f0f3e4f1a54521986346db9515cc42.app.codebuddy.work`
- **GitHub Pages**：可选，在仓库 Settings → Pages 中启用
