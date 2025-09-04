# 数据模型设计 (Data Model Design)

## 概述

Alpha Seeker 是一个专注于发现和分享有价值洞察的情报聚合平台。系统采用三层分类体系和个人能力提升导向，从海量信息中筛选出真正有价值的洞察，帮助用户追踪重要趋势，发现机会。

当前系统使用 **Airtable** 作为主要数据源，本文档定义了完整的数据模型和字段规范。

---

## 🏗️ 核心设计理念

### 情报策展模式
- **价值发现**：从海量信息中筛选真正有价值的洞察
- **三级分类**：Domain → Second Domain → Tags 的精准分类体系
- **社区验证**：通过投票和评论机制验证内容质量
- **持续进化**：内容随时间和反馈不断优化更新

### 个人能力提升导向
系统采用三级分类体系，所有内容都围绕"提升个人能力"这一核心目标：

1. **Domain** (3个个人能力提升大类)
2. **Second_Domain** (7个核心能力模块)  
3. **Tags** (35个具体技能标签)

### 重要性分级
系统采用Tier分级来标识内容的重要程度：
- **Tier 1: must read** - 必读内容，核心洞察
- **Tier 2: noteworthy** - 值得关注的重要内容
- **Tier 3: interesting** - 有趣的补充内容

---

## 📊 Airtable 表结构设计

### Intelligence 表（核心内容表）

#### 1. 核心内容字段

| 字段名 | 代码引用 | 数据类型 | 必填 | 说明 |
|--------|----------|----------|------|------|
| Title | `fields.Title` | 单行文本 | ✅ | 情报标题 |
| Insight | `fields.Insight` | 长文本 | ✅ | 核心洞察内容 |
| Author | `fields.Author` | 单行文本 | ✅ | 作者名称 |
| Author_Link | `fields.Author_Link` | URL | ❌ | 作者链接 |
| Source_Context | `fields.Source_Context` | 单行文本 | ✅ | 来源上下文 |
| Source_Link | `fields.Source_Link` | URL | ❌ | 原文链接 |

#### 2. 个人能力提升分类字段

| 字段名 | 代码引用 | 数据类型 | 必填 | 说明 |
|--------|----------|----------|------|------|
| Domain | `fields.Domain` | 单选 | ❌ | 个人能力提升大类 |
| Second_domain | `fields.Second_domain` | 单选 | ❌ | 核心能力模块 |
| Tier | `fields.Tier` | 单选 | ✅ | 重要程度分级 |
| Tags | `fields.Tags` | 文本(JSON) | ✅ | 标准化标签数组 |
| User_Level | `fields.User_Level` | 单选 | ❌ | 适用用户级别 |

#### 3. 互动与质量字段

| 字段名 | 代码引用 | 数据类型 | 必填 | 说明 |
|--------|----------|----------|------|------|
| VoteCount | `fields.VoteCount` | 数字 | ✅ | 投票数 |
| Quality_Score | `fields.Quality_Score` | 数字 | ❌ | 质量评分 |
| Community_Notes | `fields.Community_Notes` | 长文本(JSON) | ❌ | 社区笔记 |

#### 4. 时间相关字段

| 字段名 | 代码引用 | 数据类型 | 必填 | 说明 |
|--------|----------|----------|------|------|
| Published_At | `fields.Published_At` | 日期时间 | ✅ | 发布时间 |
| Featured_Date | `fields.Featured_Date` | 日期 | ❌ | 精选日期 |

#### 5. 战略价值字段

| 字段名 | 代码引用 | 数据类型 | 必填 | 说明 |
|--------|----------|----------|------|------|
| Positioning_Context | `fields.Positioning_Context` | 长文本 | ❌ | 定位上下文 |


---

## 🔧 字段选项值定义

### Domain 字段选项（个人能力提升大类）
```
- AI-Powered Professional Skills
- Entrepreneurial Skills  
- Product & Design Skills
```

### Second_Domain 字段选项（核心能力模块）
```
// AI-Powered Professional Skills
- AI Programming
- AI Workflow Automation
- AI Content & Creation

// Entrepreneurial Skills
- Founder & Fundraising
- Growth & Monetization

// Product & Design Skills
- AI Product Management
- AI-native UX/UI
```

### Tier 字段选项
```
- Tier 1: must read
- Tier 2: noteworthy
- Tier 3: interesting
```

### User_Level 字段选项
```
- newcomer
- veteran
- both
```

### Tags 字段选项（35个个人能力提升标签）

#### AI-Powered Professional Skills (15个)
```
// AI Programming
- Code Generation
- AI-assisted Debugging
- Automated Testing
- Multi-agent Development
- Prompt-driven Development

// AI Workflow Automation
- Personal Automation
- No-code Development
- API Integration
- Agentic Task Management
- Workflow Optimization

// AI Content & Creation
- Prompt Engineering
- AI-assisted Writing
- AI-assisted Design
- AI-powered Research
- Knowledge Management
```

