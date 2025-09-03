# 环境变量配置 (Environment Variables)

## 概述

Alpha Seeker 项目运行所需的环境变量配置说明。由于项目采用 Next.js 框架和 Airtable 数据服务，配置相对简单，主要关注外部API密钥和基础应用配置。

## 配置分类

### Airtable 数据库配置
```bash
# Airtable API 配置 (核心必需)
NEXT_PUBLIC_AIRTABLE_BASE_ID=appxxxxxxxxxxxxxxxx    # Airtable Base ID，用于标识数据库
NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN=patxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  # Airtable 个人访问令牌

# 或者使用服务端变量（推荐用于生产环境）
AIRTABLE_BASE_ID=appxxxxxxxxxxxxxxxx             # 服务端 Airtable Base ID
AIRTABLE_PERSONAL_ACCESS_TOKEN=patxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  # 服务端访问令牌
```

### AI 服务配置
```bash
# 智谱 AI 服务配置
ZHIPUAI_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  # 智谱 AI API 密钥，用于 AI 功能调用

# 可选的其他 AI 服务
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  # OpenAI API 密钥（如果需要）
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  # Anthropic API 密钥（如果需要）
```

### Next.js 应用配置
```bash
# Next.js 基础配置
NODE_ENV=development        # 运行环境: development/production
PORT=3000                   # 应用本地开发端口
NEXT_PUBLIC_APP_URL=http://localhost:3000  # 应用公开访问 URL
NEXT_PUBLIC_APP_NAME=Alpha Seeker  # 应用名称
NEXT_PUBLIC_APP_VERSION=0.1.0  # 应用版本号

# 内容管理配置
NEXT_PUBLIC_ENABLE_CONTENT_LIKING=true  # 是否启用内容点赞功能
NEXT_PUBLIC_ENABLE_CONTENT_FILTERING=true  # 是否启用内容筛选功能
NEXT_PUBLIC_CONTENT_PAGE_SIZE=12  # 每页显示的内容数量
```

### 分析和监控配置
```bash
# 可选的分析服务
NEXT_PUBLIC_GOOGLE_ANALYTICS_ID=G-XXXXXXXXXX  # Google Analytics ID（如果需要）
NEXT_PUBLIC_VERCEL_ANALYTICS_ID=xxxxxxxxxxxxxxxxxxxxxxxx  # Vercel Analytics ID（如果部署在 Vercel）

# 日志配置
NEXT_PUBLIC_LOG_LEVEL=info    # 日志级别: error/warn/info/debug
LOG_TO_CONSOLE=true          # 是否在控制台输出日志
```

### 开发工具配置
```bash
# 开发环境特定配置
NEXT_PUBLIC_DEV_MODE=true    # 开发模式开关
NEXT_PUBLIC_SHOW_DEV_TOOLS=false  # 是否显示开发工具
NEXT_PUBLIC_ENABLE_MOCK_DATA=false  # 是否启用模拟数据
```

## 环境配置文件示例

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
NEXT_PUBLIC_DEV_MODE=true
NEXT_PUBLIC_SHOW_DEV_TOOLS=true
NEXT_PUBLIC_LOG_LEVEL=debug
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
AIRTABLE_BASE_ID=${AIRTABLE_BASE_ID}
AIRTABLE_PERSONAL_ACCESS_TOKEN=${AIRTABLE_PERSONAL_ACCESS_TOKEN}

# AI 服务生产配置
ZHIPUAI_API_KEY=${ZHIPUAI_API_KEY}

# 生产功能开关
NEXT_PUBLIC_ENABLE_CONTENT_LIKING=true
NEXT_PUBLIC_ENABLE_CONTENT_FILTERING=true
NEXT_PUBLIC_DEV_MODE=false
NEXT_PUBLIC_SHOW_DEV_TOOLS=false
NEXT_PUBLIC_LOG_LEVEL=info

# 可选的生产服务
NEXT_PUBLIC_GOOGLE_ANALYTICS_ID=${GOOGLE_ANALYTICS_ID}
```

## 配置加载和验证

### Next.js 配置加载
```typescript
// config/airtable.ts
export interface AirtableConfig {
  baseId: string;
  personalAccessToken: string;
}

// 从环境变量加载 Airtable 配置
export const getAirtableConfig = (): AirtableConfig => {
  const baseId = process.env.NEXT_PUBLIC_AIRTABLE_BASE_ID || 
                process.env.AIRTABLE_BASE_ID;
  
  const personalAccessToken = process.env.NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN || 
                            process.env.AIRTABLE_PERSONAL_ACCESS_TOKEN;
  
  // 验证必需的配置
  if (!baseId || !personalAccessToken) {
    throw new Error('Airtable 配置缺失：请设置 AIRTABLE_BASE_ID 和 AIRTABLE_PERSONAL_ACCESS_TOKEN');
  }
  
  return {
    baseId,
    personalAccessToken
  };
};

