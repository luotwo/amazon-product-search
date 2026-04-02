---
name: amazon-product-search
description: 通过 Sorftime MCP 搜索亚马逊产品数据，自动创建飞书多维表格(Base)并生成仪表盘(Dashboard)进行可视化分析。当用户需要亚马逊选品分析、产品调研、市场研究并希望结果输出到飞书时触发。
---

# Amazon 产品搜索 → 飞书多维表格 + 仪表盘

## 触发条件
当用户请求以下任务时，Agent 应主动使用本 Skill：
1. 搜索亚马逊产品并生成飞书报表
2. 亚马逊选品分析 / 市场调研并输出到飞书
3. 查询亚马逊某关键词/品类的产品数据并创建多维表格
4. 对亚马逊产品数据进行可视化分析

## 入门提示词（直接复制使用）

如果你是新手，不知道怎么开始，直接复制下面任意一句发给 Agent 即可：

### 最简单 — 一句话搞定
> 帮我搜索亚马逊上 wireless earbuds 的热销产品，生成飞书多维表格和仪表盘

### 带价格和销量筛选
> 搜索美国亚马逊上 yoga mat 相关产品，价格在 20-50 美元之间，月销量大于 1000，帮我把数据写入飞书多维表格，并创建一个可视化仪表盘

### 分析指定品牌
> 帮我分析亚马逊上 Anker 品牌的充电宝产品，生成飞书报表和仪表盘，我想看各产品的销量、价格和评分对比

### 搜索其他国家站点
> 搜索日本亚马逊上保温杯的产品数据，生成飞书多维表格和分析仪表盘

### 完整选品调研
> 我想做亚马逊选品调研，请帮我：
> 1. 搜索 phone case 关键词的热销产品（前50个）
> 2. 把产品数据写入飞书多维表格（包含名称、价格、销量、评分、品牌等）
> 3. 创建仪表盘，展示品牌销量对比、价格分布、配送方式占比

> **提示**：把上面加粗/关键词部分替换成你自己想调研的产品即可。支持的站点：US(美国)、GB(英国)、DE(德国)、FR(法国)、JP(日本)、CA(加拿大)、ES(西班牙)、IT(意大利)、MX(墨西哥)、AE(阿联酋)、AU(澳大利亚)、BR(巴西)、SA(沙特)、IN(印度)。

## 完整工作流

### 第一步：通过 Sorftime MCP 采集数据

根据用户需求选择合适的 MCP 工具组合。**默认站点为 US（美国站）**，用户可指定其他站点(GB/DE/FR/JP/CA 等)。

#### 主要数据采集工具

| 场景 | MCP 工具 | 说明 |
|------|----------|------|
| 按关键词搜索产品 | `mcp__sorftime__product_search` | **最常用**，按月销量倒序返回产品列表 |
| 查看单个产品详情 | `mcp__sorftime__product_detail` | 传入 ASIN 获取完整产品信息 |
| 类目 Top100 报告 | `mcp__sorftime__category_report` | 需先通过 `category_name_search` 获取 nodeId |
| 关键词搜索量详情 | `mcp__sorftime__keyword_detail` | 查看关键词月搜索量、CPC 等数据 |
| 搜索相关类目 | `mcp__sorftime__category_search_from_product_name` | 按产品名称查找相关细分类目市场 |
| 产品历史趋势 | `mcp__sorftime__product_trend` | 查看产品销量/价格/排名趋势 |
| 竞品关键词分析 | `mcp__sorftime__competitor_product_keywords` | 查看竞品在各关键词下的曝光位置 |
| 搜索潜力产品 | `mcp__sorftime__potential_product` | 发现高潜力产品 |

#### 典型调用示例

**按关键词搜索产品（最常用）：**
```
mcp__sorftime__product_search(
  searchName="wireless earbuds",
  amzSite="US",
  page=1
)
```

**可选筛选参数：**
- `price_min` / `price_max` — 价格范围
- `month_sales_volume_min` / `month_sales_volume_max` — 月销量范围
- `ratings_min` / `ratings_max` — 星级范围
- `ratings_count_min` / `ratings_count_max` — 评论数范围
- `brand` — 品牌筛选
- `delivery_type` — FBA / FBM 筛选

**Sorftime 实际返回的字段名（中文键名）及映射关系：**

| Sorftime 返回字段 | 多维表格字段 | 示例值 |
|-------------------|-------------|--------|
| `产品ASIN码` | ASIN | `B0D7FVQ1ZB` |
| `标题` | 产品名称 | `Apple AirPods 4 Wireless Earbuds` |
| `品牌` | 品牌 | `Apple` |
| `价格` | 价格(USD) | `114.95` |
| `月销量` | 月销量 | `21498` |
| `月销额` | 月销额(USD) | `2471195.10` |
| `星级` | 评分 | `4.5` |
| `评论数` | 评论数 | `26694` |
| `上架时间` | 上架时间 | `2024-09-10` |
| `发货方式` | 配送方式 | `FBA` / `AmzFBA` / `FBM` |
| `所属细分类目` | 类目排名 | `所属细分类目：Earbud Headphones（排名:3）` |
| `主图` | (可选) | 图片 URL |