#### Entrepreneurial Skills (10个)
```
// Founder & Fundraising
- Fundraising Strategy
- Pitch Deck Crafting
- Valuation
- Term Sheet Negotiation
- Investor Relations

// Growth & Monetization
- Growth Hacking
- User Acquisition
- Monetization Models
- SaaS Metrics
- Community Building
```

#### Product & Design Skills (10个)
```
// AI Product Management
- AI Product Strategy
- AI-native Roadmapping
- Data-driven Decisions
- Agile for AI
- Feature Prioritization

// AI-native UX/UI
- Design for AI
- Prototyping AI Interfaces
- Human-AI Interaction
- Conversational UI
- Ethical AI Design
```

---

## 📝 TypeScript 类型定义

### 核心类型定义

```typescript
// 用户级别类型
export type UserLevel = 'newcomer' | 'veteran' | 'both';

// 时间过滤器类型
export type TimeFilter = 'today' | 'week' | 'month' | 'sixMonths' | 'all';

// 内容重要性分级
export type TierLevel = 'Tier 1: must read' | 'Tier 2: noteworthy' | 'Tier 3: interesting';

// 个人能力提升大类
export type DomainLevel = 
  | 'AI-Powered Professional Skills'
  | 'Entrepreneurial Skills' 
  | 'Product & Design Skills';

// 核心能力模块
export type SecondDomainLevel = 
  | 'AI Programming' | 'AI Workflow Automation' | 'AI Content & Creation'
  | 'Founder & Fundraising' | 'Growth & Monetization'
  | 'AI Product Management' | 'AI-native UX/UI';

// 情报条目数据模型
export interface IntelligenceItem {
  // 核心标识
  id: string;
  
  // 内容展示
  title: string;
  insight: string;
  
  // 内容归属
  author: string;
  author_link?: string;
  source_context: string;
  source_link?: string;
  
  // 三级分类体系
  domain: DomainLevel;
  second_domain: SecondDomainLevel;
  tier: TierLevel;
  tags: string[];
  
  // 社区互动
  vote_count: number;
  quality_score: number;
  
  // 时间相关
  published_at: Date;
  
  // 信号策展
  user_level: UserLevel;
  positioning_context: string;
  
  // 扩展元数据
  featured_date?: Date;
  community_notes: string[];
  
  // 扩展字段
  extensions?: {
    learning_path?: {
      path_id?: string;
      step_order?: number;
      difficulty_level?: number;
      expected_outcome?: string;
      time_to_complete?: number;
      prerequisite_knowledge?: string[];
    };
    recommendation?: {
      score?: number;
      reason?: string;
    };
    community?: {
      quality_score?: number;
      expert_validations?: string[];
      discussion_count?: number;
      success_indicators?: string[];
    };
  };
}

// 筛选状态类型
export interface FilterState {
  timeFilter: TimeFilter;
  selectedTags: string[];
  userLevel: UserLevel;
}

// 增强筛选状态
export interface EnhancedFilterState {
  selectedDomain: string;
  selectedSecondDomain: string;
  selectedTags: string[];
  availableTags: string[];
  availableSecondDomains: string[];
  timeFilter: TimeFilter;
  userLevel: UserLevel;
}

// Airtable记录类型
export interface AirtableIntelligenceRecord {
  id: string;
  fields: {
    Title: string;
    Insight: string;
    Author: string;
    Author_Link?: string;
    Source_Context: string;
    Source_Link?: string;
    Domain?: string;
    Second_domain?: string;
    Tier: string;
    Tags: string;
    VoteCount: number;
    Published_At: string;
    
    // 信号策展字段
    User_Level?: string;
    Positioning_Context?: string;
    Quality_Score?: number;
    Featured_Date?: string;
    Community_Notes?: string;
  };
}

export interface AirtableResponse {
  records: AirtableIntelligenceRecord[];
  offset?: string;
}
```


---

## 🔄 数据流转逻辑

### 内容处理流程
```
情报提交 → AI审核 → 分类标签化 → 存储到Airtable → 社区验证
```

### 个人能力提升三级映射
```
Domain (3个大类) → Second_Domain (7个模块) → Tags (35个具体技能)
```

### 字段关联关系
1. **Domain** → **Second_domain** → **Tags** 形成三级分类体系
2. **VoteCount** 和 **Quality_Score** 反映内容质量
3. **User_Level** 和 **Positioning_Context** 提供策展上下文
4. **Community_Notes** 支持社区协作增强

---

## 🚀 数据访问层设计

