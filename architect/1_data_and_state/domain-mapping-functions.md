# Domain 映射函数文档

## 概述
本文档定义了个人能力提升三级分类体系的映射关系和相关的工具函数。

---

## 🏗️ 个人能力提升三级架构

### 类型定义

```typescript
// Domain 类型 - 个人能力提升的三个主要方向（最高层级）
// 这个类型定义了用户需要提升的三个核心能力领域
type DomainLevel = 
  | 'AI-Powered Professional Skills'     // AI赋能的专业技能 - 利用AI技术提升专业工作效率
                                         // 涵盖编程、自动化、内容创作等AI应用场景
  | 'Entrepreneurial Skills'            // 创业者核心能力 - 创业和商业化所需的技能
                                         // 包括融资、增长、商业模式等创业必备能力
  | 'Product & Design Skills';          // 产品与设计能力 - AI时代的产品思维和设计能力
                                         // 聚焦AI产品设计、用户体验和决策能力

// Second_Domain 类型 - 每个Domain下的细分能力模块（中间层级）
// 这个类型定义了7个具体的能力模块，每个模块都对应着一个专业技能领域
type SecondDomainLevel = 
  // AI-Powered Professional Skills 下属的三个技能模块
  | 'AI Programming'                     // AI编程 - 使用AI工具进行软件开发
                                         // 包括代码生成、调试、测试等开发全流程
  | 'AI Workflow Automation'             // AI工作流自动化 - 构建智能化的工作流程
                                         // 涵盖个人自动化、无代码开发、流程优化等
  | 'AI Content & Creation'             // AI内容创作 - 利用AI进行创意内容生产
                                         // 包括提示工程、写作、设计、研究等创作活动
  
  // Entrepreneurial Skills 下属的两个技能模块
  | 'Founder & Fundraising'             // 创始人与融资 - 创业初期的核心能力
                                         // 涵盖融资策略、商业计划书、估值谈判等
  | 'Growth & Monetization'             // 增长与变现 - 企业的持续发展能力
                                         // 包括增长黑客、用户获取、商业模式等
  
  // Product & Design Skills 下属的两个技能模块
  | 'AI Product Management'             // AI产品管理 - AI时代的产品管理能力
                                         // 涵盖AI产品策略、路线规划、数据决策等
  | 'AI-native UX/UI';                  // AI原生用户体验 - 针对AI产品的设计能力
                                         // 包括AI界面设计、人机交互、伦理设计等
```

---

## 📊 映射关系表

### Domain 到 Second_Domain 的映射

```typescript
// Domain 到 Second_Domain 的映射关系表
// 这个映射表定义了每个主要能力领域包含哪些具体的技能模块
// 确保每个 Second_Domain 都能找到对应的上级 Domain
const DOMAIN_TO_SECOND_DOMAIN_MAP: Record<DomainLevel, SecondDomainLevel[]> = {
  'AI-Powered Professional Skills': [    // AI赋能的专业技能 - 包含3个核心技能模块
    'AI Programming',                     // AI编程：学习使用AI工具进行高效开发
    'AI Workflow Automation',             // AI工作流自动化：构建智能化的工作流程
    'AI Content & Creation'              // AI内容创作：利用AI进行创意内容生产
  ],
  'Entrepreneurial Skills': [            // 创业者核心能力 - 包含2个商业技能模块
    'Founder & Fundraising',             // 创始人与融资：学习创业初期必备的融资技能
    'Growth & Monetization'             // 增长与变现：掌握企业持续发展的核心能力
  ],
  'Product & Design Skills': [          // 产品与设计能力 - 包含2个产品设计技能模块
    'AI Product Management',             // AI产品管理：学习AI时代的产品管理方法
    'AI-native UX/UI'                   // AI原生用户体验：设计针对AI产品的用户界面
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
 * 验证传入的字符串是否为有效的Domain值
 * 这个函数确保传入的domain值在预定义的三个主要能力领域范围内
 * 使用TypeScript的类型保护，返回布尔值同时进行类型收窄
 */
export const isValidDomain = (domain: string): domain is DomainLevel => {
  // 检查传入的domain字符串是否存在于预定义的DOMAIN_LEVELS数组中
  // 如果存在，返回true，同时TypeScript会知道这个字符串是DomainLevel类型
  return DOMAIN_LEVELS.includes(domain as DomainLevel);
};

/**
 * 验证传入的字符串是否为有效的Second_Domain值
 * 这个函数确保传入的secondDomain值在预定义的7个技能模块范围内
 * 使用TypeScript的类型保护，返回布尔值同时进行类型收窄
 */
export const isValidSecondDomain = (secondDomain: string): secondDomain is SecondDomainLevel => {
  // 检查传入的secondDomain字符串是否存在于预定义的SECOND_DOMAIN_LEVELS数组中
  // 如果存在，返回true，同时TypeScript会知道这个字符串是SecondDomainLevel类型
  return SECOND_DOMAIN_LEVELS.includes(secondDomain as SecondDomainLevel);
};

/**
 * 验证传入的字符串是否为有效的Tag值
 * 这个函数确保传入的tag值在预定义的35个标准标签范围内
 * 由于标签是字符串类型，不需要类型保护，直接返回布尔值
 */
export const isValidTag = (tag: string): boolean => {
  // 检查传入的tag字符串是否存在于预定义的STANDARD_TAG_OPTIONS数组中
  // 这个数组包含了所有35个标准技能标签，确保标签的标准化和一致性
  return STANDARD_TAG_OPTIONS.includes(tag);
};
```