// 应用配置
export interface AppConfig {
  appName: string;
  appVersion: string;
  appUrl: string;
  enableContentLiking: boolean;
  enableContentFiltering: boolean;
  contentPageSize: number;
  logLevel: string;
  isDevMode: boolean;
}

export const getAppConfig = (): AppConfig => {
  const appName = process.env.NEXT_PUBLIC_APP_NAME || 'Alpha Seeker';
  const appVersion = process.env.NEXT_PUBLIC_APP_VERSION || '0.1.0';
  const appUrl = process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000';
  
  return {
    appName,
    appVersion,
    appUrl,
    enableContentLiking: process.env.NEXT_PUBLIC_ENABLE_CONTENT_LIKING === 'true',
    enableContentFiltering: process.env.NEXT_PUBLIC_ENABLE_CONTENT_FILTERING === 'true',
    contentPageSize: parseInt(process.env.NEXT_PUBLIC_CONTENT_PAGE_SIZE || '12'),
    logLevel: process.env.NEXT_PUBLIC_LOG_LEVEL || 'info',
    isDevMode: process.env.NODE_ENV === 'development'
  };
};
```

### 配置验证中间件
```typescript
// lib/config-validation.ts
export function validateConfig(): { isValid: boolean; errors: string[] } {
  const errors: string[] = [];
  
  // 验证 Airtable 配置 - 核心内容存储
  const airtableBaseId = process.env.NEXT_PUBLIC_AIRTABLE_BASE_ID || 
                         process.env.AIRTABLE_BASE_ID;
  const airtableToken = process.env.NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN || 
                        process.env.AIRTABLE_PERSONAL_ACCESS_TOKEN;
  
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
  
  // 验证首席策展人模式配置
  if (process.env.NODE_ENV === 'production') {
    if (!process.env.NEXT_PUBLIC_ENABLE_CURATOR_NOTES) {
      errors.push('策展人笔记功能未启用 - 生产环境必需');
    }
    if (!process.env.NEXT_PUBLIC_SHOW_SOURCE_ATTRIBUTION) {
      errors.push('来源归因功能未启用 - 生产环境必需');
    }
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

### 1. 环境变量安全
```bash
# ✅ 正确做法：使用客户端安全的环境变量（NEXT_PUBLIC_ 前缀）
NEXT_PUBLIC_AIRTABLE_BASE_ID=appxxxxxxxxxxxxxxxx

# ✅ 正确做法：敏感信息使用服务端环境变量（无 NEXT_PUBLIC_ 前缀）
AIRTABLE_PERSONAL_ACCESS_TOKEN=patxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ❌ 错误做法：不要将敏感信息暴露给客户端
NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN=patxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 2. 配置文件管理
```typescript
# .gitignore 文件中应该包含：
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# 确保敏感配置不会被提交到代码仓库
```

### 3. 部署平台配置
```typescript
// Vercel 部署配置 (vercel.json)
{
  "env": {
    "AIRTABLE_BASE_ID": "@airtable_base_id",
    "AIRTABLE_PERSONAL_ACCESS_TOKEN": "@airtable_personal_access_token",
    "ZHIPUAI_API_KEY": "@zhipuai_api_key"
  },
  "build": {
    "env": {
      "NEXT_PUBLIC_APP_URL": "https://your-domain.vercel.app"
    }
  }
}
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

### 配置测试
```typescript
// 测试配置加载
describe('Configuration', () => {
  test('should load Airtable config', () => {
    const config = getAirtableConfig();
    expect(config.baseId).toBeDefined();
    expect(config.personalAccessToken).toBeDefined();
  });
  
  test('should validate config', () => {
    const validation = validateConfig();
    // 在开发环境允许缺失配置用于测试
    if (process.env.NODE_ENV === 'production') {
      expect(validation.isValid).toBe(true);
    }
  });
});
```

## 故障排除

### 常见配置问题

1. **Airtable 连接失败**
   ```
   错误：Airtable configuration missing
   解决：检查 AIRTABLE_BASE_ID 和 AIRTABLE_PERSONAL_ACCESS_TOKEN 是否正确设置
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

### 配置调试工具
```typescript
// utils/debug-config.ts
export function debugConfig(): void {
  if (process.env.NODE_ENV !== 'development') {
    console.warn('配置调试仅在开发环境可用');
    return;
  }
  
  console.log('=== 应用配置 ===');
  console.log('应用名称:', process.env.NEXT_PUBLIC_APP_NAME);
  console.log('应用版本:', process.env.NEXT_PUBLIC_APP_VERSION);
  console.log('应用URL:', process.env.NEXT_PUBLIC_APP_URL);
  console.log('运行环境:', process.env.NODE_ENV);
  
  console.log('\n=== Airtable 配置 (内容存储) ===');
  console.log('Base ID:', process.env.NEXT_PUBLIC_AIRTABLE_BASE_ID ? '已设置' : '未设置');
  console.log('Access Token:', process.env.NEXT_PUBLIC_AIRTABLE_PERSONAL_ACCESS_TOKEN ? '已设置' : '未设置');
  console.log('内容表:', process.env.NEXT_PUBLIC_AIRTABLE_CONTENTS_TABLE || 'Contents');
  console.log('专题表:', process.env.NEXT_PUBLIC_AIRTABLE_TOPICS_TABLE || 'Topics');
  
  console.log('\n=== AI 服务配置 (策展辅助) ===');
  console.log('智谱AI:', process.env.ZHIPUAI_API_KEY ? '已设置' : '未设置');
  
  console.log('\n=== 首席策展人模式功能 ===');
  console.log('内容点赞:', process.env.NEXT_PUBLIC_ENABLE_CONTENT_LIKING);
  console.log('内容筛选:', process.env.NEXT_PUBLIC_ENABLE_CONTENT_FILTERING);
  console.log('策展人笔记:', process.env.NEXT_PUBLIC_ENABLE_CURATOR_NOTES);
  console.log('来源归因:', process.env.NEXT_PUBLIC_SHOW_SOURCE_ATTRIBUTION);
  console.log('反馈收集:', process.env.NEXT_PUBLIC_ENABLE_FEEDBACK_COLLECTION);
  console.log('策展工作流:', process.env.NEXT_PUBLIC_ENABLE_CURATION_WORKFLOW);
  console.log('开发模式:', process.env.NEXT_PUBLIC_DEV_MODE);
}
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

## 环境配置管理

### 开发环境 (.env.development)
```bash
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379
JWT_SECRET=dev-jwt-secret
LOG_LEVEL=debug
ENABLE_CACHE=false
```

### 测试环境 (.env.test)
```bash
NODE_ENV=test
PORT=3001
DATABASE_URL=postgresql://localhost:5432/myapp_test
REDIS_URL=redis://localhost:6379/1
JWT_SECRET=test-jwt-secret
LOG_LEVEL=error
ENABLE_EMAIL_VERIFICATION=false
```

### 生产环境 (.env.production)
```bash
NODE_ENV=production
PORT=3000
DATABASE_URL=${DATABASE_URL}  # 从部署平台获取
REDIS_URL=${REDIS_URL}
JWT_SECRET=${JWT_SECRET}
LOG_LEVEL=info
ENABLE_CACHE=true
```

## 配置验证

### 环境变量验证
```javascript
// config/validation.js
const Joi = require('joi')

const envSchema = Joi.object({
  NODE_ENV: Joi.string()
    .valid('development', 'test', 'staging', 'production')
    .default('development'),
  
  PORT: Joi.number()
    .port()
    .default(3000),
  
  DATABASE_URL: Joi.string()
    .uri()
    .required(),
  
  JWT_SECRET: Joi.string()
    .min(32)
    .required(),
  
  JWT_EXPIRES_IN: Joi.string()
    .default('24h'),
  
  BCRYPT_ROUNDS: Joi.number()
    .integer()
    .min(10)
    .max(15)
    .default(12),
  
  LOG_LEVEL: Joi.string()
    .valid('error', 'warn', 'info', 'debug')
    .default('info'),
  
  ENABLE_REGISTRATION: Joi.boolean()
    .default(true),
  
  MAX_UPLOAD_SIZE: Joi.number()
    .integer()
    .positive()
    .default(10485760) // 10MB
})

function validateEnv() {
  const { error, value } = envSchema.validate(process.env, {
    allowUnknown: true,
    stripUnknown: true
  })
  
  if (error) {
    throw new Error(`环境变量验证失败: ${error.message}`)
  }
  
  return value
}

module.exports = { validateEnv }
```

### 配置加载
```javascript
// config/index.js
const { validateEnv } = require('./validation')

// 加载环境变量
require('dotenv').config({
  path: `.env.${process.env.NODE_ENV || 'development'}`
})

// 验证并导出配置
const config = validateEnv()

module.exports = {
  app: {
    name: config.APP_NAME,
    version: config.APP_VERSION,
    port: config.PORT,
    env: config.NODE_ENV,
    baseUrl: config.BASE_URL
  },
  
  database: {
    url: config.DATABASE_URL,
    host: config.DB_HOST,
    port: config.DB_PORT,
    name: config.DB_NAME,
    user: config.DB_USER,
    password: config.DB_PASSWORD,
    ssl: config.DB_SSL
  },
  
  redis: {
    url: config.REDIS_URL,
    host: config.REDIS_HOST,
    port: config.REDIS_PORT,
    password: config.REDIS_PASSWORD,
    db: config.REDIS_DB
  },
  
  auth: {
    jwtSecret: config.JWT_SECRET,
    jwtExpiresIn: config.JWT_EXPIRES_IN,
    bcryptRounds: config.BCRYPT_ROUNDS
  },
  
  features: {
    registration: config.ENABLE_REGISTRATION,
    emailVerification: config.ENABLE_EMAIL_VERIFICATION,
    payment: config.ENABLE_PAYMENT,
    cache: config.ENABLE_CACHE,
    rateLimiting: config.ENABLE_RATE_LIMITING
  },
  
  upload: {
    maxSize: config.MAX_UPLOAD_SIZE
  },
  
  logging: {
    level: config.LOG_LEVEL,
    format: config.LOG_FORMAT,
    file: config.LOG_FILE
  }
}
```

## 安全最佳实践

### 1. 敏感信息保护
```bash
# ❌ 错误：在代码中硬编码密钥
const JWT_SECRET = 'my-secret-key'

# ✅ 正确：使用环境变量
const JWT_SECRET = process.env.JWT_SECRET
```

### 2. 密钥轮换
```javascript
// 支持多个 JWT 密钥，便于密钥轮换
const JWT_SECRETS = [
  process.env.JWT_SECRET_CURRENT,  // 当前密钥
  process.env.JWT_SECRET_PREVIOUS  // 上一个密钥（用于验证旧 token）
]

function verifyToken(token) {
  for (const secret of JWT_SECRETS) {
    try {
      return jwt.verify(token, secret)
    } catch (error) {
      continue
    }
  }
  throw new Error('Invalid token')
}
```

### 3. 环境隔离
```javascript
// 确保生产环境不会意外使用开发配置
if (process.env.NODE_ENV === 'production') {
  const requiredVars = [
    'DATABASE_URL',
    'JWT_SECRET',
    'REDIS_URL'
  ]
  
  for (const varName of requiredVars) {
    if (!process.env[varName]) {
      throw new Error(`生产环境缺少必需的环境变量: ${varName}`)
    }
  }
}
```

## 部署配置

### Docker 环境变量
```dockerfile
# Dockerfile
FROM node:18-alpine

# 设置默认环境变量
ENV NODE_ENV=production
ENV PORT=3000

# 复制应用代码
COPY . /app
WORKDIR /app

# 安装依赖
RUN npm ci --only=production

# 暴露端口
EXPOSE $PORT

# 启动应用
CMD ["npm", "start"]
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    env_file:
      - .env.production
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Kubernetes ConfigMap
```yaml
# k8s-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  NODE_ENV: "production"
  PORT: "3000"
  LOG_LEVEL: "info"
  ENABLE_CACHE: "true"

---
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
type: Opaque
data:
  JWT_SECRET: <base64-encoded-secret>
  DATABASE_URL: <base64-encoded-url>
```

## 监控和调试

### 配置检查端点
```javascript
// routes/health.js
app.get('/health/config', (req, res) => {
  // 只在非生产环境暴露配置信息
  if (process.env.NODE_ENV === 'production') {
    return res.status(403).json({ error: 'Forbidden' })
  }
  
  const safeConfig = {
    app: {
      name: config.app.name,
      version: config.app.version,
      env: config.app.env
    },
    features: config.features,
    database: {
      host: config.database.host,
      port: config.database.port,
      name: config.database.name
      // 不暴露密码等敏感信息
    }
  }
  
  res.json(safeConfig)
})
```

## 修改指南

### ✅ 安全的修改
- 添加新的功能开关
- 调整非敏感配置的默认值
- 增加配置验证规则

### ⚠️ 需要谨慎的修改
- 修改数据库连接配置
- 更改 JWT 密钥
- 调整缓存配置

### 🚨 危险操作
- 删除必需的环境变量
- 在代码中硬编码密钥
- 在生产环境暴露敏感配置
