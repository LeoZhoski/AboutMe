# 用户认证逻辑 (Authentication)

## 概述

Alpha Seeker 采用**简化认证架构**，基于用户ID的轻量级认证系统。作为情报聚合平台，系统需要基础的用户身份管理来支持投票、评论等社区功能，但避免了复杂的注册流程。

## 当前架构状态

### 简化认证设计的原因

1. **功能导向**：支持投票、评论、审核等核心社区功能
2. **用户体验**：简化登录流程，降低使用门槛
3. **快速迭代**：轻量级实现，便于功能开发和测试
4. **扩展性**：为未来功能扩展预留接口

### 当前认证特性

- **基于ID认证**：使用用户ID而非邮箱/密码组合
- **角色管理**：支持管理员和普通用户角色区分
- **状态持久化**：使用localStorage保存登录状态
- **API保护**：需要认证的API接口进行权限验证

### 技术实现

- **前端**：React Context API + useReducer 状态管理
- **后端**：Next.js API Routes + 简化验证
- **存储**：localStorage 客户端状态持久化
- **类型安全**：完整的 TypeScript 类型定义

## 当前认证实现

### 用户类型定义

```typescript
// src/lib/user-types.ts
export interface User {
  id: string;
  displayName: string;
  role: 'admin' | 'user';
  userLevel: 'newcomer' | 'veteran' | 'both';
  avatarURL?: string;
  email?: string;
  isActive: boolean;
  createdAt: string;
  lastLogin?: string;
  loginCount?: number;
  voteCount?: number;
  submissionCount?: number;
}

export interface LoginRequest {
  id: string;
}

export interface LoginResponse {
  success: boolean;
  user?: User;
  error?: string;
}
```

### 认证状态管理

```typescript
// src/lib/auth-context.tsx
import React, { createContext, useContext, useReducer, useEffect } from 'react';

interface AuthState {
  user: User | null;
  isLoggedIn: boolean;
  isLoading: boolean;
  error: string | null;
}

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_FAILURE'; payload: string }
  | { type: 'LOGOUT' }
  | { type: 'CLEAR_ERROR' }
  | { type: 'SET_LOADING'; payload: boolean };

interface AuthContextType extends AuthState {
  login: (id: string) => Promise<void>;
  logout: () => Promise<void>;
  clearError: () => void;
  isAdmin: () => boolean;
}

function authReducer(state: AuthState, action: AuthAction): AuthState {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true, error: null };
    case 'LOGIN_SUCCESS':
      return { 
        ...state, 
        user: action.payload, 
        isLoggedIn: true, 
        isLoading: false, 
        error: null 
      };
    case 'LOGIN_FAILURE':
      return { 
        ...state, 
        user: null, 
        isLoggedIn: false, 
        isLoading: false, 
        error: action.payload 
      };
    case 'LOGOUT':
      return { 
        ...state, 
        user: null, 
        isLoggedIn: false, 
        isLoading: false, 
        error: null 
      };
    case 'CLEAR_ERROR':
      return { ...state, error: null };
    case 'SET_LOADING':
      return { ...state, isLoading: action.payload };
    default:
      return state;
  }
}
```

### 认证提供者组件

```typescript
// src/lib/auth-context.tsx (continued)
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
```

### 自定义Hook

```typescript
// src/lib/auth-context.tsx (continued)
export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

### API端点实现

```typescript
// src/app/api/auth/login/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const { id } = await request.json();
    
    // 简化验证逻辑 - 实际应用中应该连接数据库
    const user = {
      id,
      displayName: `用户${id}`,
      role: 'user' as const,
      userLevel: 'both' as const,
      isActive: true,
      createdAt: new Date().toISOString(),
    };

    return NextResponse.json({
      success: true,
      user,
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: '登录失败',
    });
  }
}

// src/app/api/auth/logout/route.ts
export async function POST() {
  return NextResponse.json({
    success: true,
    message: '登出成功',
  });
}
```

## 使用示例

### 组件中使用认证

```typescript
// components/Header.tsx
'use client';

import { useAuth } from '@/lib/auth-context';
import { Button } from '@/components/ui/button';

