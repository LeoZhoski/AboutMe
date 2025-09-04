# 代码注释示例与模板

本文档提供详细的代码注释示例和模板，作为 `README.md` 中注释标准的补充参考。

---

## 📋 快速导航

- [基础注释示例](#基础注释示例)
- [复杂组件注释标准](#复杂组件注释标准)
- [数据访问层示例](#数据访问层示例)
- [业务逻辑层示例](#业务逻辑层示例)
- [注释模板库](#注释模板库)

---

## 基础注释示例

### ✅ 有价值的注释（推荐）

```typescript
// 用户反馈分析：从多个维度评估反馈内容，为产品改进提供数据支持
async analyzeUserFeedback(contentId: string): Promise<FeedbackAnalysis> {
  const feedback = await this.getFeedback(contentId);
  
  return {
    sentiment: this.analyzeSentiment(feedback),          // 情感倾向分析
    commonThemes: this.extractThemes(feedback),           // 主题提取
    suggestedImprovements: this.extractSuggestions(feedback), // 改进建议
    engagementLevel: this.calculateEngagement(feedback)   // 参与度评估
  };
}
```

### ❌ 冗余注释（避免）

```typescript
async voteIntelligence(id: string, voteType: 'up' | 'down'): Promise<VoteResult> {
  try {
    // 获取当前记录  ← 冗余：代码已经很清楚
    const record = await this.getIntelligenceById(id);
    
    const currentVotes = record.fields.VoteCount || 0;
    const newVoteCount = voteType === 'up' ? currentVotes + 1 : Math.max(0, currentVotes - 1);

    // 更新记录  ← 冗余：fetch调用明显是在更新
    const updateUrl = `https://api.airtable.com/v0/${this.baseId}/Intelligence/${id}`;
    // ...
  }
}
```

---

## 复杂组件注释标准

### 1. 类级别注释

**适用于**：数据访问层、业务逻辑层、核心服务类

```typescript
/**
 * 数据访问层核心服务类，负责与Airtable数据库的所有交互操作
 * 
 * 职责范围：
 * - 情报数据的CRUD操作
 * - Airtable API的封装和错误处理
 * - 数据格式转换和标准化
 * - 投票和社区互动功能
 * 
 * 数据流转：前端组件 → IntelligenceService → Airtable API → 数据库
 * 
 * @example
 * const service = new IntelligenceService();
 * const intelligence = await service.getUserLevelIntelligence('newcomer');
 */
export class IntelligenceService {
  private baseId: string;           // Airtable Base ID，用于API认证
  private accessToken: string;     // Airtable访问令牌，用于API认证
}
```

### 2. 方法级别注释

**适用于**：公共方法、复杂的私有方法、业务逻辑方法

```typescript
/**
 * 根据用户级别获取个性化情报内容
 * 
 * 功能：实现用户级别的个性化内容推荐，支持新手/老手/全部用户
 * 业务逻辑：通过Airtable过滤公式实现内容分级，确保用户看到最适合的内容
 * 数据流转：用户级别 → 过滤条件 → Airtable查询 → 标准化情报数据
 * 性能考虑：使用Airtable过滤公式而非内存过滤，提高大数据量下的性能
 * 
 * @param userLevel 用户级别：'newcomer'(新手) | 'veteran'(老手) | 'both'(全部)
 * @returns Promise<IntelligenceItem[]> 适合该用户级别的情报数组
 * @throws {Error} 当Airtable API调用失败时抛出异常
 * 
 * @example
 * // 获取新手友好的内容
 * const newcomerContent = await service.getUserLevelIntelligence('newcomer');
 * 
 * // 获取所有内容
 * const allContent = await service.getUserLevelIntelligence('both');
 */
async getUserLevelIntelligence(userLevel: UserLevel): Promise<IntelligenceItem[]> {
  // 实现代码...
}
```

### 3. 数据转换方法注释

**适用于**：API数据转换、格式化、序列化等方法

```typescript
/**
 * Airtable记录转换为标准化情报对象
 * 
 * 功能：将Airtable API返回的原始记录转换为前端使用的标准化数据格式
 * 业务逻辑：处理多种数据格式（JSON、字符串、数组），确保数据一致性
 * 数据流转：Airtable记录 → 字段解析 → 格式标准化 → 类型转换 → 前端对象
 * 错误处理：对JSON解析错误进行容错处理，提供默认值
 * 
 * @param record Airtable API返回的原始记录
 * @returns IntelligenceItem 标准化的情报对象
 * 
 * @example
 * const rawRecord = await airtableAPI.getRecord('rec123');
 * const intelligence = service.transformAirtableToIntelligenceItem(rawRecord);
 */
private transformAirtableToIntelligenceItem(record: AirtableIntelligenceRecord): IntelligenceItem {
  // 实现代码...
}
```

---

## 数据访问层示例

### 完整的服务类示例

```typescript
/**
 * 情报数据访问服务
 * 
 * 职责：负责与Airtable数据库的所有交互操作
 * 数据流转：前端组件 → IntelligenceService → Airtable API → 数据库
 * 设计考虑：使用直接API调用而非后端路由，减少网络延迟
 */
export class IntelligenceService {
  private baseId: string;
  private accessToken: string;

  constructor() {
    this.baseId = process.env.AIRTABLE_BASE_ID || '';
    this.accessToken = process.env.AIRTABLE_ACCESS_TOKEN || '';
  }

  /**
   * 情报投票功能（支持点赞/点踩）
   * 
   * 功能：实现社区投票机制，让用户对情报内容进行质量评估
   * 业务逻辑：支持点赞和点踩操作，防止投票数变为负数，确保数据一致性
   * 性能优化：直接更新Airtable记录，绕过API路由，减少网络开销
   * 数据流转：用户投票 → 获取当前记录 → 计算新投票数 → 更新数据库 → 返回结果
   * 
   * @param id 情报记录的唯一标识符
   * @param voteType 投票类型：'up'(点赞) | 'down'(点踩)
   * @returns Promise<VoteResult> 投票结果，包含成功状态和新的投票数
   */
  async voteIntelligence(id: string, voteType: 'up' | 'down'): Promise<VoteResult> {
    try {
      // 获取当前记录并计算新投票数
      // 防止投票数变为负数，确保数据一致性
      const record = await this.getIntelligenceById(id);
      if (!record) {
        return { success: false, error: '记录未找到' };
      }

      const currentVotes = record.fields.VoteCount || 0;
      const newVoteCount = voteType === 'up' ? currentVotes + 1 : Math.max(0, currentVotes - 1);

      // 直接更新Airtable记录，而非通过API路由
      // 减少一层网络请求，提高性能
      const updateUrl = `https://api.airtable.com/v0/${this.baseId}/Intelligence/${id}`;
      const response = await fetch(updateUrl, {
        method: 'PATCH',
        headers: {
          'Authorization': `Bearer ${this.accessToken}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          fields: {
            VoteCount: newVoteCount,
          },
        }),
      });

      if (!response.ok) {
        throw new Error(`更新失败: ${response.status} ${response.statusText}`);
      }

      return { success: true, newVoteCount };
    } catch (error) {
      return { 
        success: false, 
        error: error instanceof Error ? error.message : '投票失败' 
      };
    }
  }

  /**
   * 调试辅助方法，便于问题排查和开发调试
   * 
   * 功能：提供内部状态的访问接口，便于开发调试和问题排查
   * 注意：这些方法主要用于开发环境，生产环境中应该谨慎使用
   */
  getBaseId(): string {
    return this.baseId;
  }
}
```

---

## 业务逻辑层示例

### 业务规则处理示例

```typescript
/**
 * 用户权限验证服务
 * 
 * 职责：处理用户权限验证和访问控制逻辑
 * 业务规则：基于用户角色、内容级别、时间限制等多维度验证
 * 数据流转：用户请求 → 权限验证 → 规则检查 → 访问决策
 */
export class AuthorizationService {
  /**
   * 验证用户是否可以访问特定内容
   * 
   * 功能：基于多维度规则验证用户访问权限
   * 业务规则：
   * 1. 管理员可以访问所有内容
   * 2. 普通用户只能访问符合其级别的内容
   * 3. 特定内容有时间限制访问
   * 
   * @param user 用户信息
   * @param content 内容信息
   * @returns boolean 是否允许访问
   */
  canAccessContent(user: User, content: IntelligenceItem): boolean {
    // 管理员拥有所有权限
    if (user.role === 'admin') {
      return true;
    }

    // 检查用户级别匹配
    if (content.user_level !== 'both' && content.user_level !== user.userLevel) {
      return false;
    }

    // 检查时间限制（如果存在）
    if (content.accessRestriction?.expiresAt) {
      const now = new Date();
      const expiresAt = new Date(content.accessRestriction.expiresAt);
      if (now > expiresAt) {
        return false;
      }
    }

    return true;
  }
}
```

---

## 注释模板库

### 1. 通用方法模板

```typescript
/**
 * [方法功能概述]
 * 
 * 功能：[具体的功能说明]
 * 业务逻辑：[关键的业务规则和设计决策]
 * 性能考虑：[性能相关的考虑和优化策略]
 * 数据流转：[完整的数据处理流程]
 * 错误处理：[异常情况的处理方式]
 * 
 * @param paramName [参数说明]
 * @param paramName [参数说明]
 * @returns [返回值说明]
 * @throws {ErrorType} [可能抛出的异常说明]
 * 
 * @example
 * // 使用示例
 * const result = methodName(param1, param2);
 */
```

### 2. 数据转换模板

```typescript
/**
 * [源格式]转换为[目标格式]
 * 
 * 功能：将[源格式]数据转换为[目标格式]
 * 业务逻辑：[转换的业务规则和考虑]
 * 数据流转：[数据转换的完整流程]
 * 错误处理：[格式错误的处理方式]
 * 
 * @param input [输入参数说明]
 * @returns [输出结果说明]
 * 
 * @example
 * const converted = transformMethod(inputData);
 */
```

### 3. API调用模板

```typescript
/**
 * [API功能描述]
 * 
 * 功能：调用外部API实现[具体功能]
 * 业务逻辑：[API调用的业务背景]
 * 数据流转：[请求参数 → API调用 → 响应处理 → 返回结果]
 * 错误处理：[API错误和超时的处理方式]
 * 性能考虑：[缓存、重试等优化策略]
 * 
 * @param param [参数说明]
 * @returns Promise<[返回类型>] [返回结果说明]
 * @throws {NetworkError} 网络错误
 * @throws {ApiError} API错误
 */
```

### 4. 事件处理模板

```typescript
/**
 * [事件处理功能]
 * 
 * 功能：处理[具体事件]的用户交互
 * 业务逻辑：[事件触发的业务规则]
 * 用户体验：[交互设计考虑]
 * 副作用：[可能产生的副作用]
 * 
 * @param event [事件参数]
 * @returns void
 */
```

---

## 注释检查清单

### ✅ 注释完整性检查

- [ ] 方法是否有清晰的功能说明
- [ ] 是否说明了业务逻辑和设计决策
- [ ] 是否描述了数据流转过程
- [ ] 参数和返回值是否有完整说明
- [ ] 是否包含错误处理说明
- [ ] 复杂逻辑是否有行内注释
- [ ] 性能关键点是否有说明
- [ ] 是否提供了使用示例

### ❌ 避免的注释问题

- [ ] 不要注释显而易见的代码
- [ ] 避免过时的注释
- [ ] 不要使用模糊不清的描述
- [ ] 避免与代码矛盾的注释
- [ ] 不要过度注释简单逻辑

---

## 不同角色的注释需求

### 产品经理关注点
- 业务目标和价值
- 数据流转和业务规则
- 用户体验考虑
- 功能边界和限制

### 开发者关注点
- 技术实现细节
- 参数和返回值
- 错误处理方式
- 性能考虑

### 维护者关注点
- 设计决策背景
- 业务规则逻辑
- 数据流转路径
- 潜在的修改影响

---

**相关文档**：
- [README.md](./README.md) - 基础注释标准和原则
- [data-model.md](./1_data_and_state/data-model.md) - 数据模型注释示例