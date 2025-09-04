# 环境变量配置 (Environment Variables)

## 概述

Alpha Seeker 项目运行所需的环境变量配置说明。由于项目采用 Next.js 框架和 Airtable 数据服务，配置相对简单，主要关注外部API密钥和基础应用配置。

## 核心配置

### 必需环境变量
```bash
# Airtable API 配置 (核心必需)
NEXT_PUBLIC_AIRTABLE_BASE_ID=appxxxxxxxxxxxxxxxx    # Airtable Base ID，用于标识数据库
NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN=patxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  # Airtable 个人访问令牌

# AI 服务配置
ZHIPUAI_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  # 智谱 AI API 密钥，用于 AI 功能调用
```

### 可选配置
```bash
# Next.js 应用配置
NODE_ENV=development        # 运行环境: development/production
PORT=3000                   # 应用本地开发端口
NEXT_PUBLIC_APP_URL=http://localhost:3000  # 应用公开访问 URL
NEXT_PUBLIC_APP_NAME=Alpha Seeker  # 应用名称
NEXT_PUBLIC_APP_VERSION=0.1.0  # 应用版本号

# 功能开关
NEXT_PUBLIC_ENABLE_CONTENT_LIKING=true  # 是否启用内容点赞功能
NEXT_PUBLIC_ENABLE_CONTENT_FILTERING=true  # 是否启用内容筛选功能
NEXT_PUBLIC_CONTENT_PAGE_SIZE=12  # 每页显示的内容数量
```

## 配置文件示例

### 开发环境 (.env.development)
```bash
# 开发环境配置 - 本地开发使用
NODE_ENV=development
PORT=3000
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_APP_NAME=Alpha Seeker (Dev)
NEXT_PUBLIC_APP_VERSION=0.1.0-dev

# Airtable 开发配置
NEXT_PUBLIC_AIRTABLE_BASE_ID=appxxxxxxxxxxxxxxxx
NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN=patxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# AI 服务开发配置
ZHIPUAI_API_KEY=dev_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# 开发功能开关
NEXT_PUBLIC_ENABLE_CONTENT_LIKING=true
NEXT_PUBLIC_ENABLE_CONTENT_FILTERING=true
```

### 生产环境 (.env.production)
```bash
# 生产环境配置 - 部署时使用
NODE_ENV=production
PORT=3000
NEXT_PUBLIC_APP_URL=https://alpha-seeker.vercel.app  # 替换为实际域名
NEXT_PUBLIC_APP_NAME=Alpha Seeker
NEXT_PUBLIC_APP_VERSION=0.1.0

# Airtable 生产配置（从部署平台环境变量获取）
NEXT_PUBLIC_AIRTABLE_BASE_ID=${AIRTABLE_BASE_ID}
NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN=${AIRTABLE_PERSONAL_ACCESS_TOKEN}

# AI 服务生产配置
ZHIPUAI_API_KEY=${ZHIPUAI_API_KEY}

# 生产功能开关
NEXT_PUBLIC_ENABLE_CONTENT_LIKING=true
NEXT_PUBLIC_ENABLE_CONTENT_FILTERING=true
```

## 配置验证

### 基础验证函数
```typescript
// lib/config-validation.ts
export function validateConfig(): { isValid: boolean; errors: string[] } {
  const errors: string[] = [];
  
  // 验证 Airtable 配置 - 核心内容存储
  const airtableBaseId = process.env.NEXT_PUBLIC_AIRTABLE_BASE_ID;
  const airtableToken = process.env.NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN;
  
  if (!airtableBaseId) {
    errors.push('Airtable Base ID 未配置 - 无法访问内容库');
  }
  
  if (!airtableToken) {
    errors.push('Airtable Personal Access Token 未配置 - 无法读取内容');
  }
  
  // 验证 AI 服务配置 - 策展辅助
  if (!process.env.ZHIPUAI_API_KEY) {
    errors.push('智谱 AI API Key 未配置 - 无法使用AI策展辅助');
  }
  
  // 验证应用基础配置
  if (!process.env.NEXT_PUBLIC_APP_URL) {
    errors.push('应用 URL 未配置');
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
}

// 在应用启动时验证配置
export function ensureValidConfig(): void {
  const validation = validateConfig();
  
  if (!validation.isValid) {
    console.error('配置验证失败：');
    validation.errors.forEach(error => {
      console.error(`  - ${error}`);
    });
    
    if (process.env.NODE_ENV === 'production') {
      throw new Error('应用配置无效，请检查环境变量设置');
    }
  }
}
```

## 安全最佳实践

### 环境变量安全
```bash
# ✅ 正确做法：使用客户端安全的环境变量（NEXT_PUBLIC_ 前缀）
NEXT_PUBLIC_AIRTABLE_BASE_ID=appxxxxxxxxxxxxxxxx

# ✅ 正确做法：敏感信息使用服务端环境变量（无 NEXT_PUBLIC_ 前缀）
AIRTABLE_PERSONAL_ACCESS_TOKEN=patxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ❌ 错误做法：不要将敏感信息暴露给客户端
NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN=patxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 配置文件管理
```bash
# .gitignore 文件中应该包含：
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# 确保敏感配置不会被提交到代码仓库
```

## 开发和部署指南

### 本地开发设置
```bash
# 1. 复制环境变量模板
cp .env.example .env.development

# 2. 编辑 .env.development 文件，填入实际配置
# 3. 安装依赖
npm install

# 4. 启动开发服务器
npm run dev
```

### 生产环境部署
```bash
# 1. 在部署平台设置环境变量
# Vercel、Netlify 或其他平台的环境变量设置界面

# 2. 确保所有必需的配置都已设置
# - Airtable 配置
# - AI 服务配置
# - 应用基础配置

# 3. 构建和部署
npm run build
npm start
```

## 故障排除

### 常见配置问题

1. **Airtable 连接失败**
   ```
   错误：Airtable configuration missing
   解决：检查 NEXT_PUBLIC_AIRTABLE_BASE_ID 和 NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN 是否正确设置
   ```

2. **AI 服务不可用**
   ```
   错误：ZhipuAI API key missing
   解决：确保 ZHIPUAI_API_KEY 环境变量已正确设置
   ```

3. **客户端配置问题**
   ```
   错误：Cannot access NEXT_PUBLIC_ variable on client
   解决：确保客户端需要的环境变量使用了 NEXT_PUBLIC_ 前缀
   ```

## 修改指南

### ✅ 安全的修改
- 添加新的内容功能开关（如新的内容形态）
- 调整日志级别和内容质量监控配置
- 修改客户端可见的非敏感配置（如显示选项）

### ⚠️ 需要谨慎的修改
- 修改 Airtable 连接配置（可能影响内容库访问）
- 更改 AI 服务配置（可能影响策展质量）
- 调整应用基础 URL（可能影响内容分享和SEO）
- 更改内容表结构（需要数据迁移）

### 🚨 危险操作
- 删除必需的环境变量（会导致应用无法运行）
- 在客户端暴露敏感信息（安全风险）
- 在生产环境使用开发配置（性能和安全问题）
- 修改策展工作流配置（可能影响内容质量）
- 关闭来源归因功能（可能侵犯知识产权）