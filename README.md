# Model Compare Search

多模型对比搜索工具 - 自动从 NewAPI 获取可用模型，并行调用多模型获取综合答案。

## 功能特性

- **自动模型发现** - 从 NewAPI 动态获取可用模型列表
- **智能模型选择** - 自动过滤小模型，优先选择大模型、Pro 版本、推理模型
- **并行调用** - 最多 4 个并发请求，带失败重试和回退机制
- **结果保存** - 每个模型独立文件夹存储，方便后续分析
- **Kimi 总结** - 自动调用 Kimi 整合各模型观点

## 安装

### 方式 1：直接下载

```bash
# 克隆仓库
git clone https://github.com/throokie/model-compare-search.git
cd model-compare-search

# 或者只下载单个文件
curl -LO https://raw.githubusercontent.com/throokie/model-compare-search/main/search.mjs
chmod +x search.mjs
```

### 依赖

- Node.js 18+
- [Kimi CLI](https://kimi.com)（可选，用于结果总结）

## 配置

**⚠️ 必须配置环境变量才能使用**

### 方法 1：环境变量（推荐）

```bash
# 添加到 ~/.bashrc 或 ~/.zshrc
export NEWAPI_BASE_URL="http://your-newapi-server:4000/v1"
export NEWAPI_KEY="sk-your-api-key-here"

# 可选：自定义数据保存目录
export MCS_DATA_DIR="$HOME/.model-compare-search/data"
```

### 方法 2：运行时指定

```bash
NEWAPI_KEY="sk-your-api-key" node search.mjs "你的问题"
```

### 获取 NewAPI 账号

1. 部署 [NewAPI](https://github.com/Calcium-Ion/new-api) 或联系管理员
2. 在 NewAPI 控制台创建 API Key
3. 确保 NewAPI 已配置模型渠道（SiliconFlow、DeepSeek、Kimi 等）

## 使用方法

### 基本用法

```bash
# 使用默认模型自动搜索
./search.mjs "什么是优秀的API设计原则"

# 禁用 Kimi 总结（更快）
./search.mjs "你的问题" --no-kimi

# 完整路径调用
node /path/to/search.mjs "你的问题"
```

### 输出结构

结果自动保存到 `~/.model-compare-search/data/YYYYMMDD-HHMMSS/`：

```
~/.model-compare-search/data/
├── 2026-03-23-120000/
│   ├── meta.json          # 搜索元数据（成功率、token统计）
│   ├── summary.md         # Kimi综合总结
│   └── models/            # 各模型结果
│       ├── Pro_zai-org_GLM-5/
│       │   └── response.md
│       ├── Pro_moonshotai_Kimi-K2.5/
│       │   └── response.md
│       └── ...
```

### 查看历史结果

```bash
# 列出最近搜索
ls -lt ~/.model-compare-search/data/ | head -10

# 查看某次总结
cat ~/.model-compare-search/data/2026-03-23-120000/summary.md

# 查看某模型回答
cat ~/.model-compare-search/data/2026-03-23-120000/models/Pro_moonshotai_Kimi-K2.5/response.md
```

## 工作流程

```
用户输入
    │
    ▼
[动态获取模型列表] → 过滤小模型 → 排序选择前8个
    │
    ▼
[并行请求] → 4并发 → 失败自动回退
    │
    ▼
[保存结果] → 每个模型独立文件夹
    │
    ▼
[Kimi总结] → 综合各模型观点
    │
    ▼
[显示摘要] → 成功率 + 数据目录
```

## 模型选择策略

工具会自动从 NewAPI 获取模型并按以下规则筛选：

1. **去重** - 排除重复/相似模型名（保留完整版本）
2. **过滤** - 排除 <14B 小模型
3. **排序** - 按模型大小、Pro版本、推理能力评分
4. **限制** - 最多选择 8 个模型

支持的主要模型供应商：
- DeepSeek (R1, V3.2)
- 智谱AI (GLM-5, GLM-4.7)
- 月之暗面 (Kimi-K2.5)
- Qwen (2.5-72B, 3系列)
- MiniMax
- Doubao

## 配置选项

在 `search.mjs` 开头可修改默认配置：

```javascript
const CONFIG = {
  baseUrl: process.env.NEWAPI_BASE_URL || 'http://localhost:4000/v1',
  apiKey: process.env.NEWAPI_KEY || '',
  dataDir: process.env.MCS_DATA_DIR || join(homedir(), '.model-compare-search', 'data'),
  maxParallel: 4,      // 并发数
  maxRetries: 2,       // 失败重试次数
  retryDelay: 1000     // 重试间隔(ms)
};
```

## 故障排查

### 无法获取模型列表

```
⚠️ 无法获取模型列表，使用默认配置
```
- 检查 NEWAPI_BASE_URL 是否正确
- 检查 NEWAPI_KEY 是否有效
- 确认 NewAPI 服务可访问：`curl $NEWAPI_BASE_URL/models`

### 所有模型调用失败

- 检查 API Key 余额是否充足
- 检查 NewAPI 渠道配置是否正常
- 查看具体错误信息在 `models/<model>/error.md`

### Kimi 总结失败

- 确认已安装 Kimi CLI: `which kimi-cli`
- 或使用 `--no-kimi` 跳过总结

## 注意事项

1. **响应时间**：通常 30-120 秒，取决于最慢模型
2. **Token 消耗**：并行调用 8 个模型，总消耗较高
3. **数据持久化**：所有结果保存到本地，可长期保留
4. **隐私**：查询内容和结果仅保存在本地

## 示例输出

```
🚀 Model Compare Search v6.0
📝 什么是优秀的API设计原则

📡 获取可用模型...
📊 选择 5 个模型:
   • Pro/deepseek-ai/DeepSeek-R1
   • Pro/zai-org/GLM-5
   • Pro/moonshotai/Kimi-K2.5
   • ...

📡 正在请求 5 个模型...
█████ 5/5

🧠 Kimi 正在整合分析...

╔══════════════════════════════════════╗
║      多模型搜索完成                 ║
╚══════════════════════════════════════╝

📊 结果
   成功率: 5/5
   Tokens: 12456
   耗时: 45678ms

✅ 成功
   • Pro/deepseek-ai/DeepSeek-R1
   • Pro/zai-org/GLM-5
   • ...

📁 数据
   ~/.model-compare-search/data/2026-03-23-120000
```

## 许可证

MIT License

## 贡献

欢迎 Issue 和 PR！

## 相关项目

- [NewAPI](https://github.com/Calcium-Ion/new-api) - 统一的大模型 API 网关
