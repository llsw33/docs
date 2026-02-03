# SyncoBoard 概要设计文档（HLD）

## 1. 架构概览
采用前后端分离架构：
- 前端：Vue3 + Element Plus + ECharts + SortableJS/vuedraggable
- 后端：Spring Boot 3 + Spring Security + JWT + WebSocket(STOMP)
- 数据：PostgreSQL（本地直装）

### 1.1 逻辑分层
- 表现层：Vue3 + Element Plus
- 应用层：REST API + WebSocket 事件
- 领域层：项目、看板、聊天、活动流
- 基础设施层：数据库、内存缓存、对象存储（预留）

## 2. 模块划分
### 2.1 前端模块
- 认证模块：登录/注册/刷新
- 项目模块：项目列表、邀请、成员
- 看板模块：列/卡片、拖拽
- 聊天模块：消息列表与输入
- Dashboard 模块：KPI + 图表
- 活动流模块：审计事件展示
- 管理模块：管理员用户管理

### 2.2 后端模块
- Auth Service：注册、登录、JWT 管理
- Project Service：项目与成员管理
- Board Service：列/卡片、拖拽排序
- Chat Service：聊天消息
- Activity Service：活动流记录
- Dashboard Service：统计与图表数据

## 3. 数据流概述
1. 用户登录获取 JWT
2. 前端携带 JWT 调用 REST API
3. 看板拖拽完成后保存排序到数据库
4. 后端广播 WebSocket 事件给同项目在线成员

## 4. 安全与权限
- JWT 认证与过期策略
- RBAC 与项目成员校验
- WebSocket 订阅时校验项目成员身份

## 5. 关键设计要点
- position 字段使用“整数间隔或浮点策略”减少全量重排
- WebSocket 事件包含 cardId、fromListId、toListId、newPosition
- RBAC：用户必须是项目成员
- 聊天与活动流写入前进行内容长度校验
- WebSocket 事件与 REST 写库保持最终一致性

## 6. 部署架构
仅支持本地直装运行（无 Docker 依赖）：
- backend（8080）
- frontend（5173）
- postgres（本地服务）

### 6.1 运行时组件
- API Gateway（后端统一入口）
- STOMP Broker（内置或外部）
- Cache（内存缓存，用于会话与热点数据）

## 7. 监控与日志
- Spring Boot Actuator
- 访问日志 + 审计日志
- WebSocket 连接数监控与错误告警

## 8. 扩展规划
- 引入 Yjs 进行文档与白板协同
- 多项目全局搜索
- 企业级权限与审计
