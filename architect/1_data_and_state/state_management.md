# 前端状态管理 (Frontend State Management)

## 概述
定义Alpha Seeker前端应用的状态管理架构，包括全局状态、本地状态和状态更新机制。

## 架构目标
- **数据一致性**：确保多组件间数据同步
- **性能优化**：避免不必要的组件重渲染
- **开发效率**：简化状态操作和调试流程
- **可维护性**：清晰的状态结构和更新逻辑

## 技术选型
使用 **React Context API** 作为状态管理方案，具有以下优势：
- React 内置，无需额外依赖
- 简洁的 API，易于理解和使用
- 适合中小型应用的状态管理需求

### 全局状态结构

```typescript
// src/lib/auth-context.tsx - 认证状态管理
'use client';

import React, { createContext, useContext, useReducer, useEffect } from 'react';
import { User } from '@/lib/user-types';

// 认证状态类型定义
interface AuthState {
  user: User | null;
  isLoggedIn: boolean;
  isLoading: boolean;
  error: string | null;
}

// 认证动作类型
type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_FAILURE'; payload: string }
  | { type: 'LOGOUT' }
  | { type: 'CLEAR_ERROR' }
  | { type: 'SET_LOADING'; payload: boolean };

// 初始状态
const initialState: AuthState = {
  user: null,
  isLoggedIn: false,
  isLoading: false,
  error: null,
};

// 认证 Reducer
function authReducer(state: AuthState, action: AuthAction): AuthState {
  switch (action.type) {
    case 'LOGIN_START':
      return {
        ...state,
        isLoading: true,
        error: null,
      };
    case 'LOGIN_SUCCESS':
      return {
        ...state,
        user: action.payload,
        isLoggedIn: true,
        isLoading: false,
        error: null,
      };
    case 'LOGIN_FAILURE':
      return {
        ...state,
        user: null,
        isLoggedIn: false,
        isLoading: false,
        error: action.payload,
      };
    case 'LOGOUT':
      return {
        ...state,
        user: null,
        isLoggedIn: false,
        isLoading: false,
        error: null,
      };
    case 'CLEAR_ERROR':
      return {
        ...state,
        error: null,
      };
    case 'SET_LOADING':
      return {
        ...state,
        isLoading: action.payload,
      };
    default:
      return state;
  }
}

// 认证 Context 类型
interface AuthContextType extends AuthState {
  login: (id: string) => Promise<void>;
  logout: () => Promise<void>;
  clearError: () => void;
  isAdmin: () => boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

// 认证 Provider 组件
export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(authReducer, initialState);

  // 从localStorage恢复登录状态
  useEffect(() => {
    const savedUser = localStorage.getItem('user');
    if (savedUser) {
      try {
        const user = JSON.parse(savedUser);
        dispatch({ type: 'LOGIN_SUCCESS', payload: user });
      } catch (error) {
        console.error('Failed to parse saved user:', error);
        localStorage.removeItem('user');
      }
    }
  }, []);

  // 登录函数
  const login = async (id: string) => {
    dispatch({ type: 'LOGIN_START' });

    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ id }),
      });

      const data = await response.json();

      if (data.success) {
        localStorage.setItem('user', JSON.stringify(data.user));
        dispatch({ type: 'LOGIN_SUCCESS', payload: data.user });
      } else {
        dispatch({ type: 'LOGIN_FAILURE', payload: data.error || '登录失败' });
      }
    } catch (error) {
      console.error('Login error:', error);
      dispatch({ type: 'LOGIN_FAILURE', payload: '网络错误，请稍后重试' });
    }
  };

  // 登出函数
  const logout = async () => {
    try {
      await fetch('/api/auth/logout', { method: 'POST' });
    } catch (error) {
      console.error('Logout error:', error);
    } finally {
      localStorage.removeItem('user');
      dispatch({ type: 'LOGOUT' });
    }
  };

  const clearError = () => dispatch({ type: 'CLEAR_ERROR' });
  
  const isAdmin = () => state.user?.role === 'admin';

  const value: AuthContextType = {
    ...state,
    login,
    logout,
    clearError,
    isAdmin,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// 自定义 Hook 用于访问认证状态
export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

**重要说明**：
- 使用 useReducer 管理复杂的认证状态逻辑
- 状态自动保存到 localStorage 实现持久化
- 通过 Context 在组件树中共享状态
- 提供类型安全的 TypeScript 支持

### 主题状态管理

```typescript
// components/theme-provider.tsx - 主题状态管理
'use client'

