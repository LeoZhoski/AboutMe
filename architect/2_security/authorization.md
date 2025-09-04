# 用户授权逻辑 (Authorization)

## 概述

Alpha Seeker 采用**简化角色授权模型**，基于用户角色的轻量级权限控制系统。系统通过区分管理员和普通用户角色，为不同操作提供适当的权限管理。

## 当前授权状态

### 简化授权设计原则

1. **角色分离**：清晰区分管理员和普通用户权限
2. **功能导向**：权限设计围绕核心业务功能
3. **最小权限**：每个角色只拥有必要的权限
4. **易于维护**：简化的权限结构便于管理

### 当前角色模型

```
用户 → 角色分配 → 权限检查 → 功能访问
    ↓       ↓        ↓         ↓
   ID登录   admin/user 权限验证 受控操作
```

### 系统特点

- **双角色系统**：管理员(admin)和普通用户(user)
- **权限分层**：关键操作需要管理员权限
- **操作保护**：投票、评论等需要用户身份
- **内容审核**：情报审核需要管理员权限

## 当前授权实现

### 角色权限定义

```typescript
// 用户角色枚举：admin（管理员）和user（普通用户）
export type UserRole = 'admin' | 'user';

// 权限类型：使用资源:操作的命名格式
export type Permission = 
  | 'read:intelligence'      // 读取情报权限
  | 'vote:intelligence'      // 投票权限
  | 'create:comment'         // 创建评论权限
  | 'submit:intelligence'    // 提交情报权限
  | 'review:intelligence'    // 审核情报权限
  | 'manage:users'           // 管理用户权限
  | 'manage:system'          // 系统管理权限
  | 'delete:content';        // 删除内容权限

// 角色权限映射表：定义每个角色拥有的具体权限列表
const ROLE_PERMISSIONS: Record<UserRole, Permission[]> = {
  admin: [                    // 管理员拥有完整权限
    'read:intelligence',
    'vote:intelligence', 
    'create:comment',
    'submit:intelligence',
    'review:intelligence',
    'manage:users',
    'manage:system',
    'delete:content'
  ],
  user: [                     // 普通用户拥有基础权限
    'read:intelligence',
    'vote:intelligence',
    'create:comment', 
    'submit:intelligence'
  ]
};
```

### 权限检查服务

```typescript
// src/lib/authorization.ts
import { User } from '@/lib/user-types';
import type { Permission, UserRole } from './types';

// 授权服务类 - 提供统一的权限检查和角色验证功能
export class AuthorizationService {
  // 检查用户是否拥有特定权限，用于保护需要权限的操作
  static hasPermission(user: User | null, permission: Permission): boolean {
    if (!user) return false;                    // 用户不存在
    if (!user.isActive) return false;           // 账户未激活
    return ROLE_PERMISSIONS[user.role].includes(permission);
  }
  
  // 检查用户是否拥有特定角色，用于基于角色的访问控制
  static hasRole(user: User | null, role: UserRole): boolean {
    if (!user) return false;                    // 用户不存在
    if (!user.isActive) return false;           // 账户未激活
    return user.role === role;
  }
  
  // 检查用户是否为管理员，便捷方法
  static isAdmin(user: User | null): boolean {
    return this.hasRole(user, 'admin');
  }
  
  // 获取用户的所有权限列表，用于前端界面权限展示
  static getUserPermissions(user: User | null): Permission[] {
    if (!user || !user.isActive) return [];    // 用户无效或未激活
    return ROLE_PERMISSIONS[user.role];
  }
}
```

### React Hook 集成

```typescript
// hooks/use-authorization.ts
import { useAuth } from '@/lib/auth-context';
import { AuthorizationService, type Permission } from '@/lib/authorization';

// 授权Hook - 提供React组件中的权限检查功能
export function useAuthorization() {
  const { user } = useAuth();
  
  // 权限检查函数 - 提供语义化的权限检查接口
  const can = (permission: Permission): boolean => {
    return AuthorizationService.hasPermission(user, permission);
  };
  
  // 反向权限检查函数 - 提供语义化的权限否定检查
  const cannot = (permission: Permission): boolean => {
    return !can(permission);
  };
  
  // 管理员角色检查函数
  const isAdmin = (): boolean => {
    return AuthorizationService.isAdmin(user);
  };
  
  // 普通用户角色检查函数
  const isUser = (): boolean => {
    return AuthorizationService.hasRole(user, 'user');
  };
  
  return {
    user,                                          // 当前用户信息
    can,                                           // 权限检查函数
    cannot,                                        // 反向权限检查
    isAdmin,                                       // 管理员检查
    isUser,                                        // 普通用户检查
    permissions: AuthorizationService.getUserPermissions(user)  // 用户权限列表
  };
}
```

### 权限保护组件

```typescript
// components/AuthorizationGuard.tsx
'use client';

import { useAuthorization } from '@/hooks/use-authorization';

interface AuthorizationGuardProps {
  permission: string;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

export function AuthorizationGuard({ 
  permission, 
  children, 
  fallback 
}: AuthorizationGuardProps) {
  const { can } = useAuthorization();
  
  if (!can(permission as any)) {
    return <>{fallback || <div>权限不足</div>}</>;
  }
  
  return <>{children}</>;
}

// 管理员专用组件
interface AdminGuardProps {
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

export function AdminGuard({ children, fallback }: AdminGuardProps) {
  const { isAdmin } = useAuthorization();
  
  if (!isAdmin()) {
    return <>{fallback || <div>需要管理员权限</div>}</>;
  }
  
  return <>{children}</>;
}
```

