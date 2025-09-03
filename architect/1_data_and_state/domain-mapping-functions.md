# Domain 映射函数文档

## 概述
本文档定义了个人能力提升三级分类体系的映射关系和相关的工具函数。

---

## 🏗️ 个人能力提升三级架构

### 类型定义

```typescript
// Domain 类型（个人能力提升大类）
type DomainLevel = 
  | 'AI-Powered Professional Skills'     // AI赋能的专业技能
  | 'Entrepreneurial Skills'            // 创业者核心能力
  | 'Product & Design Skills';          // 产品与设计能力

// Second_Domain 类型（核心能力模块）
type SecondDomainLevel = 
  // AI-Powered Professional Skills
  | 'AI Programming' 
  | 'AI Workflow Automation' 
  | 'AI Content & Creation'
  // Entrepreneurial Skills
  | 'Founder & Fundraising' 
  | 'Growth & Monetization'
  // Product & Design Skills
  | 'AI Product Management' 
  | 'AI-native UX/UI';
```

---

## 📊 映射关系表

### Domain 到 Second_Domain 的映射

```typescript
const DOMAIN_TO_SECOND_DOMAIN_MAP: Record<DomainLevel, SecondDomainLevel[]> = {
  'AI-Powered Professional Skills': [
    'AI Programming', 
    'AI Workflow Automation', 
    'AI Content & Creation'
  ],
  'Entrepreneurial Skills': [
    'Founder & Fundraising', 
    'Growth & Monetization'
  ],
  'Product & Design Skills': [
    'AI Product Management', 
    'AI-native UX/UI'
  ]
};
```

### Second_Domain 到 Tags 的映射

```typescript
const SECOND_DOMAIN_TO_TAGS_MAP: Record<SecondDomainLevel, string[]> = {
  // AI Programming
  'AI Programming': [
    'Code Generation', 
    'AI-assisted Debugging', 
    'Automated Testing', 
    'Multi-agent Development', 
    'Prompt-driven Development'
  ],
  
  // AI Workflow Automation
  'AI Workflow Automation': [
    'Personal Automation', 
    'No-code Development', 
    'API Integration', 
    'Agentic Task Management', 
    'Workflow Optimization'
  ],
  
  // AI Content & Creation
  'AI Content & Creation': [
    'Prompt Engineering', 
    'AI-assisted Writing', 
    'AI-assisted Design', 
    'AI-powered Research', 
    'Knowledge Management'
  ],
  
  // Founder & Fundraising
  'Founder & Fundraising': [
    'Fundraising Strategy', 
    'Pitch Deck Crafting', 
    'Valuation', 
    'Term Sheet Negotiation', 
    'Investor Relations'
  ],
  
  // Growth & Monetization
  'Growth & Monetization': [
    'Growth Hacking', 
    'User Acquisition', 
    'Monetization Models', 
    'SaaS Metrics', 
    'Community Building'
  ],
  
  // AI Product Management
  'AI Product Management': [
    'AI Product Strategy', 
    'AI-native Roadmapping', 
    'Data-driven Decisions', 
    'Agile for AI', 
    'Feature Prioritization'
  ],
  
  // AI-native UX/UI
  'AI-native UX/UI': [
    'Design for AI', 
    'Prototyping AI Interfaces', 
    'Human-AI Interaction', 
    'Conversational UI', 
    'Ethical AI Design'
  ]
};
```

---

## 🔧 工具函数

### 1. 验证函数

```typescript
/**
 * 验证 Domain 值是否有效
 */
export const isValidDomain = (domain: string): domain is DomainLevel => {
  return DOMAIN_LEVELS.includes(domain as DomainLevel);
};

/**
 * 验证 Second_Domain 值是否有效
 */
export const isValidSecondDomain = (secondDomain: string): secondDomain is SecondDomainLevel => {
  return SECOND_DOMAIN_LEVELS.includes(secondDomain as SecondDomainLevel);
};

/**
 * 验证 Tag 值是否有效
 */
export const isValidTag = (tag: string): boolean => {
  return STANDARD_TAG_OPTIONS.includes(tag);
};
```

### 2. 获取函数

```typescript
/**
 * 根据 Domain 获取所有相关的 Second_Domain
 */
export const getSecondDomainsByDomain = (domain: DomainLevel): SecondDomainLevel[] => {
  return DOMAIN_TO_SECOND_DOMAIN_MAP[domain] || [];
};

/**
 * 根据 Second_Domain 获取所有相关的 Tags
 */
export const getTagsBySecondDomain = (secondDomain: SecondDomainLevel): string[] => {
  return SECOND_DOMAIN_TO_TAGS_MAP[secondDomain] || [];
};

/**
 * 根据 Domain 获取所有相关的 Tags
 */
export const getAllTagsByDomain = (domain: DomainLevel): string[] => {
  const secondDomains = getSecondDomainsByDomain(domain);
  return secondDomains.flatMap(secondDomain => getTagsBySecondDomain(secondDomain));
};
```