import * as React from 'react'
import { ThemeProvider as NextThemesProvider } from 'next-themes'
import { type ThemeProviderProps } from 'next-themes/dist/types'

export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>
}

// components/theme-toggle.tsx - 主题切换组件
'use client'

import * as React from 'react'
import { Moon, Sun } from 'lucide-react'
import { useTheme } from 'next-themes'
import { Button } from '@/components/ui/button'

export function ThemeToggle() {
  const { theme, setTheme } = useTheme()
  const [mounted, setMounted] = React.useState(false)

  React.useEffect(() => {
    setMounted(true)
  }, [])

  if (!mounted) {
    return null
  }

  return (
    <Button
      variant="outline"
      size="icon"
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
    >
      <Sun className="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
      <span className="sr-only">切换主题</span>
    </Button>
  )
}
```

### 情报数据状态管理

```typescript
// hooks/use-intelligence.ts - 情报数据管理Hook
import { useState, useEffect } from 'react'
import { IntelligenceItem, UserLevel } from '@/lib/types'
import { intelligenceService } from '@/lib/airtable'

export function useIntelligence(userLevel: UserLevel = 'both') {
  const [intelligence, setIntelligence] = useState<IntelligenceItem[]>([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    const fetchIntelligence = async () => {
      try {
        setLoading(true)
        setError(null)
        const data = await intelligenceService.getUserLevelIntelligence(userLevel)
        setIntelligence(data)
      } catch (err) {
        setError(err instanceof Error ? err.message : '获取数据失败')
      } finally {
        setLoading(false)
      }
    }

    fetchIntelligence()
  }, [userLevel])

  const refresh = () => {
    setLoading(true)
    setError(null)
    intelligenceService.getUserLevelIntelligence(userLevel)
      .then(setIntelligence)
      .catch(err => setError(err instanceof Error ? err.message : '获取数据失败'))
      .finally(() => setLoading(false))
  }

  return { intelligence, loading, error, refresh }
}
```

### 过滤状态管理

```typescript
// hooks/use-filters.ts - 过滤状态管理
import { useState, useCallback } from 'react'
import { UserLevel, DomainLevel, SecondDomainLevel } from '@/lib/types'

export interface FilterState {
  selectedDomain: string
  selectedSecondDomain: string
  selectedTags: string[]
  timeFilter: 'today' | 'week' | 'month' | 'sixMonths' | 'all'
  userLevel: UserLevel
}

export function useFilters() {
  const [filters, setFilters] = useState<FilterState>({
    selectedDomain: 'all',
    selectedSecondDomain: 'all',
    selectedTags: [],
    timeFilter: 'all',
    userLevel: 'both'
  })

  const updateFilter = useCallback(<K extends keyof FilterState>(
    key: K,
    value: FilterState[K]
  ) => {
    setFilters(prev => ({ ...prev, [key]: value }))
  }, [])

  const resetFilters = useCallback(() => {
    setFilters({
      selectedDomain: 'all',
      selectedSecondDomain: 'all',
      selectedTags: [],
      timeFilter: 'all',
      userLevel: 'both'
    })
  }, [])

  const addTag = useCallback((tag: string) => {
    setFilters(prev => ({
      ...prev,
      selectedTags: prev.selectedTags.includes(tag)
        ? prev.selectedTags.filter(t => t !== tag)
        : [...prev.selectedTags, tag]
    }))
  }, [])

  const removeTag = useCallback((tag: string) => {
    setFilters(prev => ({
      ...prev,
      selectedTags: prev.selectedTags.filter(t => t !== tag)
    }))
  }, [])

  return {
    filters,
    updateFilter,
    resetFilters,
    addTag,
    removeTag
  }
}
```

## 在组件中使用状态

### 基本用法示例

```typescript
// components/Header.tsx - 头部组件
'use client'

import { useAuth } from '@/lib/auth-context'
import { Button } from '@/components/ui/button'