> **关键注意事项：**
> - Sorftime 返回的是**中文键名**，不是驼峰英文
> - `发货方式` 的值可能是 `FBA`、`AmzFBA` 或 `FBM`，写入多维表格时需将 `AmzFBA` 统一映射为 `FBA`
> - `所属细分类目` 是一段文本（如 `所属细分类目：Earbud Headphones（排名:3）`），需要用正则提取排名数字
> - 每页返回约 50 条数据，如需更多可翻页 `page=2, 3...`

### 第二步：创建飞书多维表格

#### 2.1 创建 Base

```bash
lark-cli base +base-create --name "Amazon产品分析-{keyword}-{date}"
```

记录返回的 `base_token` 和 URL。

#### 2.2 创建数据表（含字段定义）

> **踩坑经验：** `+table-create` 的 `--fields` 中 **不支持 `property` 字段**（会报 `Unrecognized key 'property'` 错误），number 精度等属性需要后续通过 `+field-create` 单独设置。`single_select` 在 `+table-create` 里也不被支持，需改用 `+field-create` 创建。

**推荐做法：先用 `+table-create` 创建基础 text/number 字段，再用 `+field-create` 补充 select 等特殊类型字段。**

**Step 1 — 创建表（基础字段）：**
```bash
lark-cli base +table-create \
  --base-token {base_token} \
  --name "产品数据" \
  --fields '[
    {"name":"产品名称","type":"text"},
    {"name":"ASIN","type":"text"},
    {"name":"品牌","type":"text"},
    {"name":"价格(USD)","type":"number"},
    {"name":"月销量","type":"number"},
    {"name":"月销额(USD)","type":"number"},
    {"name":"评分","type":"number"},
    {"name":"评论数","type":"number"},
    {"name":"上架时间","type":"text"},
    {"name":"类目排名","type":"number"},
    {"name":"卖家","type":"text"}
  ]'
```

**Step 2 — 补充 select 字段：**
```bash
lark-cli base +field-create \
  --base-token {base_token} \
  --table-id {table_id} \
  --json '{"name":"配送方式","type":"select","multiple":false,"options":[{"name":"FBA"},{"name":"FBM"}]}'
```

> **字段说明**：`--fields` 数组的第一个元素会替换系统默认的首列，后续元素为新增字段。如果表名已存在会报错，此时直接用 `+field-list` 获取已有 table_id 即可。

#### 2.3 写入产品数据

对每条产品数据调用 `+record-upsert` 写入记录：

```bash
lark-cli base +record-upsert \
  --base-token {base_token} \
  --table-id {table_id} \
  --json '{
    "产品名称": "Sony WF-1000XM5",
    "ASIN": "B0C8Z7XXXX",
    "品牌": "Sony",
    "价格(USD)": 279.99,
    "月销量": 12000,
    "月销额(USD)": 3359880,
    "评分": 4.5,
    "评论数": 8234,
    "上架时间": "2023-07-01",
    "配送方式": "FBA",
    "类目排名": 3,
    "卖家": "Sony Official"
  }'
```

**批量写入规则：**
- 每批最多 500 条记录
- 批次间间隔 0.5-1 秒
- 不要写入 read-only 字段（formula / lookup / auto_number / created_at 等）

> **批量写入实战经验：**
> - 直接在 bash shell 中用 heredoc 写大量 JSON 会因单引号嵌套导致 `unexpected EOF` 错误
> - **推荐做法**：生成一个临时 Python 脚本，用 `subprocess.run()` 逐条调用 `lark-cli`
> - Windows 环境下 `subprocess` 找不到 `lark-cli`，需使用完整路径（如 `C:/Users/{user}/AppData/Roaming/npm/lark-cli.cmd`），可通过 `which lark-cli` 获取
> - Python 脚本示例结构：
> ```python
> import subprocess, json, time
> for product in products:
>     j = json.dumps(product, ensure_ascii=False)
>     cmd = ["lark-cli.cmd完整路径", "base", "+record-upsert",
>            "--base-token", BASE_TOKEN, "--table-id", TABLE_ID, "--json", j]
>     subprocess.run(cmd, capture_output=True, text=True, encoding="utf-8")
>     time.sleep(0.5)
> ```

### 第三步：创建仪表盘

#### 3.1 创建 Dashboard

```bash
lark-cli base +dashboard-create \
  --base-token {base_token} \
  --name "产品分析仪表盘" \
  --theme-style SimpleBlue
```

