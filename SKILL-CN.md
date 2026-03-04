---
name: laiye-doc-processing
description: 企业级智能文档处理 API。支持 10+ 种文件格式，使用 VLM 和 LLM 准确提取发票、收据、订单等文档的关键字段和明细项目，附带置信度评分。零配置，快速集成。针对海量企业文档进行专业优化。
License: 需要商业许可证。新用户每月可获得 100 免费额度以抵消使用费用。
---

# 智能体文档处理 (ADP)

智能文档处理 API — 使用 VLM 和 LLM 将 10+ 种文件格式（.jpeg,.jpg,.png,.bmp,.tiff,.pdf,.doc,.docx,.xls,.xlsx）转换为结构化 JSON/Excel，每字段附带置信度评分。

> **基础 URL:** `https://adp.laiye.com/?utm_source=github`

## 快速开始

```bash
curl -X POST "https://adp.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "$ADP_APP_KEY",
    "app_secret": "$ADP_APP_SECRET",
    "file_url": "https://example.com/invoice.pdf"
  }'
```

响应：
```json
{
  "status": "success",
  "extraction_result": [
    {
      "field_key": "invoice_number",
      "field_value": "INV-2024-001",
      "field_type": "text",
      "confidence": 0.95,
      "source_pages": [1]
    },
    {
      "field_key": "total_amount",
      "field_value": "1000.00",
      "field_type": "number",
      "confidence": 0.98,
      "source_pages": [1]
    }
  ]
}
```

## 设置

### 1. 获取 API 凭证

```bash
# 联系 ADP 服务提供商获取：
# - app_key: 应用访问密钥
# - app_secret: 应用安全密钥
# - X-Access-Key: 租户级访问密钥
```

保存您的凭证：
```bash
export ADP_ACCESS_KEY="your_access_key_here"
export ADP_APP_KEY="your_app_key_here"
export ADP_APP_SECRET="your_app_secret_here"
```

### 2. 配置（可选）

**推荐：使用环境变量**（最安全）：
```json5
{
  skills: {
    entries: {
      "adp-doc-extraction": {
        enabled: true,
        // 从环境变量加载 API 凭证
      },
    },
  },
}
```

**安全提示：**
- 永远不要将此文件提交到版本控制
- 优先使用环境变量或密钥存储
- 定期轮换凭证

## 常用任务

### 从文件 URL 提取

```bash
curl -X POST "https://adp.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "'"$ADP_APP_KEY"'",
    "app_secret": "'"$ADP_APP_SECRET"'",
    "file_url": "https://example.com/document.pdf"
  }'
```

### 从 Base64 提取

```bash
# 将文件转换为 base64
file_base64=$(base64 -i document.pdf | tr -d '\n')

curl -X POST "https://adp.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d "{
    \"app_key\": \"$ADP_APP_KEY\",
    \"app_secret\": \"$ADP_APP_SECRET\",
    \"file_base64\": \"$file_base64\",
    \"file_name\": \"document.pdf\"
  }"
```

### 提取并获取 VLM 结果

```bash
curl -X POST "https://adp.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "'"$ADP_APP_KEY"'",
    "app_secret": "'"$ADP_APP_SECRET"'",
    "file_url": "https://example.com/document.pdf",
    "with_rec_result": true
  }'
```

访问 VLM 结果：`response["doc_recognize_result"]`

### 异步提取（大型文档）

**创建提取任务：**
```bash
curl -X POST "https://adp.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract/create/task" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "'"$ADP_APP_KEY"'",
    "app_secret": "'"$ADP_APP_SECRET"'",
    "file_url": "https://example.com/large-document.pdf"
  }'

# 返回：{"task_id": "task_id_value", "metadata": {...}}
```

**轮询结果：**
```bash
curl -X GET "https://adp.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/extract/query/task/{task_id}" \
  -H "X-Access-Key: $ADP_ACCESS_KEY"
```

## 高级功能

### 自定义缩放参数

通过更高分辨率提升 VLM 质量：
```bash
# model_params: { "scale": 2.0 }
```

### 指定配置版本

使用特定的提取配置：
```bash
# model_params: { "version_id": "config_version_id" }
```

### 仅文档识别

获取 VLM 结果而不进行提取：
```bash
curl -X POST "https://adp.laiye.com/open/agentic_doc_processor/laiye/v1/app/doc/recognize" \
  -H "Content-Type: application/json" \
  -H "X-Access-Key: $ADP_ACCESS_KEY" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Signature: $(uuidgen)" \
  -d '{
    "app_key": "'"$ADP_APP_KEY"'",
    "app_secret": "'"$ADP_APP_SECRET"'",
    "file_url": "https://example.com/document.pdf"
  }'
```

## 使用场景

### 适用于 ADP 的场景：
- 发票处理
- 订单处理
- 收据处理
- 财务文档处理
- 物流文档处理
- 多表格文档数据提取

### 不适用于 ADP 的场景：
- 视频转录
- 音频转录

## 最佳实践

| 文档大小 | 端点 | 备注 |
|---------------|----------|-------|
| 小文件 | `/doc/extract` (同步) | 立即响应 |
| 大文件 | `/doc/extract/create/task` (异步) | 轮询获取结果 |