export function Header() {
  // 获取认证状态
  const { user, login, logout, isLoading, error, isAdmin } = useAuth()
  
  const handleLogin = () => {
    const userId = prompt('请输入用户ID:')
    if (userId) {
      login(userId)
    }
  }
  
  const handleLogout = () => {
    logout()
  }
  
  if (isLoading) {
    return <header>加载中...</header>
  }
  
  return (
    <header className="flex justify-between items-center p-4 border-b">
      <h1 className="text-xl font-bold">Alpha Seeker</h1>
      
      <div className="flex items-center gap-4">
        {user ? (
          <>
            <span>欢迎，{user.displayName}</span>
            {isAdmin() && <span className="text-sm bg-blue-100 px-2 py-1 rounded">管理员</span>}
            <Button onClick={handleLogout} variant="outline">
              退出
            </Button>
          </>
        ) : (
          <Button onClick={handleLogin}>
            登录
          </Button>
        )}
      </div>
      
      {error && (
        <div className="text-red-500 text-sm">
          {error}
          <button onClick={() => clearError()} className="ml-2">×</button>
        </div>
      )}
    </header>
  )
}
```

### 情报列表组件示例

```typescript
// components/IntelligenceList.tsx - 情报列表组件
'use client'

import { useIntelligence } from '@/hooks/use-intelligence'
import { useFilters } from '@/hooks/use-filters'
import { IntelligenceCard } from './IntelligenceCard'
import { FilterControls } from './FilterControls'

export function IntelligenceList() {
  // 获取情报数据
  const { intelligence, loading, error, refresh } = useIntelligence()
  
  // 获取过滤状态
  const { filters } = useFilters()
  
  // 应用过滤逻辑
  const filteredIntelligence = intelligence.filter(item => {
    // Domain过滤
    if (filters.selectedDomain !== 'all' && item.domain !== filters.selectedDomain) {
      return false
    }
    
    // Second Domain过滤
    if (filters.selectedSecondDomain !== 'all' && item.second_domain !== filters.selectedSecondDomain) {
      return false
    }
    
    // Tag过滤
    if (filters.selectedTags.length > 0) {
      const hasMatchingTag = filters.selectedTags.some(tag => item.tags.includes(tag))
      if (!hasMatchingTag) return false
    }
    
    // User Level过滤
    if (filters.userLevel !== 'both' && item.user_level !== filters.userLevel) {
      return false
    }
    
    return true
  })
  
  if (loading) {
    return <div className="text-center py-8">加载情报中...</div>
  }
  
  if (error) {
    return (
      <div className="text-center py-8">
        <p className="text-red-500 mb-4">{error}</p>
        <button onClick={refresh} className="text-blue-500 hover:underline">
          重试
        </button>
      </div>
    )
  }
  
  return (
    <div className="space-y-6">
      <FilterControls />
      
      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {filteredIntelligence.map(item => (
          <IntelligenceCard key={item.id} item={item} />
        ))}
      </div>
      
      {filteredIntelligence.length === 0 && (
        <div className="text-center py-8 text-gray-500">
          没有找到符合条件的情报
        </div>
      )}
    </div>
  )
}
```

### 情报卡片组件示例

```typescript
// components/IntelligenceCard.tsx - 情报卡片组件
'use client'

import { useState } from 'react'
import { IntelligenceItem } from '@/lib/types'
import { useAuth } from '@/lib/auth-context'
import { Button } from '@/components/ui/button'

interface IntelligenceCardProps {
  item: IntelligenceItem
}