export function Header() {
  const { user, login, logout, isLoading, error } = useAuth();

  const handleLogin = () => {
    const userId = prompt('请输入用户ID:');
    if (userId) {
      login(userId);
    }
  };

  if (isLoading) {
    return <header>加载中...</header>;
  }

  return (
    <header className="flex justify-between items-center p-4 border-b">
      <h1 className="text-xl font-bold">Alpha Seeker</h1>
      
      <div className="flex items-center gap-4">
        {user ? (
          <>
            <span>欢迎，{user.displayName}</span>
            {user.role === 'admin' && (
              <span className="text-sm bg-blue-100 px-2 py-1 rounded">管理员</span>
            )}
            <Button onClick={logout} variant="outline">
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
  );
}
```

### 受保护的路由

```typescript
// components/ProtectedRoute.tsx
'use client';

import { useAuth } from '@/lib/auth-context';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRole?: 'admin' | 'user';
}

export function ProtectedRoute({ children, requiredRole }: ProtectedRouteProps) {
  const { user, isLoading } = useAuth();

  if (isLoading) {
    return <div>加载中...</div>;
  }

  if (!user) {
    return <div>请先登录</div>;
  }

  if (requiredRole && user.role !== requiredRole) {
    return <div>权限不足</div>;
  }

  return <>{children}</>;
}
```

### API路由保护

```typescript
// src/lib/auth.ts
import { NextRequest } from 'next/server';

export function getUserFromRequest(request: NextRequest): User | null {
  // 在实际应用中，这里应该解析token或session
  // 当前简化版本中，我们使用header或cookie
  const authHeader = request.headers.get('authorization');
  if (!authHeader) return null;
  
  try {
    const userData = JSON.parse(authHeader.replace('Bearer ', ''));
    return userData;
  } catch {
    return null;
  }
}

export function requireAuth(request: NextRequest): User {
  const user = getUserFromRequest(request);
  if (!user) {
    throw new Error('未认证');
  }
  return user;
}

export function requireRole(request: NextRequest, role: 'admin' | 'user'): User {
  const user = requireAuth(request);
  if (user.role !== role) {
    throw new Error('权限不足');
  }
  return user;
}
```

### 在API中使用认证

```typescript
// src/app/api/vote/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { requireAuth } from '@/lib/auth';

export async function POST(request: NextRequest) {
  try {
    const user = requireAuth(request);
    const { intelligenceId, voteType } = await request.json();
    
    // 验证用户权限并处理投票逻辑
    // ...
    
    return NextResponse.json({
      success: true,
      message: '投票成功',
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error instanceof Error ? error.message : '投票失败',
    });
  }
}
```

## 数据隐私保护

### 当前隐私措施

1. **最小化数据收集**：仅收集必要的用户标识信息
2. **本地存储**：用户信息仅存储在localStorage中
3. **无追踪**：不收集用户行为数据
4. **透明处理**：所有数据处理逻辑公开透明

### 用户数据安全

```typescript
// 安全的数据处理
export const secureUserStorage = {
  save: (user: User) => {
    const userData = {
      ...user,
      // 移除敏感信息
      email: user.email ? `${user.email.substring(0, 3)}***` : undefined,
    };
    localStorage.setItem('user', JSON.stringify(userData));
  },
  
  load: (): User | null => {
    try {
      const saved = localStorage.getItem('user');
      return saved ? JSON.parse(saved) : null;
    } catch {
      return null;
    }
  },
  
  clear: () => {
    localStorage.removeItem('user');
  }
};
```

## 实施建议

### 部署步骤

1. **配置AuthProvider**：在应用根部包裹AuthProvider组件
2. **实现API端点**：创建`/api/auth/login`和`/api/auth/logout`路由
3. **更新组件**：在需要认证的组件中使用useAuth Hook
4. **添加路由保护**：为需要认证的页面添加保护逻辑
5. **测试验证**：确保登录、登出、权限验证等功能正常

### 最佳实践

1. **错误处理**：提供清晰的用户反馈和错误信息
2. **加载状态**：在认证操作期间显示适当的加载状态
3. **状态同步**：确保多个标签页间的认证状态同步
4. **安全性**：定期清理localStorage，避免敏感信息泄露

### 性能优化

```typescript
// 优化认证状态检查
export function useOptimizedAuth() {
  const context = useContext(AuthContext);
  
  // 使用useMemo缓存计算结果
  const isAuthenticated = useMemo(() => {
    return context.user !== null && context.isLoggedIn;
  }, [context.user, context.isLoggedIn]);
  
  const userPermissions = useMemo(() => {
    if (!context.user) return [];
    return context.user.role === 'admin' 
      ? ['read', 'write', 'admin'] 
      : ['read', 'write'];
  }, [context.user]);
  
  return {
    ...context,
    isAuthenticated,
    userPermissions,
  };
}
```

## 修改指南

### ✅ 安全的修改

- 扩展User接口添加新字段
- 优化认证流程用户体验
- 添加新的认证方法（如OAuth）
- 增强错误处理和验证

### ⚠️ 需要谨慎的修改

- 修改认证状态结构
- 更新localStorage存储格式
- 更改API端点路径
- 修改权限验证逻辑

### 🚨 危险操作

- 移除认证中间件
- 硬编码敏感信息
- 禁用状态持久化
- 绕过权限检查

## 当前认证流程图

```
用户输入ID → 前端验证 → 调用登录API → 服务器验证 → 返回用户信息
     ↓           ↓          ↓           ↓           ↓
  prompt输入   基本格式检查   POST请求   简化验证    localStorage存储
```

## 总结

Alpha Seeker 当前采用基于用户ID的简化认证架构，具有以下特点：

1. **简洁性**：无需复杂注册流程，用户输入ID即可登录
2. **实用性**：支持投票、评论、审核等核心社区功能
3. **扩展性**：基于React Context的架构便于未来功能扩展
4. **安全性**：最小化数据收集，本地存储保护用户隐私

该认证系统为平台的社区功能提供了必要的用户身份管理，同时保持了良好的用户体验和开发效率。随着平台发展，可以逐步扩展为更完整的认证系统。