### 3. 关系查询函数

```typescript
/**
 * 检查 Second_Domain 是否属于指定的 Domain
 */
export const isSecondDomainInDomain = (
  secondDomain: SecondDomainLevel, 
  domain: DomainLevel
): boolean => {
  return DOMAIN_TO_SECOND_DOMAIN_MAP[domain].includes(secondDomain);
};

/**
 * 检查 Tag 是否属于指定的 Second_Domain
 */
export const isTagInSecondDomain = (
  tag: string, 
  secondDomain: SecondDomainLevel
): boolean => {
  return SECOND_DOMAIN_TO_TAGS_MAP[secondDomain].includes(tag);
};

/**
 * 检查 Tag 是否属于指定的 Domain
 */
export const isTagInDomain = (tag: string, domain: DomainLevel): boolean => {
  const secondDomains = getSecondDomainsByDomain(domain);
  return secondDomains.some(secondDomain => isTagInSecondDomain(tag, secondDomain));
};
```

### 4. 反向查询函数

```typescript
/**
 * 根据 Second_Domain 查找其所属的 Domain
 */
export const getDomainBySecondDomain = (secondDomain: SecondDomainLevel): DomainLevel | null => {
  for (const [domain, secondDomains] of Object.entries(DOMAIN_TO_SECOND_DOMAIN_MAP)) {
    if (secondDomains.includes(secondDomain)) {
      return domain as DomainLevel;
    }
  }
  return null;
};

/**
 * 根据 Tag 查找其所属的 Second_Domain
 */
export const getSecondDomainByTag = (tag: string): SecondDomainLevel | null => {
  for (const [secondDomain, tags] of Object.entries(SECOND_DOMAIN_TO_TAGS_MAP)) {
    if (tags.includes(tag)) {
      return secondDomain as SecondDomainLevel;
    }
  }
  return null;
};

/**
 * 根据 Tag 查找其所属的 Domain
 */
export const getDomainByTag = (tag: string): DomainLevel | null => {
  const secondDomain = getSecondDomainByTag(tag);
  return secondDomain ? getDomainBySecondDomain(secondDomain) : null;
};
```

### 5. 分组函数

```typescript
/**
 * 将 Tags 按 Second_Domain 分组
 */
export const groupTagsBySecondDomain = (tags: string[]): Record<SecondDomainLevel, string[]> => {
  const result: Record<SecondDomainLevel, string[]> = {} as Record<SecondDomainLevel, string[]>;
  
  // 初始化所有 Second_Domain
  SECOND_DOMAIN_LEVELS.forEach(secondDomain => {
    result[secondDomain] = [];
  });
  
  // 分配 Tags
  tags.forEach(tag => {
    const secondDomain = getSecondDomainByTag(tag);
    if (secondDomain) {
      result[secondDomain].push(tag);
    }
  });
  
  return result;
};

/**
 * 将 Second_Domains 按 Domain 分组
 */
export const groupSecondDomainsByDomain = (secondDomains: SecondDomainLevel[]): Record<DomainLevel, SecondDomainLevel[]> => {
  const result: Record<DomainLevel, SecondDomainLevel[]> = {} as Record<DomainLevel, SecondDomainLevel[]>;
  
  // 初始化所有 Domain
  DOMAIN_LEVELS.forEach(domain => {
    result[domain] = [];
  });
  
  // 分配 Second_Domains
  secondDomains.forEach(secondDomain => {
    const domain = getDomainBySecondDomain(secondDomain);
    if (domain) {
      result[domain].push(secondDomain);
    }
  });
  
  return result;
};
```

### 6. 统计函数

```typescript
/**
 * 统计每个 Domain 的 Tag 数量
 */
export const countTagsByDomain = (domain: DomainLevel): number => {
  return getAllTagsByDomain(domain).length;
};

/**
 * 统计每个 Second_Domain 的 Tag 数量
 */
export const countTagsBySecondDomain = (secondDomain: SecondDomainLevel): number => {
  return getTagsBySecondDomain(secondDomain).length;
};

/**
 * 获取所有 Domain 的统计信息
 */
export const getDomainStatistics = (): Array<{
  domain: DomainLevel;
  secondDomainCount: number;
  tagCount: number;
}> => {
  return DOMAIN_LEVELS.map(domain => ({
    domain,
    secondDomainCount: getSecondDomainsByDomain(domain).length,
    tagCount: countTagsByDomain(domain)
  }));
};
```