export function IntelligenceCard({ item }: IntelligenceCardProps) {
  const { state } = useAuth()
  const [voteCount, setVoteCount] = useState(item.vote_count)
  const [hasVoted, setHasVoted] = useState(false)
  
  const handleVote = async () => {
    if (!state.user) {
      alert('请先登录后再投票')
      return
    }
    
    if (hasVoted) {
      alert('您已经投过票了')
      return
    }
    
    try {
      const response = await fetch('/api/vote', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          intelligenceId: item.id,
          voteType: 'up'
        })
      })
      
      const result = await response.json()
      if (result.success) {
        setVoteCount(result.data.newVoteCount)
        setHasVoted(true)
      }
    } catch (error) {
      alert('投票失败，请稍后重试')
    }
  }
  
  return (
    <div className="border rounded-lg p-6 space-y-4 hover:shadow-lg transition-shadow">
      {/* Tier标签 */}
      <div className="flex justify-between items-start">
        <span className={`px-2 py-1 text-xs rounded ${
          item.tier.includes('1') ? 'bg-red-100 text-red-800' :
          item.tier.includes('2') ? 'bg-yellow-100 text-yellow-800' :
          'bg-green-100 text-green-800'
        }`}>
          {item.tier}
        </span>
        
        <span className="text-sm text-gray-500">
          {item.user_level === 'both' ? '全部用户' : item.user_level === 'newcomer' ? '新手' : '老手'}
        </span>
      </div>
      
      {/* 标题和洞察 */}
      <div>
        <h3 className="font-semibold text-lg mb-2">{item.title}</h3>
        <p className="text-gray-600 text-sm">{item.insight}</p>
      </div>
      
      {/* 分类信息 */}
      <div className="flex flex-wrap gap-2">
        <span className="text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded">
          {item.domain}
        </span>
        <span className="text-xs bg-purple-100 text-purple-800 px-2 py-1 rounded">
          {item.second_domain}
        </span>
        {item.tags.map(tag => (
          <span key={tag} className="text-xs bg-gray-100 text-gray-800 px-2 py-1 rounded">
            {tag}
          </span>
        ))}
      </div>
      
      {/* 来源信息 */}
      <div className="text-sm text-gray-500">
        <span>作者：{item.author}</span>
        {item.source_link && (
          <a 
            href={item.source_link} 
            target="_blank" 
            rel="noopener noreferrer"
            className="ml-2 text-blue-500 hover:underline"
          >
            查看原文
          </a>
        )}
      </div>
      
      {/* 投票区域 */}
      <div className="flex justify-between items-center pt-4 border-t">
        <Button
          onClick={handleVote}
          disabled={hasVoted || !state.user}
          variant="outline"
          size="sm"
        >
          👍 {voteCount}
        </Button>
        
        <span className="text-xs text-gray-400">
          {new Date(item.published_at).toLocaleDateString()}
        </span>
      </div>
    </div>
  )
}
```

## 状态更新最佳实践

### ✅ 好的做法

```typescript
// 1. 使用 dispatch 更新状态
const handleLogin = (userData: User) => {
  dispatch({ type: 'LOGIN_SUCCESS', payload: { user: userData, isAdmin: false } })
}

// 2. 异步操作的错误处理
const fetchIntelligence = async () => {
  setLoading(true)
  setError(null)
  try {
    const data = await intelligenceService.getUserLevelIntelligence('both')
    setIntelligence(data)
  } catch (err) {
    setError(err instanceof Error ? err.message : '获取数据失败')
  } finally {
    setLoading(false)
  }
}

// 3. 使用 useCallback 优化回调函数
const handleVote = useCallback(async (intelligenceId: string) => {
  if (!state.user) return
  
  try {
    const response = await fetch('/api/vote', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ intelligenceId, voteType: 'up' })
    })
    
    const result = await response.json()
    if (result.success) {
      // 更新本地状态
      setIntelligence(prev => prev.map(item => 
        item.id === intelligenceId 
          ? { ...item, vote_count: result.data.newVoteCount }
          : item
      ))
    }
  } catch (error) {
    console.error('投票失败:', error)
  }
}, [state.user])
```

### ❌ 避免的做法

```typescript
// 1. 直接修改状态对象（会导致组件不更新）
const badUpdate = () => {
  intelligence[0].vote_count += 1  // ❌ 直接修改，不会触发重新渲染
}

// 2. 在 useEffect 中缺少依赖数组
function BadComponent() {
  const [data, setData] = useState([])
  
  useEffect(() => {
    fetchData()  // ❌ 每次渲染都会执行
  })  // 缺少依赖数组
}

