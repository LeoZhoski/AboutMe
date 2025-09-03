# AboutMe
## @AlphaSeeker.md 简略版的需求文档

## @architect/ 详细的架构与功能文档
### 1. 数据与状态 (`1_data_and_state/`)
定义数据存储、传输和管理的架构
- `data-model.md` - 统一数据模型设计（包含 Airtable 字段定义和类型定义）
- `api_contracts.md` - 前后端数据交换协议
- `state_management.md` - 前端状态管理方案
- `domain-mapping-functions.md` - 个人能力提升三级映射函数

### 2. 权限与安全 (`2_security/`)
用户身份验证和权限控制机制
- `authentication.md` - 用户身份验证流程
- `authorization.md` - 权限控制和访问管理

### 3. 核心业务逻辑 (`3_business_logic/`)
应用核心功能的实现逻辑
- `payment_flow.md` - 支付业务流程
- 其他关键业务流程...

### 4. 配置与环境 (`4_configuration/`)
应用配置和环境管理
- `env_variables.md` - 环境变量配置说明

### 5. 外部依赖 (`5_dependencies/`)
第三方服务和库的集成
- `key_libraries.md` - 核心第三方库说明
