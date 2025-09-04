# API 数据契约 (API Contracts)

## 概述
定义Alpha Seeker情报聚合平台的前后端数据交换标准格式，包括请求参数、响应结构和错误处理机制。

## 设计原则
- **一致性**：所有 API 遵循统一的数据格式和命名规范
- **可预测性**：明确的输入输出规范，减少集成错误
- **可维护性**：清晰的文档便于团队协作和项目维护
- **安全性**：适当的认证和授权机制

## 通用响应格式

所有 API 响应都遵循统一格式，确保前端应用能够一致地处理所有API调用结果：

```json
{
  "success": true,          // 请求处理状态
  "data": {},              // 业务数据
  "message": "操作成功",    // 操作结果描述
  "code": 200,             // HTTP状态码
  "timestamp": 1640995200  // 响应时间戳
}
```

## 情报相关 API

### 获取情报列表
**用途**：从Airtable数据库中获取所有情报内容，支持根据用户级别进行智能过滤，确保为不同经验水平的用户提供最相关的内容。

```http
GET /api/intelligence?userLevel=both

查询参数：
- userLevel: 用户级别过滤（newcomer/veteran/both，默认both）

成功响应：
{
  "success": true,
  "data": [
    {
      "id": "rec123456",
      "title": "AI编程的最佳实践",
      "insight": "通过合理的提示工程和代码生成策略...",
      "author": "张三",
      "author_link": "https://example.com/author",
      "source_context": "技术博客",
      "source_link": "https://example.com/article",
      "domain": "AI-Powered Professional Skills",
      "second_domain": "AI Programming",
      "tier": "Tier 1: must read",
      "tags": ["Code Generation", "AI-assisted Debugging"],
      "vote_count": 15,
      "quality_score": 8.5,
      "published_at": "2024-01-01T00:00:00Z",
      "user_level": "both",
      "positioning_context": "在AI编程领域的重要实践"
    }
  ],
  "message": "获取成功",
  "code": 200
}
```

### 提交新情报
**用途**：用户提交新的情报内容

```http
POST /api/intelligence
Content-Type: application/json

请求体：
{
  "title": "情报标题",
  "insight": "核心洞察内容",
  "author": "作者名称",
  "source_context": "来源上下文",
  "source_link": "https://example.com/source",
  "domain": "AI-Powered Professional Skills",
  "second_domain": "AI Programming",
  "tags": ["Code Generation", "AI-assisted Debugging"],
  "tier": "Tier 2: noteworthy"
}

成功响应：
{
  "success": true,
  "data": {
    "id": "rec789012",
    "title": "情报标题",
    "insight": "核心洞察内容"
  },
  "message": "情报提交成功，等待审核",
  "code": 201
}
```

## 认证相关 API

### 用户登录
**用途**：基于用户ID的简化认证流程，用户只需提供唯一标识符即可登录系统，无需复杂的密码验证。

```http
POST /api/auth/login
Content-Type: application/json

请求体：
{
  "id": "user123"    // 用户唯一标识符
}

成功响应：
{
  "success": true,
  "user": {
    "id": "user123",
    "displayName": "用户名称",
    "role": "user",
    "userLevel": "both",
    "avatarURL": "https://example.com/avatar.jpg",
    "email": "user@example.com"
  },
  "message": "登录成功"
}
```

### 用户登出
**用途**：用户登出

```http
POST /api/auth/logout

成功响应：
{
  "success": true,
  "data": null,
  "message": "登出成功",
  "code": 200
}
```

### 获取当前用户信息
**用途**：验证并获取当前登录用户的详细信息，用于前端界面显示和权限判断。

```http
GET /api/auth/me

成功响应：
{
  "success": true,
  "data": {
    "user": {
      "id": "user123",
      "displayName": "用户名称",
      "role": "admin"
    },
    "isAdmin": true
  },
  "message": "获取成功",
  "code": 200
}
```

## 投票相关 API

### 情报投票
**用途**：用户对情报内容进行投票表达喜好，支持点赞和点踩操作，用于社区内容质量评估。

