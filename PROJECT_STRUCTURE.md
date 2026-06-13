# PROJECT_STRUCTURE.md — 项目结构与架构说明

## 目录结构

```
qdii-fund-viewer/
├── index.html              # 主页面（单文件应用，包含全部代码）
├── README.md               # 项目说明
├── AGENTS.md               # Codex 开发指南
├── PROJECT_STRUCTURE.md    # 本文件
├── .gitignore              # Git 忽略规则
└── .workbuddy/             # WorkBuddy 项目文件（已加入 .gitignore）
    └── memory/             # 工作日志和项目记忆
```

## index.html 内部结构

整个应用是单文件架构，`index.html` 包含 684 行代码，按以下区域划分：

### HTML 结构（第 1-253 行）

```
<html>
  <head>
    <style>           → 第 7-198 行：全部 CSS 样式
  </head>
  <body>
    <header>          → 第 200-210 行：页面标题 + 3 个导航按钮
    <main>
      <section#sectionTop>      → 第 213-220 行：业绩前20榜单区域
      <section#sectionLimitTop> → 第 222-229 行：限购额度前20榜单区域
      <section#sectionQuery>    → 第 231-246 行：基金代码查询区域
      <section>                 → 第 248-252 行：数据来源与截止日期
    </main>
    <script>          → 第 255-682 行：全部 JavaScript 逻辑
  </body>
</html>
```

### 样式位置（第 7-198 行）

所有样式通过 `<style>` 标签内联，不使用外部 CSS 文件。

| CSS 变量 | 用途 | 定义行 |
|----------|------|--------|
| `--ink` | 正文颜色 | 第 9 行 |
| `--muted` | 辅助文字颜色 | 第 10 行 |
| `--line` | 边框颜色 | 第 11 行 |
| `--navy` | 主色调（深蓝） | 第 12 行 |
| `--gold` | 强调色 | 第 13 行 |
| `--bg` | 页面背景色 | 第 14 行 |
| `--card` | 卡片背景色 | 第 15 行 |
| `--risk` | 风险提示色 | 第 16 行 |
| `--watch` | 关注提示色 | 第 17 行 |
| `--ok` | 安全提示色 | 第 18 行 |

**响应式断点**：`@media (max-width: 780px)` 在第 182 行，主要调整：
- 单列布局替代双列
- 字体和间距缩小
- 搜索区域改为纵向排列

### JavaScript 逻辑（第 255-682 行）

#### 数据区

| 变量/常量 | 行号 | 说明 |
|-----------|------|------|
| `FUND_DB` | 257-280 | 硬编码基金数据库（22 只基金），每周手动更新 |
| `DATA_VERSION` | 282 | 数据版本号，格式 `"YYYY-MM-DD"` |
| `navCache` | 285 | 实时净值缓存对象 `{fundcode: data}` |
| `navCallbacks` | 286 | JSONP 等待回调 `{fundcode: resolve}` |
| `pendingScripts` | 287 | 未完成的 JSONP script 标签引用 |

#### 工具函数

| 函数 | 行号 | 说明 |
|------|------|------|
| `escapeHtml(text)` | 309-312 | HTML 转义，防 XSS（必须用 `function` 语法，不能用箭头函数） |
| `link(url, label)` | 314-316 | 生成外部链接 `<a>` 标签 |
| `fmtPct(v)` | 318-323 | 格式化百分比，正数加 `+` 号 |
| `navHtml(data)` | 325-330 | 生成实时净值 HTML（含涨跌颜色） |

#### 接口调用

| 函数 | 行号 | 说明 |
|------|------|------|
| `window.jsonpgz(data)` | 290-307 | JSONP 固定回调，处理天天基金实时净值响应 |
| `updateNavInAllCards(code, data)` | 333-341 | 更新页面中所有该基金卡片的净值显示 |
| `fetchNavJsonp(code)` | 344-382 | 加载单只基金实时净值（JSONP，8 秒超时） |
| `fetchAllNav(codes, onProgress)` | 385-406 | 串行批量加载净值（每只间隔 300ms 防限流） |
| `fetchPingZhongData(code)` | 471-513 | 加载基金详情（pingzhongdata 脚本注入，10 秒超时） |
| `fetchFundBasicInfo(code)` | 516-551 | 调用东方财富移动端 API（fetch，获取业绩基准/限购/规模） |

#### 渲染函数