### IntelligenceService 类

**数据访问层核心服务类，负责与Airtable数据库的所有交互操作**

**职责范围**：
- 情报数据的CRUD操作
- Airtable API的封装和错误处理
- 数据格式转换和标准化
- 投票和社区互动功能

**数据流转**：前端组件 → IntelligenceService → Airtable API → 数据库

```typescript
export class IntelligenceService {
  private baseId: string;           // Airtable Base ID，用于API认证
  private accessToken: string;     // Airtable访问令牌，用于API认证

  constructor() {
    this.baseId = process.env.AIRTABLE_BASE_ID || '';
    this.accessToken = process.env.AIRTABLE_ACCESS_TOKEN || '';
  }

  /**
   * 获取情报数据列表（支持过滤和分页）
   * 
   * 功能：从Airtable获取情报数据，支持复杂的过滤条件和分页
   * 数据流转：Airtable API → JSON响应 → 标准化返回格式
   * 
   * @param options 可选参数
   * @param options.filter Airtable过滤公式，支持复杂查询
   * @param options.maxRecords 最大返回记录数
   * @param options.offset 分页偏移量
   * @returns Promise<AirtableResponse> 包含记录数组和分页信息
   */
  async getIntelligence(options?: {
    filter?: string;
    maxRecords?: number;
    offset?: string;
  }): Promise<AirtableResponse> {
    const url = `https://api.airtable.com/v0/${this.baseId}/Intelligence`;
    const params = new URLSearchParams();
    
    if (options?.filter) params.append('filterByFormula', options.filter);
    if (options?.maxRecords) params.append('maxRecords', options.maxRecords.toString());
    if (options?.offset) params.append('offset', options.offset);

    const response = await fetch(`${url}?${params}`, {
      headers: {
        'Authorization': `Bearer ${this.accessToken}`,
      },
    });

    if (!response.ok) {
      throw new Error(`Airtable API error: ${response.status} ${response.statusText}`);
    }

    return response.json();
  }

  /**
   * 根据用户级别获取个性化情报内容
   * 
   * 功能：实现用户级别的个性化内容推荐，支持新手/老手/全部用户
   * 业务逻辑：通过Airtable过滤公式实现内容分级，确保用户看到最适合的内容
   * 数据流转：用户级别 → 过滤条件 → Airtable查询 → 标准化情报数据
   * 
   * @param userLevel 用户级别：'newcomer'(新手) | 'veteran'(老手) | 'both'(全部)
   * @returns Promise<IntelligenceItem[]> 适合该用户级别的情报数组
   */
  async getUserLevelIntelligence(userLevel: UserLevel): Promise<IntelligenceItem[]> {
    const filter = userLevel === 'both' 
      ? `OR({User_Level} = 'newcomer', {User_Level} = 'veteran', {User_Level} = 'both')`
      : `OR({User_Level} = '${userLevel}', {User_Level} = 'both')`;

    const data = await this.getIntelligence({ filter });
    return data.records.map(record => this.transformAirtableToIntelligenceItem(record));
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
   * 创建新情报内容（用户提交功能）
   * 
   * 功能：实现用户情报提交功能，支持完整的元数据设置
   * 业务逻辑：自动设置默认值（Tier、发布时间等），确保数据完整性
   * 数据流转：用户输入 → 数据验证 → Airtable创建 → 标准化返回
   * 
   * @param data 情报数据对象
   * @param data.title 情报标题
   * @param data.insight 核心洞察内容
   * @param data.author 作者名称
   * @param data.sourceContext 来源上下文
   * @param data.sourceLink 原文链接（可选）
   * @param data.domain 能力领域（可选，默认AI技能）
   * @param data.secondDomain 技能模块（可选，默认AI编程）
   * @param data.tags 标签数组（可选）
   * @param data.userLevel 用户级别（可选，默认全部用户）
   * @returns Promise<IntelligenceItem> 创建成功的情报对象
   */
  async createIntelligence(data: {
    title: string;
    insight: string;
    author: string;
    sourceContext: string;
    sourceLink?: string;
    domain?: string;
    secondDomain?: string;
    tags?: string[];
    userLevel?: UserLevel;
  }): Promise<IntelligenceItem> {
    const url = `https://api.airtable.com/v0/${this.baseId}/Intelligence`;
    
    const newRecord = {
      fields: {
        Title: data.title,
        Insight: data.insight,
        Author: data.author,
        Source_Context: data.sourceContext,
        Source_Link: data.sourceLink || '',
        Tags: JSON.stringify(data.tags || []),
        Domain: data.domain || 'AI-Powered Professional Skills',
        Second_domain: data.secondDomain || 'AI Programming',
        Tier: 'Tier 3: interesting',
        VoteCount: 0,
        Published_At: new Date().toISOString(),
        User_Level: data.userLevel || 'both',
      },
    };

    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.accessToken}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(newRecord),
    });

    if (!response.ok) {
      throw new Error(`创建失败: ${response.status} ${response.statusText}`);
    }

    const result = await response.json();
    return this.transformAirtableToIntelligenceItem(result);
  }

  /**
   * Airtable记录转换为标准化情报对象
   * 
   * 功能：将Airtable API返回的原始记录转换为前端使用的标准化数据格式
   * 业务逻辑：处理多种数据格式（JSON、字符串、数组），确保数据一致性
   * 数据流转：Airtable记录 → 字段解析 → 格式标准化 → 类型转换 → 前端对象
   * 
   * @param record Airtable API返回的原始记录
   * @returns IntelligenceItem 标准化的情报对象
   */
  private transformAirtableToIntelligenceItem(record: AirtableIntelligenceRecord): IntelligenceItem {
    const fields = record.fields;
    
    // 解析JSON字段，处理可能的格式错误
    const parseJsonField = (field: string | undefined, fallback: string[] = []): string[] => {
      if (!field) return fallback;
      try {
        return JSON.parse(field);
      } catch {
        return fallback;
      }
    };

    // 支持多种标签格式：数组、JSON字符串、逗号分隔
    const parseTags = (tags: string | string[]): string[] => {
      if (Array.isArray(tags)) return tags;
      if (typeof tags === 'string') {
        try {
          return JSON.parse(tags);
        } catch {
          return tags.split(',').map(tag => tag.trim()).filter(Boolean);
        }
      }
      return [];
    };

    // 标准化Tier格式，支持多种输入格式
    const normalizeTier = (tier: string): TierLevel => {
      if (tier.includes('1') || tier.toLowerCase().includes('must')) {
        return 'Tier 1: must read';
      } else if (tier.includes('2') || tier.toLowerCase().includes('noteworthy')) {
        return 'Tier 2: noteworthy';
      } else {
        return 'Tier 3: interesting';
      }
    };

    return {
      id: record.id,
      title: fields.Title,
      insight: fields.Insight,
      author: fields.Author,
      author_link: fields.Author_Link,
      source_context: fields.Source_Context,
      source_link: fields.Source_Link,
      domain: (fields.Domain as DomainLevel) || 'AI-Powered Professional Skills',
      second_domain: (fields.Second_domain as SecondDomainLevel) || 'AI Programming',
      tier: normalizeTier(fields.Tier),
      tags: parseTags(fields.Tags),
      vote_count: fields.VoteCount || 0,
      quality_score: fields.Quality_Score || 0,
      published_at: new Date(fields.Published_At),
      user_level: (fields.User_Level as UserLevel) || 'both',
      positioning_context: fields.Positioning_Context || '',
      featured_date: fields.Featured_Date ? new Date(fields.Featured_Date) : undefined,
      community_notes: parseJsonField(fields.Community_Notes),
    };
  }

  /**
   * 根据ID获取单个情报记录
   * 
   * 功能：通过唯一标识符获取特定的情报记录，用于投票、编辑等操作
   * 业务逻辑：处理404错误，支持记录不存在的情况，确保调用方的健壮性
   * 数据流转：记录ID → Airtable查询 → 记录存在性检查 → 返回记录或null
   * 
   * @param id 情报记录的唯一标识符
   * @returns Promise<AirtableIntelligenceRecord | null> 找到的记录或null
   */
  private async getIntelligenceById(id: string): Promise<AirtableIntelligenceRecord | null> {
    try {
      const url = `https://api.airtable.com/v0/${this.baseId}/Intelligence/${id}`;
      const response = await fetch(url, {
        headers: {
          'Authorization': `Bearer ${this.accessToken}`,
        },
      });

      if (!response.ok) {
        if (response.status === 404) {
          return null;
        }
        throw new Error(`获取失败: ${response.status} ${response.statusText}`);
      }

      return await response.json();
    } catch (error) {
      console.error('获取情报失败:', error);
      return null;
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

  getAccessToken(): string {
    return this.accessToken;
  }
}
```

---

---

## 📝 注意事项

1. **字段名称必须与代码完全一致**，包括大小写
2. **选项值必须严格按照上述定义**，避免拼写错误
3. **Tags字段使用JSON字符串格式**存储数组
4. **日期时间字段使用ISO格式**存储
5. **URL字段必须包含完整的协议**（http:// 或 https://）
6. **个人能力提升标签必须使用标准化定义**，确保分类体系的一致性
7. **Domain和Second_domain字段为可选**，系统会根据标签自动推断
8. **Tier字段标准化处理**：支持多种格式输入，统一转换为标准格式