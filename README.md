# Amazon Product Search → Lark Base + Dashboard

通过 [Sorftime MCP](https://www.sorftime.com) 搜索亚马逊产品数据，自动创建 [飞书多维表格](https://www.feishu.cn/product/bitable) 并生成可视化仪表盘。

一句话完成：**亚马逊选品调研 → 数据采集 → 飞书报表 → 可视化仪表盘**，全程由 AI Agent 自动执行。

## 效果展示

输入一句提示词：
> 帮我搜索亚马逊上 wireless earbuds 的热销产品，生成飞书多维表格和仪表盘

Agent 自动完成：
- 从美国亚马逊采集 50 条热销产品数据
- 创建飞书多维表格（含产品名称、价格、销量、评分、品牌等 12 个字段）
- 创建仪表盘（3 个统计卡片 + 品牌销量柱状图 + 配送方式饼图 + 价格评论散点图）
- 返回飞书链接，打开即可查看

## 支持的站点

美国 (US) | 英国 (GB) | 德国 (DE) | 法国 (FR) | 日本 (JP) | 加拿大 (CA) | 西班牙 (ES) | 意大利 (IT) | 墨西哥 (MX) | 阿联酋 (AE) | 澳大利亚 (AU) | 巴西 (BR) | 沙特 (SA) | 印度 (IN)

---

## 安装教程（从零开始）

### 前提条件

- **Node.js v18+**：[下载地址](https://nodejs.org)
- **Anthropic 账号**：用于 Claude Code 登录
- **Sorftime 账号**：用于获取亚马逊数据 API Key
- **飞书企业账号**：用于创建飞书应用和多维表格

### 第一步：安装 Claude Code

```bash
# 安装
npm install -g @anthropic-ai/claude-code

# 验证
claude --version

# 首次登录（按提示完成 OAuth 认证）
claude
```

> 详细文档：https://docs.anthropic.com/en/docs/claude-code

### 第二步：配置 Sorftime MCP Server

1. 访问 [Sorftime 官网](https://www.sorftime.com) 注册并获取 API Key

2. 在终端中添加 MCP Server：

```bash
claude mcp add sorftime -- npx -y @anthropic-ai/mcp-remote@latest https://mcp.sorftime.com/mcp?key=你的API_KEY
```

或手动编辑 `~/.claude/claude_desktop_config.json`：

```json
{
  "mcpServers": {
    "sorftime": {
      "command": "npx",
      "args": [
        "-y",
        "@anthropic-ai/mcp-remote@latest",
        "https://mcp.sorftime.com/mcp?key=你的API_KEY"
      ]
    }
  }
}
```

3. 验证 — 在 Claude Code 中输入：

```
帮我查一下 wireless earbuds 在美国亚马逊的搜索量
```

返回搜索量数据即表示配置成功。

### 第三步：安装和配置飞书 lark-cli

1. 安装 lark-cli：

```bash
npm install -g @nicepkg/lark-cli

# 验证
lark-cli --version
```

2. 创建飞书应用并获取凭证：

   - 访问 [飞书开放平台](https://open.feishu.cn) → 创建企业自建应用
   - 在「凭证与基础信息」获取 **App ID** 和 **App Secret**
   - 在「权限管理」添加以下权限：

   | 权限 | 用途 |
   |------|------|
   | `bitable:app` | 多维表格读写 |
   | `bitable:app:readonly` | 多维表格只读 |
   | `drive:drive` | 云空间读写（创建 Base） |

   - 发布应用并等待审批通过

3. 初始化和登录：

```bash
# 输入 App ID 和 App Secret
lark-cli config init

# 浏览器 OAuth 认证
lark-cli auth login
```

### 第四步：安装本 Skill

**方法一：手动复制到 skills 目录（推荐）**

```bash
# 克隆仓库
git clone https://github.com/luotwo/amazon-product-search.git /tmp/amazon-product-search

# 创建 skill 目录并复制
mkdir -p ~/.claude/skills/amazon-product-search
cp /tmp/amazon-product-search/SKILL.md ~/.claude/skills/amazon-product-search/
cp /tmp/amazon-product-search/_meta.json ~/.claude/skills/amazon-product-search/
```

**方法二：在仓库目录中直接使用**

```bash
git clone https://github.com/luotwo/amazon-product-search.git
cd amazon-product-search
claude   # 启动 Claude Code，Skill 自动加载
```

### 第五步：验证安装

在 Claude Code 中输入：

```
帮我搜索亚马逊上 wireless earbuds 的热销产品，生成飞书多维表格和仪表盘
```

**预期结果：**
- Sorftime 返回约 50 条产品数据
- 飞书自动创建多维表格 + 仪表盘
- 终端输出飞书多维表格的访问链接

---

## 使用示例

### 最简单 — 一句话搞定
```
帮我搜索亚马逊上 wireless earbuds 的热销产品，生成飞书多维表格和仪表盘
```

### 带筛选条件
```
搜索美国亚马逊上 yoga mat 相关产品，价格 20-50 美元，月销量大于 1000，生成飞书多维表格和仪表盘
```

### 指定品牌
```
帮我分析亚马逊上 Anker 品牌的充电宝产品，生成飞书报表和仪表盘，我想看各产品的销量、价格和评分对比
```

### 其他国家站点
```
搜索日本亚马逊上保温杯的产品数据，生成飞书多维表格和分析仪表盘
```

### 完整选品调研
```
我想做亚马逊选品调研，请帮我：
1. 搜索 phone case 关键词的热销产品（前50个）
2. 把产品数据写入飞书多维表格（包含名称、价格、销量、评分、品牌等）
3. 创建仪表盘，展示品牌销量对比、价格分布、配送方式占比
```

---

## 生成的多维表格字段

| 字段 | 类型 | 说明 |
|------|------|------|
| 产品名称 | text | 产品标题 |
| ASIN | text | 亚马逊产品唯一标识 |
| 品牌 | text | 品牌名称 |
| 价格(USD) | number | 当前售价 |
| 月销量 | number | 预估月销量 |
| 月销额(USD) | number | 预估月销售额 |
| 评分 | number | 用户平均星级 |
| 评论数 | number | 用户评论总数 |
| 上架时间 | text | 产品首次上架日期 |
| 配送方式 | select | FBA / FBM |
| 类目排名 | number | 细分类目销量排名 |
| 卖家 | text | 卖家名称 |

## 生成的仪表盘组件

| 组件 | 类型 | 说明 |
|------|------|------|
| 产品总数 | 统计卡片 | 采集的产品数量 |
| 平均价格 | 统计卡片 | 所有产品的平均售价 |
| 总月销量 | 统计卡片 | 所有产品月销量合计 |
| 各品牌月销量对比 | 柱状图 | 按品牌分组的销量对比 |
| 配送方式分布 | 饼图 | FBA vs FBM 占比 |
| 价格与评论数关系 | 散点图 | 价格和评论数的关联分析 |

---

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `claude: command not found` | Claude Code 未安装 | `npm install -g @anthropic-ai/claude-code` |
| Sorftime MCP 工具不可用 | MCP Server 未配置或 Key 错误 | 重新执行 `claude mcp add` 命令 |
| `lark-cli: command not found` | lark-cli 未安装 | `npm install -g @nicepkg/lark-cli` |
| lark-cli 报 `Permission denied` | 飞书应用权限不足 | 添加 `bitable:app` 和 `drive:drive` 权限 |
| 多维表格创建后无法访问 | Bot 权限限制 | 在飞书中手动分享给自己 |
| `+table-create` 报 `Unrecognized key 'property'` | `--fields` 不支持 property | Skill 已处理此问题，无需手动干预 |

## 技术架构

```
用户提示词
    ↓
Claude Code (Agent)
    ↓
┌───────────────────┐     ┌──────────────────┐
│  Sorftime MCP     │     │  飞书 lark-cli   │
│  (数据采集)        │     │  (数据存储+可视化) │
│                   │     │                  │
│  product_search   │────→│  +base-create    │
│  product_detail   │     │  +table-create   │
│  category_report  │     │  +field-create   │
│  keyword_detail   │     │  +record-upsert  │
│  ...              │     │  +dashboard-*    │
└───────────────────┘     └──────────────────┘
    ↓                         ↓
  亚马逊产品数据            飞书多维表格 + 仪表盘
```

## License

MIT