记录返回的 `dashboard_id`。

#### 3.2 添加统计卡片（KPI）

**产品总数：**
```bash
lark-cli base +dashboard-block-create \
  --base-token {base_token} \
  --dashboard-id {dashboard_id} \
  --name "产品总数" \
  --type statistics \
  --data-config '{"table_name":"产品数据","count_all":true}'
```

**平均价格：**
```bash
lark-cli base +dashboard-block-create \
  --base-token {base_token} \
  --dashboard-id {dashboard_id} \
  --name "平均价格" \
  --type statistics \
  --data-config '{"table_name":"产品数据","series":[{"field_name":"价格(USD)","rollup":"AVERAGE"}]}'
```

**总月销量：**
```bash
lark-cli base +dashboard-block-create \
  --base-token {base_token} \
  --dashboard-id {dashboard_id} \
  --name "总月销量" \
  --type statistics \
  --data-config '{"table_name":"产品数据","series":[{"field_name":"月销量","rollup":"SUM"}]}'
```

#### 3.3 添加图表

**品牌月销量柱状图：**
```bash
lark-cli base +dashboard-block-create \
  --base-token {base_token} \
  --dashboard-id {dashboard_id} \
  --name "各品牌月销量对比" \
  --type column \
  --data-config '{"table_name":"产品数据","series":[{"field_name":"月销量","rollup":"SUM"}],"group_by":[{"field_name":"品牌","mode":"integrated"}]}'
```

**配送方式分布饼图：**
```bash
lark-cli base +dashboard-block-create \
  --base-token {base_token} \
  --dashboard-id {dashboard_id} \
  --name "配送方式分布" \
  --type pie \
  --data-config '{"table_name":"产品数据","count_all":true,"group_by":[{"field_name":"配送方式","mode":"integrated"}]}'
```

**价格 vs 评论数散点图：**
```bash
lark-cli base +dashboard-block-create \
  --base-token {base_token} \
  --dashboard-id {dashboard_id} \
  --name "价格与评论数关系" \
  --type scatter \
  --data-config '{"table_name":"产品数据","series":[{"field_name":"评论数","rollup":"SUM"}],"group_by":[{"field_name":"价格(USD)","mode":"integrated"}]}'
```

### 第四步：返回结果

向用户输出：
1. 飞书多维表格的访问链接
2. 采集到的产品数量摘要
3. 仪表盘已包含的图表说明

示例输出：
> 已完成！共采集 50 条产品数据，已写入飞书多维表格。
> - 多维表格链接：{base_url}
> - 仪表盘包含：产品总数、平均价格、总月销量统计卡片 + 品牌销量柱状图 + 配送方式饼图 + 价格评论散点图

## 可选扩展能力

根据用户进一步需求，Agent 可追加以下操作：

| 需求 | 操作 |
|------|------|
| 查看某产品的销量趋势 | 调用 `product_trend` 后追加趋势数据表 |
| 分析竞品关键词 | 调用 `competitor_product_keywords` 后追加关键词表 |
| 对比不同时间段数据 | 调用 `product_search_from_history` 获取历史数据 |
| 查看类目市场趋势 | 调用 `category_trend` 后追加类目趋势表 |
| 查找1688采购货源 | 调用 `ali1688_similar_product` 追加采购成本表 |
| 添加更多仪表盘图表 | 使用 `+dashboard-block-create` 追加 line / bar / area 等图表 |

## 错误处理与踩坑记录

1. **Sorftime MCP 返回空数据**：提示用户换关键词或调整筛选条件
2. **lark-cli 权限不足**：提示用户检查飞书应用权限配置，参考 lark-shared skill
3. **记录写入失败**：检查字段类型是否匹配，确保不写入 read-only 字段
4. **仪表盘图表创建失败**：确认 `table_name` 与实际创建的表名一致，`field_name` 与实际字段名一致
5. **`+table-create` 报 `Unrecognized key 'property'`**：`--fields` 不支持 `property`，number 字段直接用 `{"type":"number"}` 不带 precision；select 字段改用 `+field-create` 单独创建
6. **`+table-create` 报 `validation_error` 表名重复**：说明表已创建（可能上一次部分成功），用 `+field-list` 获取已有 table_id 继续操作，再用 `+field-create` 补充缺失字段
7. **Bash heredoc 单引号嵌套导致 EOF 错误**：大量 JSON 数据不要用 shell heredoc 写入，改用 Python 脚本调用 `subprocess`
8. **Windows 下 `subprocess` 找不到 lark-cli**：需使用 `.cmd` 后缀的完整路径，如 `C:/Users/xxx/AppData/Roaming/npm/lark-cli.cmd`
9. **Sorftime `发货方式` 字段值不统一**：`FBA`、`AmzFBA` 都表示亚马逊配送，写入多维表格前统一映射为 `FBA`