| 函数 | 行号 | 说明 |
|------|------|------|
| `renderTopList()` | 409-436 | 渲染业绩前 20 榜单（按 `one_year_return` 降序） |
| `renderLimitTopList()` | 438-467 | 渲染限购额度前 20 榜单（按 `limit_amount` 降序，`.slice(0,20)`） |
| `queryFund()` | 553-645 | 基金代码查询逻辑 |

#### 事件绑定与初始化

| 代码 | 行号 | 说明 |
|------|------|------|
| 示例 chips 生成 | 648-663 | 查询区域快捷按钮（159696、161128、012920） |
| `refreshTop` 按钮事件 | 666-668 | 重新渲染业绩榜单 |
| `refreshLimitTop` 按钮事件 | 670-672 | 重新渲染限购榜单 |
| `query` 按钮事件 | 674 | 触发查询 |
| `code` 输入框回车 | 675-677 | 回车触发查询 |
| 初始渲染 | 680-681 | 页面加载时渲染两个榜单 |

## 页面区域与 DOM ID 映射

| 区域 | DOM ID | 对应功能 |
|------|--------|----------|
| 业绩榜单卡片容器 | `topList` | `renderTopList()` 渲染目标 |
| 限购榜单卡片容器 | `limitTopList` | `renderLimitTopList()` 渲染目标 |
| 业绩刷新按钮 | `refreshTop` | 点击调用 `renderTopList()` |
| 限购刷新按钮 | `refreshLimitTop` | 点击调用 `renderLimitTopList()` |
| 基金代码输入框 | `code` | 6 位数字，默认值 `159696` |
| 查询按钮 | `query` | 点击调用 `queryFund()` |
| 查询结果容器 | `result` | `queryFund()` 渲染目标 |
| 示例按钮容器 | `chips` | 动态生成 3 个快捷查询按钮 |
| 业绩板块锚点 | `sectionTop` | 导航按钮平滑滚动目标 |
| 限购板块锚点 | `sectionLimitTop` | 导航按钮平滑滚动目标 |
| 查询板块锚点 | `sectionQuery` | 导航按钮平滑滚动目标 |

## 外部接口依赖

### 1. 天天基金实时净值（JSONP）

- **URL**：`//fundgz.1234567.com.cn/js/{code}.js?rt={timestamp}`
- **方式**：动态创建 `<script>` 标签
- **回调**：固定全局函数 `window.jsonpgz(data)`
- **返回数据结构**：
  ```json
  {
    "fundcode": "012920",
    "name": "易方达全球成长精选混合(QDII)人民币A",
    "dwjz": "3.1234",
    "gsz": "3.1345",
    "gszzl": "0.36",
    "gztime": "2026-06-13 15:00"
  }
  ```

### 2. 天天基金 pingzhongdata（脚本注入）

- **URL**：`//fund.eastmoney.com/pingzhongdata/{code}.js?rt={timestamp}`
- **方式**：动态创建 `<script>` 标签
- **注入的全局变量**：
  - `fS_name`：基金名称
  - `fS_code`：基金代码
  - `syl_1n` / `syl_6y` / `syl_3y` / `syl_1y`：近一年/六月/三月/一月收益率
  - `Data_fluctuationScale`：规模数据（含 `.series` 和 `.categories`）

### 3. 东方财富移动端 API（fetch）

- **URL**：`https://fundmobapi.eastmoney.com/FundMNewApi/FundMNNBasicInformation?...`
- **方式**：`fetch()` GET 请求
- **关键参数**：`FCODE`（基金代码）、`plat=Android`、`appType=ttjj`、`product=EFund`
- **返回关键字段**：
  - `Datas.BENCH`：业绩基准
  - `Datas.MAXSG`：单日限购金额
  - `Datas.ENDNAV`：基金规模（单位：元）
  - `Datas.FEGMRQ`：规模日期
  - `Datas.SGZT`：申购状态
  - `Datas.SHZT`：赎回状态
  - `Datas.FTYPE`：基金类型
  - `Datas.MINSG`：最小申购金额
  - `Datas.RISKLEVEL`：风险等级

## 已知限制

1. **东方财富 PC 端接口不可用**：`rankhandler.aspx`、`FundArchivesDataService.aspx` 等接口有 Referer 校验，浏览器端无法直接调用
2. **JSONP 回调名固定**：`fundgz.1234567.com.cn` 的回调名固定为 `jsonpgz`，不可自定义，因此同一时间只能处理一个 JSONP 请求
3. **移动端 API CORS 风险**：`FundMNNBasicInformation` 接口目前允许跨域调用，但未来可能收紧
4. **数据手动更新**：`FUND_DB` 中的近一年收益、限购金额、规模等数据需定期手动更新