// 3. 忘记处理加载和错误状态
const badFetch = async () => {
  const data = await api.getData()  // ❌ 没有错误处理
  setData(data)
}
```

## 调试技巧

### 使用 React DevTools
- 安装 React Developer Tools 浏览器扩展
- 使用 Components 标签页检查组件状态和 props
- 使用 Profiler 标签页分析组件渲染性能

### 添加控制台日志
```typescript
// 在 key 操作中添加日志
export function useAuth() {
  const { state, dispatch } = useContext(AuthContext)
  
  const login = useCallback((userData: User) => {
    console.log('用户登录:', userData)  // 调试日志
    dispatch({ type: 'LOGIN_SUCCESS', payload: { user: userData, isAdmin: false } })
  }, [dispatch])
  
  const logout = useCallback(() => {
    console.log('用户登出')  // 调试日志
    dispatch({ type: 'LOGOUT' })
  }, [dispatch])
  
  return { state, login, logout }
}
```

### 使用 TypeScript 类型检查
```typescript
// 确保所有状态都有明确的类型定义
interface AppState {
  intelligence: IntelligenceItem[]
  filters: FilterState
  loading: boolean
  error: string | null
}

// 使用类型守卫确保类型安全
const isError = (error: unknown): error is Error => {
  return error instanceof Error
}
```

## 性能优化

### 使用 React.memo 优化组件渲染
```typescript
import React, { memo } from 'react'

interface IntelligenceCardProps {
  item: IntelligenceItem
  onVote: (id: string) => void
}

export const IntelligenceCard = memo(function IntelligenceCard({ 
  item, 
  onVote 
}: IntelligenceCardProps) {
  return (
    <div className="border rounded-lg p-4">
      {/* 卡片内容 */}
      <button onClick={() => onVote(item.id)}>
        投票
      </button>
    </div>
  )
})
```

### 使用 useMemo 和 useCallback
```typescript
function IntelligenceList({ intelligence }: { intelligence: IntelligenceItem[] }) {
  const { filters } = useFilters()
  
  // 使用 useMemo 缓存计算结果
  const filteredIntelligence = useMemo(() => {
    return intelligence.filter(item => {
      // 复杂的过滤逻辑
      return item.domain === filters.selectedDomain
    })
  }, [intelligence, filters.selectedDomain])
  
  // 使用 useCallback 缓存回调函数
  const handleVote = useCallback((id: string) => {
    // 投票逻辑
  }, [])
  
  return (
    <div>
      {filteredIntelligence.map(item => (
        <IntelligenceCard key={item.id} item={item} onVote={handleVote} />
      ))}
    </div>
  )
}
```

### 避免不必要的重新渲染
```typescript
// ✅ 好的做法：将状态提升到合适的层级
function ParentComponent() {
  const [filters, setFilters] = useState(defaultFilters)
  
  return (
    <div>
      <FilterControls filters={filters} onFilterChange={setFilters} />
      <IntelligenceList filters={filters} />
    </div>
  )
}

// ❌ 避免的做法：在深层组件中管理共享状态
function DeepChildComponent() {
  const [filters, setFilters] = useState(defaultFilters)  // ❌ 状态应该在更上层管理
  // ...
}
```

## 修改指南

### ✅ 安全的修改
- 添加新的 Context Provider
- 扩展现有状态类型定义
- 优化组件性能（memo、useMemo、useCallback）
- 添加新的自定义 Hooks

### ⚠️ 需要谨慎的修改
- 修改状态接口（确保所有使用的地方都更新）
- 删除状态字段（确保没有组件在使用）
- 更新 Context 值结构（可能影响多个组件）
- 修改 reducer 逻辑（确保所有 action 都正确处理）

### 🚨 危险操作
- 删除 Context Provider（确保没有组件依赖）
- 修改状态持久化逻辑（可能导致数据丢失）
- 重构状态结构（需要全面测试）

## 总结

Alpha Seeker 采用 React Context API 作为状态管理方案，具有以下特点：

1. **简洁性**：使用 React 内置功能，无需额外依赖
2. **类型安全**：完整的 TypeScript 类型定义
3. **模块化**：不同功能模块使用独立的 Context 和 Hooks
4. **性能优化**：通过 memo、useMemo、useCallback 避免不必要的重新渲染
5. **可维护性**：清晰的状态结构和更新逻辑

这种架构适合中小型应用，为 Alpha Seeker 提供了高效、可维护的状态管理解决方案。