### 使用示例

#### 组件中使用权限控制

```typescript
// components/IntelligenceCard.tsx
'use client';

import { useAuthorization } from '@/hooks/use-authorization';
import { AuthorizationGuard, AdminGuard } from '@/components/AuthorizationGuard';

export function IntelligenceCard({ item }: IntelligenceCardProps) {
  const { can, user } = useAuthorization();
  
  const handleVote = async () => {
    if (!can('vote:intelligence')) {
      alert('请先登录后再投票');
      return;
    }
    // 投票逻辑...
  };
  
  const handleReview = async () => {
    if (!can('review:intelligence')) {
      alert('需要管理员权限');
      return;
    }
    // 审核逻辑...
  };

  return (
    <div className="border rounded-lg p-6">
      {/* 情报内容 */}
      
      {/* 投票按钮 - 所有登录用户可见 */}
      <AuthorizationGuard permission="vote:intelligence" fallback={<span>登录后可投票</span>}>
        <button onClick={handleVote}>👍 投票</button>
      </AuthorizationGuard>
      
      {/* 审核按钮 - 仅管理员可见 */}
      <AdminGuard fallback={null}>
        <button onClick={handleReview} className="ml-2">审核</button>
      </AdminGuard>
      
      {/* 删除按钮 - 仅管理员可见 */}
      <AdminGuard fallback={null}>
        <button onClick={() => handleDelete(item.id)} className="ml-2 text-red-500">删除</button>
      </AdminGuard>
    </div>
  );
}
```

#### API 路由权限保护

```typescript
// src/app/api/review/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { AuthorizationService } from '@/lib/authorization';
import { requireAuth } from '@/lib/auth';

export async function POST(request: NextRequest) {
  try {
    const user = requireAuth(request);
    const { intelligenceId, decision } = await request.json();
    
    // 检查审核权限
    if (!AuthorizationService.hasPermission(user, 'review:intelligence')) {
      return NextResponse.json({
        success: false,
        error: '需要管理员权限'
      }, { status: 403 });
    }
    
    // 执行审核逻辑...
    
    return NextResponse.json({
      success: true,
      message: '审核完成'
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error instanceof Error ? error.message : '审核失败'
    });
  }
}
```

#### 条件渲染示例

```typescript
// components/AdminPanel.tsx
'use client';

import { useAuthorization } from '@/hooks/use-authorization';

export function AdminPanel() {
  const { isAdmin, permissions } = useAuthorization();
  
  if (!isAdmin()) {
    return null; // 非管理员不显示
  }
  
  return (
    <div className="admin-panel">
      <h2>管理员面板</h2>
      <div className="permissions-info">
        <h3>当前权限:</h3>
        <ul>
          {permissions.map(permission => (
            <li key={permission}>{permission}</li>
          ))}
        </ul>
      </div>
      
      {/* 管理员功能 */}
      <div className="admin-functions">
        <button>用户管理</button>
        <button>系统设置</button>
        <button>内容审核</button>
      </div>
    </div>
  );
}
```

## 实施建议

### 部署步骤

1. **配置权限服务**：创建AuthorizationService类和权限映射
2. **实现Hook**：创建useAuthorization Hook供组件使用
3. **添加保护组件**：实现AuthorizationGuard和AdminGuard组件
4. **更新API路由**：在需要权限的API端点添加权限检查
5. **测试验证**：确保不同角色的权限控制正常工作

### 最佳实践

1. **权限检查**：在前端和后端都要进行权限验证
2. **用户体验**：为无权限用户提供友好的提示信息
3. **性能优化**：缓存权限检查结果，避免重复计算
4. **安全考虑**：前端权限控制仅用于UI优化，后端必须验证

### 性能优化

```typescript
// 优化的权限Hook
export function useOptimizedAuthorization() {
  const { user } = useAuth();
  
  // 使用useMemo缓存权限计算结果
  const permissions = useMemo(() => {
    return user ? AuthorizationService.getUserPermissions(user) : [];
  }, [user]);
  
  const can = useCallback((permission: Permission) => {
    return permissions.includes(permission);
  }, [permissions]);
  
  const isAdmin = useMemo(() => {
    return AuthorizationService.isAdmin(user);
  }, [user]);
  
  return {
    user,
    permissions,
    can,
    isAdmin,
  };
}
```

## 修改指南

### ✅ 安全的修改

- 添加新的权限类型
- 扩展角色权限映射
- 优化权限检查逻辑
- 添加新的保护组件

### ⚠️ 需要谨慎的修改

- 修改现有权限定义
- 更改角色权限结构
- 移除基础权限检查
- 修改权限服务接口

### 🚨 危险操作

- 移除权限验证逻辑
- 硬编码权限检查
- 绕过权限保护
- 忽略权限安全检查

## 当前授权流程图

```
用户操作 → 权限检查 → 权限验证 → 允许/拒绝
    ↓         ↓          ↓          ↓
   组件交互   Hook检查   服务验证   UI反馈
```

## 总结

Alpha Seeker 当前采用简化的角色授权模型，具有以下特点：

1. **简洁性**：仅支持admin和user两种角色，易于理解和维护
2. **实用性**：覆盖平台核心功能的权限控制需求
3. **安全性**：前后端双重验证，确保权限控制有效
4. **扩展性**：基于权限枚举的设计便于未来功能扩展

该授权系统为平台提供了必要的权限管理，支持投票、评论、审核等核心功能的访问控制，同时保持了良好的开发体验和系统性能。
