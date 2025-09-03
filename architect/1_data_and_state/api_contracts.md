# API 数据契约 (API Contracts)

## 概述
定义Alpha Seeker情报聚合平台的前后端数据交换标准格式，包括请求参数、响应结构和错误处理机制。

## 设计原则
- **一致性**：所有 API 遵循统一的数据格式和命名规范
- **可预测性**：明确的输入输出规范，减少集成错误
- **可维护性**：清晰的文档便于团队协作和项目维护
- **安全性**：适当的认证和授权机制

## 通用响应格式

所有 API 响应都遵循统一格式：

```json
{
  "success": true,          // 请求是否成功（true/false）
  "data": {},              // 实际数据（成功时有值）
  "message": "操作成功",    // 提示信息
  "code": 200,             // 状态码
  "timestamp": 1640995200  // 响应时间戳
}
```

**字段说明**：
- `success`: 布尔值，标识请求处理结果
- `data`: 实际业务数据，成功时包含请求的数据
- `message`: 人类可读的操作结果描述
- `code`: HTTP 状态码，用于程序化处理
- `timestamp`: Unix 时间戳，便于调试和日志分析

## 情报相关 API

### 获取情报列表
**用途**：获取所有情报内容（支持用户级别过滤）

```http
GET /api/intelligence?userLevel=both

查询参数：
- userLevel: 用户级别（newcomer/veteran/both，默认both）

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
**用途**：简化版用户认证（基于ID）

```http
POST /api/auth/login
Content-Type: application/json

请求体：
{
  "id": "user123"    // 用户标识符
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
**用途**：获取当前登录用户信息

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
**用途**：用户对情报进行投票

```http
POST /api/vote
Content-Type: application/json

请求体：
{
  "recordId": "rec123456",
  "voteType": "up"    // up/down
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
**用途**：获取指定情报的所有评论

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
**用途**：为情报添加评论

```http
POST /api/discussion/[intelligenceId]
Content-Type: application/json

请求体：
{
  "content": "评论内容",
  "parentId": "com123",    // 可选，回复指定评论时提供
  "userId": "user123",   // 用户标识符
  "userName": "当前用户" // 用户显示名称
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
**用途**：管理员审核提交的情报

```http
POST /api/review
Content-Type: application/json
Authorization: Bearer [admin-token]

请求体：
{
  "intelligenceId": "rec789012",
  "decision": "approve",    // approve/reject/flag
  "reason": "内容质量很高"    // 可选，审核原因
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