### 2. 获取函数

```typescript
/**
 * 根据指定的Domain获取其下属的所有Second_Domain技能模块
 * 这个函数用于获取某个主要能力领域下的所有细分技能模块
 * 例如：传入'AI-Powered Professional Skills'，返回3个相关的技能模块
 */
export const getSecondDomainsByDomain = (domain: DomainLevel): SecondDomainLevel[] => {
  // 从映射表中查找指定Domain对应的所有Second_Domain
  // 如果找到了对应的映射关系，返回Second_Domain数组
  // 如果没找到（理论上不应该发生），返回空数组以确保函数安全
  return DOMAIN_TO_SECOND_DOMAIN_MAP[domain] || [];
};

/**
 * 根据指定的Second_Domain获取其下属的所有具体技能标签
 * 这个函数用于获取某个技能模块下的所有相关技能标签
 * 例如：传入'AI Programming'，返回5个相关的技能标签
 */
export const getTagsBySecondDomain = (secondDomain: SecondDomainLevel): string[] => {
  // 从映射表中查找指定Second_Domain对应的所有技能标签
  // 如果找到了对应的映射关系，返回标签数组
  // 如果没找到（理论上不应该发生），返回空数组以确保函数安全
  return SECOND_DOMAIN_TO_TAGS_MAP[secondDomain] || [];
};

/**
 * 根据指定的Domain获取其下属的所有技能标签（跨层级获取）
 * 这个函数会获取某个主要能力领域下的所有技能标签
 * 通过先获取Second_Domain，再获取每个Second_Domain下的标签，最后合并所有标签
 * 例如：传入'AI-Powered Professional Skills'，返回15个相关的技能标签
 */
export const getAllTagsByDomain = (domain: DomainLevel): string[] => {
  // 第一步：获取指定Domain下的所有Second_Domain技能模块
  const secondDomains = getSecondDomainsByDomain(domain);
  
  // 第二步：使用flatMap将每个Second_Domain的标签数组合并为一个数组
  // flatMap会自动处理嵌套数组的扁平化，避免手动使用concat或展开运算符
  // 最终返回该Domain下的所有技能标签的完整列表
  return secondDomains.flatMap(secondDomain => getTagsBySecondDomain(secondDomain));
};
```

### 3. 关系查询函数

```typescript
/**
 * 验证技能模块与能力领域的归属关系
 * 用于内容分类的准确性检查，确保标签体系的完整性
 */
export const isSecondDomainInDomain = (
  secondDomain: SecondDomainLevel,     // 技能模块名称
  domain: DomainLevel                 // 能力领域名称
): boolean => {
  return DOMAIN_TO_SECOND_DOMAIN_MAP[domain].includes(secondDomain);
};

/**
 * 验证技能标签与技能模块的归属关系
 * 用于标签分类的正确性验证，支持内容推荐系统的精准匹配
 */
export const isTagInSecondDomain = (
  tag: string,                         // 技能标签名称
  secondDomain: SecondDomainLevel      // 技能模块名称
): boolean => {
  return SECOND_DOMAIN_TO_TAGS_MAP[secondDomain].includes(tag);
};

/**
 * 跨层级验证标签与能力领域的归属关系
 * 用于用户内容过滤和个性化推荐，支持从标签直接映射到用户偏好
 * 性能优化：使用some方法避免完整遍历
 */
export const isTagInDomain = (tag: string, domain: DomainLevel): boolean => {
  const secondDomains = getSecondDomainsByDomain(domain);
  return secondDomains.some(secondDomain => isTagInSecondDomain(tag, secondDomain));
};
```

### 4. 反向查询函数

```typescript
/**
 * 根据技能模块反向查找所属的能力领域
 * 用于内容推荐和用户偏好分析，支持从具体技能向上追溯到能力领域
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
 * 根据技能标签反向查找所属的技能模块
 * 用于内容分类的自动化处理，支持标签的智能归类和验证
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
 * 根据技能标签反向查找所属的能力领域
 * 用于用户画像构建和内容匹配，支持从具体标签直接映射到用户能力偏好
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