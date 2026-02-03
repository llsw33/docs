# SyncoBoard 软件需求规格说明书（SRS）

## 1. 引言
### 1.1 目的
定义 SyncoBoard 系统的功能与非功能需求，为设计与实现提供一致依据。

### 1.2 范围
MVP 覆盖账号体系、项目空间、看板、聊天、仪表盘与活动流。

### 1.3 术语
- 项目空间（Project）
- 看板（Board）
- 列（List）
- 卡片（Card）
- 活动流（Activity）

## 2. 总体描述
### 2.1 产品视角
系统采用前后端分离架构，前端 Vue3，后端 Spring Boot 3，使用 WebSocket(STOMP) 实现实时同步。

### 2.2 用户特征
- 普通成员：查看/编辑项目
- 负责人：项目管理与成员邀请
- 管理员：用户管理

### 2.3 约束
- 后端：Java 17+，Spring Boot 3
- 数据库：PostgreSQL
- 缓存/消息：Redis
- 前端：Vue3 + Element Plus + ECharts

## 3. 功能需求
### 3.1 账号与认证
- FR-001：用户可注册账号
- FR-002：用户可登录并获取 JWT
- FR-003：用户可刷新 token
- FR-004：用户可退出并失效 token

### 3.2 项目空间
- FR-010：用户可创建项目空间
- FR-011：项目可生成邀请链接 token
- FR-012：用户可加入项目
- FR-013：项目成员列表可查看角色与加入时间

### 3.3 看板
- FR-020：列 CRUD
- FR-021：卡片 CRUD
- FR-022：卡片支持标题、描述、标签、优先级、截止日期、指派人、评论
- FR-023：卡片拖拽在列内/列间
- FR-024：拖拽结果写入数据库
- FR-025：拖拽通过 WebSocket 广播

### 3.4 聊天
- FR-030：项目频道消息列表
- FR-031：发送消息
- FR-032：记录最后读时间

### 3.5 Dashboard
- FR-040：展示 KPI 指标
- FR-041：展示折线图与饼图/柱状图

### 3.6 活动流
- FR-050：记录创建卡片、移动卡片、发表评论、成员加入
- FR-051：活动页展示

## 4. 外部接口需求
### 4.1 REST API
- 认证接口：/api/auth/register login refresh logout
- 项目接口：/api/projects CRUD + invite + members
- 看板接口：/api/projects/{id}/board lists/cards + reorder
- 聊天接口：/api/projects/{id}/chat/messages list + create
- Dashboard：/api/projects/{id}/dashboard

### 4.2 WebSocket
- /topic/projects/{projectId}/board
  - CARD_CREATED, CARD_UPDATED, CARD_MOVED, CARD_DELETED, LIST_CREATED, LIST_REORDERED
- /topic/projects/{projectId}/chat
  - MESSAGE_CREATED
- /topic/projects/{projectId}/presence
  - USER_ONLINE, USER_OFFLINE

## 5. 非功能需求
- NFR-001：看板与聊天实时同步 < 500ms（同城）
- NFR-002：系统可用性 ≥ 99.5%（MVP 目标）
- NFR-003：JWT 认证，所有接口 RBAC
- NFR-004：敏感信息加密存储（密码 Hash）
- NFR-005：日志与审计记录完整

## 6. 数据需求
- 采用 position 字段优化排序与拖拽
- 记录 last_read_at 计算未读

## 7. 假设与依赖
- 浏览器支持现代标准（Chrome/Edge/Safari 最新版）
- Redis 用于缓存与可选消息队列