```http
POST /api/vote
Content-Type: application/json

请求体：
{
  "recordId": "rec123456",  // 情报记录ID
  "voteType": "up"         // 投票类型：up/down
}

成功响应：
{
  "success": true,
  "message": "投票成功",
  "newVoteCount": 16
}
```

## 评论相关 API

### 获取情报评论
**用途**：获取指定情报的所有评论数据，包括主评论和回复评论，支持层级化的评论结构展示。

```http
GET /api/discussion/[intelligenceId]

成功响应：
{
  "success": true,
  "comments": [
    {
      "id": "com123",
      "intelligenceId": "rec123456",
      "userId": "user456",
      "userName": "李四",
      "content": "这个洞察很有价值！",
      "level": 1,
      "likes": 3,
      "createdAt": "2024-01-01T12:00:00Z",
      "updatedAt": "2024-01-01T12:00:00Z",
      "replies": [
        {
          "id": "com124",
          "intelligenceId": "rec123456",
          "userId": "user789",
          "userName": "王五",
          "content": "我同意这个观点",
          "level": 2,
          "likes": 1,
          "createdAt": "2024-01-01T13:00:00Z",
          "updatedAt": "2024-01-01T13:00:00Z"
        }
      ]
    }
  ],
  "stats": {
    "totalComments": 2,
    "totalParticipants": 3,
    "lastActivity": "2024-01-01T13:00:00Z"
  },
  "message": "获取成功"
}
```

### 添加评论
**用途**：用户为情报添加新评论或回复现有评论，支持层级化的讨论结构。

```http
POST /api/discussion/[intelligenceId]
Content-Type: application/json

请求体：
{
  "content": "评论内容",    // 评论正文，支持Markdown
  "parentId": "com123",    // 父评论ID（可选，用于回复）
  "userId": "user123",     // 评论者用户ID
  "userName": "当前用户"   // 评论者显示名称
}

成功响应：
{
  "success": true,
  "comment": {
    "id": "com125",
    "intelligenceId": "rec123456",
    "userId": "user123",
    "userName": "当前用户",
    "content": "评论内容",
    "level": 1,
    "likes": 0,
    "createdAt": "2024-01-01T14:00:00Z",
    "updatedAt": "2024-01-01T14:00:00Z"
  },
  "message": "评论成功"
}
```

## 审核相关 API

### 内容审核
**用途**：管理员对用户提交的情报内容进行审核，决定是否批准发布、拒绝或标记问题内容。

```http
POST /api/review
Content-Type: application/json
Authorization: Bearer [admin-token]

请求体：
{
  "intelligenceId": "rec789012",  // 待审核情报ID
  "decision": "approve",         // 审核决定：approve/reject/flag
  "reason": "内容质量很高"         // 审核原因说明（可选）
}

成功响应：
{
  "success": true,
  "data": {
    "reviewStatus": "approved",
    "reviewDecision": "human approved"
  },
  "message": "审核完成",
  "code": 200
}
```

## 错误处理

### 常见错误码
```
200 - 成功
201 - 创建成功
400 - 请求参数错误
401 - 未授权
403 - 权限不足
404 - 资源不存在
500 - 服务器内部错误
```

### 错误响应格式
```json
{
  "success": false,
  "error": "具体错误信息"
}
```

**错误处理原则**：
- 统一格式：所有API错误都遵循相同的响应格式
- 明确提示：错误信息要具体明确，帮助用户理解问题
- 状态码对应：HTTP状态码与业务错误码保持一致
- 安全考虑：错误信息不包含敏感的系统信息
```

## 修改指南

### ✅ 安全的修改
- 添加新的可选字段
- 增加新的 API 端点
- 优化响应数据结构（向后兼容）

### ⚠️ 需要谨慎的修改
- 修改现有字段的数据类型
- 删除响应中的字段
- 修改错误码定义

### 🚨 危险操作（需要版本管理）
- 删除 API 端点
- 修改请求参数结构
- 修改响应格式
