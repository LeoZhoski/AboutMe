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
// 核心用户级别类型
export type UserLevel = 'newcomer' | 'veteran' | 'both';
export type TimeFilter = 'today' | 'week' | 'month' | 'sixMonths' | 'all';
export type TierLevel = 'Tier 1: must read' | 'Tier 2: noteworthy' | 'Tier 3: interesting';

// 个人能力提升大类
export type DomainLevel = 
  | 'AI-Powered Professional Skills'
  | 'Entrepreneurial Skills' 
  | 'Product & Design Skills';

// 核心能力模块
export type SecondDomainLevel = 
  // AI-Powered Professional Skills
  | 'AI Programming' | 'AI Workflow Automation' | 'AI Content & Creation'
  // Entrepreneurial Skills
  | 'Founder & Fundraising' | 'Growth & Monetization'
  // Product & Design Skills
  | 'AI Product Management' | 'AI-native UX/UI';

// 简化的情报条目模型
export interface IntelligenceItem {
  // 核心标识
  id: string;
  
  // 展示内容
  title: string;
  insight: string;
  
  // 归属信息
  author: string;
  author_link?: string;
  source_context: string;
  source_link?: string;
  
  // 分类
  domain: DomainLevel;
  second_domain: SecondDomainLevel;
  tier: TierLevel;
  tags: string[];
  
  // 参与度指标
  vote_count: number;
  quality_score: number; // 社区驱动的质量评估
  
  // 时间数据
  published_at: Date;
  
  // 信号策展字段
  user_level: UserLevel;
  positioning_context: string; // 在大局中的定位
  
  // 可选元数据
  featured_date?: Date; // 何时被精选/高亮
  community_notes: string[]; // 社区添加的洞察
  
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

// 筛选和状态管理类型
export interface FilterState {
  timeFilter: TimeFilter;
  selectedTags: string[];
  userLevel: UserLevel;
}

// 增强筛选状态（三级筛选系统）
export interface EnhancedFilterState {
  selectedDomain: string;
  selectedSecondDomain: string;
  selectedTags: string[];
  availableTags: string[];
  availableSecondDomains: string[];
  timeFilter: TimeFilter;
  userLevel: UserLevel;
}

// API响应类型（Airtable集成）
export interface AirtableIntelligenceRecord {
  id: string;
  fields: {
    // 核心字段
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
    User_Level?: string; // 'newcomer' | 'veteran' | 'both'
    Positioning_Context?: string;
    Quality_Score?: number;
    Featured_Date?: string;
    Community_Notes?: string; // JSON字符串数组
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

```typescript
export class IntelligenceService {
  private baseId: string;
  private accessToken: string;

  constructor() {
    this.baseId = process.env.AIRTABLE_BASE_ID || '';
    this.accessToken = process.env.AIRTABLE_ACCESS_TOKEN || '';
  }

  // 核心数据获取方法
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

  // 根据用户级别获取情报
  async getUserLevelIntelligence(userLevel: UserLevel): Promise<IntelligenceItem[]> {
    const filter = userLevel === 'both' 
      ? `OR({User_Level} = 'newcomer', {User_Level} = 'veteran', {User_Level} = 'both')`
      : `OR({User_Level} = '${userLevel}', {User_Level} = 'both')`;

    const data = await this.getIntelligence({ filter });
    return data.records.map(record => this.transformAirtableToIntelligenceItem(record));
  }

  // 投票功能
  async voteIntelligence(id: string, voteType: 'up' | 'down'): Promise<VoteResult> {
    try {
      // 获取当前记录
      const record = await this.getIntelligenceById(id);
      if (!record) {
        return { success: false, error: '记录未找到' };
      }

      const currentVotes = record.fields.VoteCount || 0;
      const newVoteCount = voteType === 'up' ? currentVotes + 1 : Math.max(0, currentVotes - 1);

      // 更新记录
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

  // 创建新情报
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

  // 数据转换方法
  private transformAirtableToIntelligenceItem(record: AirtableIntelligenceRecord): IntelligenceItem {
    const fields = record.fields;
    
    // 解析JSON字段
    const parseJsonField = (field: string | undefined, fallback: string[] = []): string[] => {
      if (!field) return fallback;
      try {
        return JSON.parse(field);
      } catch {
        return fallback;
      }
    };

    // 解析标签
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

    // 标准化层级格式
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

  // 获取单条记录
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

  // 辅助方法
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