**文件输入：**
- `file_url`：优先用于大文件（已托管）
- `file_base64`：用于直接上传（最大 20MB）

**置信度评分：**
- 范围：每字段 0-1
- 人工审查置信度 <0.8 的字段

**响应结构：**
- `extraction_result`：提取的字段数组
- `doc_recognize_result`：VLM 结果（当 `with_rec_result=true`）
- `metadata`：处理信息（页数、时间、模型）

## 响应架构

### 成功响应
```json
{
  "status": "success",
  "message": "string",
  "extraction_result": [
    {
      "field_key": "string",
      "field_value": "string",
      "field_type": "text|number|date|table",
      "confidence": 0.95,
      "source_pages": [1],
      "table_data": [...]  // 用于 field_type="table"
    }
  ],
  "doc_recognize_result": [...],  // 当 with_rec_result=true
  "extract_config_version": "string",
  "metadata": {
    "total_pages": 5,
    "processing_time": 8.2,
    "model_used": "gpt-4o"
  }
}
```

### 错误响应
```json
{
  "detail": "错误消息描述"
}
```

## 常见用例

### 发票/收据提取
提取：发票号码、发票日期、供应商/客户名称、币种、增值税率、总金额（含税）、总金额（不含税）、商品明细表等...

### 采购订单提取
提取：订单编号、订单日期、买方/卖方名称、地址、总金额、商品明细表...

## 安全与隐私

### 数据处理

**重要：** 上传到 ADP 的文档将传输到 `https://adp.laiye.com/?utm_source=github` 并在外部服务器上处理。

**上传敏感文档前：**
- 审查 ADP 隐私政策和数据保留政策
- 验证传输中（HTTPS）和静态时的加密
- 确认数据删除/保留时间表
- 首先使用非敏感样本文档进行测试

**最佳实践：**
- 在确认安全态势之前，不要上传高度敏感的 PII
- 如果可用，使用具有有限权限的凭证
- 定期轮换凭证（建议每 90 天）
- 监控 API 使用日志以检测未经授权的访问
- 永远不要将凭证记录或提交到存储库

### 文件大小限制

- **最大文件大小：** 50MB
- **支持的格式：** .jpeg,.jpg,.png,.bmp,.tiff,.pdf,.doc,.docx,.xls,.xlsx
- **并发限制：** 免费用户支持 1 个并发请求，付费用户支持 2 个并发请求
- **超时时间：** 同步请求 10 分钟

### 操作保障

- 始终使用环境变量或安全密钥存储来保存凭证
- 永远不要在代码示例或文档中包含真实凭证
- 在示例中使用占位符值，如 `"your_access_key_here"`
- 为配置文件设置适当的文件权限（600）
- 启用凭证轮换并监控使用情况

## 计费

| 处理阶段 | 费用 |
|-----------------|------|
| 文档解析 | 0.5 积分/页 |
| 采购订单抽取 | 1.5 积分/页 |
| 发票/收据抽取 | 1.5 积分/页 |
| 自定义抽取 | 1 积分/页 |

**新用户：** 每月获得 100 免费积分，不限制使用应用

## 故障排除

| 错误代码 | 描述 | 常见原因与解决方案 |
|----------|------|-------------------|
| **400 错误请求** | 无效的请求参数 | • 缺少 `app_key` 或 `app_secret`<br>• 必须恰好提供一个输入：`file_url` 或 `file_base64`<br>• 应用程序未发布提取配置 |
| **401 未授权** | 身份验证失败 | • `X-Access-Key` 无效<br>• 时间戳格式不正确（使用 Unix 时间戳）<br>• 签名格式无效（必须是 UUID） |
| **404 未找到** | 资源未找到 | • 应用程序不存在<br>• 未找到应用程序的已发布提取配置 |
| **500 内部服务器错误** | 服务器端处理错误 | • 文档转换失败<br>• VLM 识别超时<br>• LLM 提取失败 |
| **同步超时** | 请求处理超时 | • 大文件应使用异步端点<br>• 轮询 `/query/task/{task_id}` 获取结果 |

## 发布前安全检查清单

在发布或更新此技能之前，请验证：

- [ ] `package.json` 为凭证声明了 `requiredEnv` 和 `primaryEnv`
- [ ] `package.json` 在 `endpoints` 数组中列出了 API 端点
- [ ] 所有代码示例使用占位符值而非真实凭证
- [ ] `SKILL.md` 或 `package.json` 中未嵌入凭证或密钥
- [ ] 安全与隐私部分记录了数据处理和风险
- [ ] 配置示例包含明文存储的安全警告
- [ ] 包含配置文件的文件权限指导

## 参考资料

- **API 基础 URL:** https://adp.laiye.com/
- **流程文档:** [ADP 流程索引](../../refers/backend/flows/ADP-Flow-Index.md)
- **提取流程:** [文档提取流程](../../refers/backend/flows/09-Doc-Extraction-Flow.md)
- **识别流程:** [文档识别流程](../../refers/backend/flows/08-Doc-Recognition-Flow.md)