---

## 📝 使用示例

### 基础查询

```typescript
// 获取 AI Programming 的所有标签
const aiProgrammingTags = getTagsBySecondDomain('AI Programming');
// 返回: ['Code Generation', 'AI-assisted Debugging', ...]

// 检查标签归属
const isInDomain = isTagInDomain('Prompt Engineering', 'AI-Powered Professional Skills');
// 返回: true

// 获取标签的完整路径
const tagPath = {
  tag: 'Prompt Engineering',
  secondDomain: getSecondDomainByTag('Prompt Engineering'), // 'AI Content & Creation'
  domain: getDomainByTag('Prompt Engineering') // 'AI-Powered Professional Skills'
};
```

### 高级查询

```typescript
// 分组统计
const stats = getDomainStatistics();
// 返回: [{ domain: 'AI-Powered Professional Skills', secondDomainCount: 3, tagCount: 15 }, ...]

// 分组显示
const groupedTags = groupTagsBySecondDomain(['Code Generation', 'Prompt Engineering', 'Fundraising Strategy']);
// 返回: { 'AI Programming': ['Code Generation'], 'AI Content & Creation': ['Prompt Engineering'], ... }
```

---

## 🔄 数据完整性检查

```typescript
/**
 * 检查映射关系的完整性
 */
export const validateMappingIntegrity = (): boolean => {
  // 检查所有 Second_Domain 都有对应的 Domain
  const allSecondDomainsHaveDomain = SECOND_DOMAIN_LEVELS.every(secondDomain => 
    getDomainBySecondDomain(secondDomain) !== null
  );
  
  // 检查所有 Tag 都有对应的 Second_Domain
  const allTagsHaveSecondDomain = STANDARD_TAG_OPTIONS.every(tag => 
    getSecondDomainByTag(tag) !== null
  );
  
  // 检查所有 Tag 都有对应的 Domain
  const allTagsHaveDomain = STANDARD_TAG_OPTIONS.every(tag => 
    getDomainByTag(tag) !== null
  );
  
  return allSecondDomainsHaveDomain && allTagsHaveSecondDomain && allTagsHaveDomain;
};
```

---

## 📊 常量定义

```typescript
// 所有 Domain 级别
export const DOMAIN_LEVELS: DomainLevel[] = [
  'AI-Powered Professional Skills',
  'Entrepreneurial Skills',
  'Product & Design Skills'
];

// 所有 Second_Domain 级别
export const SECOND_DOMAIN_LEVELS: SecondDomainLevel[] = [
  'AI Programming',
  'AI Workflow Automation',
  'AI Content & Creation',
  'Founder & Fundraising',
  'Growth & Monetization',
  'AI Product Management',
  'AI-native UX/UI'
];

// 所有标准标签选项
export const STANDARD_TAG_OPTIONS = [
  // AI-Powered Professional Skills (15个)
  'Code Generation', 'AI-assisted Debugging', 'Automated Testing', 'Multi-agent Development', 'Prompt-driven Development',
  'Personal Automation', 'No-code Development', 'API Integration', 'Agentic Task Management', 'Workflow Optimization',
  'Prompt Engineering', 'AI-assisted Writing', 'AI-assisted Design', 'AI-powered Research', 'Knowledge Management',
  
  // Entrepreneurial Skills (10个)
  'Fundraising Strategy', 'Pitch Deck Crafting', 'Valuation', 'Term Sheet Negotiation', 'Investor Relations',
  'Growth Hacking', 'User Acquisition', 'Monetization Models', 'SaaS Metrics', 'Community Building',
  
  // Product & Design Skills (10个)
  'AI Product Strategy', 'AI-native Roadmapping', 'Data-driven Decisions', 'Agile for AI', 'Feature Prioritization',
  'Design for AI', 'Prototyping AI Interfaces', 'Human-AI Interaction', 'Conversational UI', 'Ethical AI Design'
];
```

---

## 🎯 设计原则

1. **层次化结构**: Domain → Second_Domain → Tags 形成清晰的三级分类
2. **唯一归属**: 每个 Tag 只属于一个 Second_Domain，每个 Second_Domain 只属于一个 Domain
3. **全面覆盖**: 35个标签覆盖了个人能力提升的主要方面
4. **易于扩展**: 映射关系设计支持未来添加新的分类和标签
5. **类型安全**: 使用 TypeScript 确保编译时类型检查