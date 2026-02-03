# SyncoBoard 概要设计文档（HLD）

## 1. 架构概览
采用前后端分离架构：
- 前端：Vue3 + Element Plus + ECharts + SortableJS/vuedraggable
- 后端：Spring Boot 3 + Spring Security + JWT + WebSocket(STOMP)
- 数据：PostgreSQL + Redis

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

## 4. 关键设计要点
- position 字段使用“整数间隔或浮点策略”减少全量重排
- WebSocket 事件包含 cardId、fromListId、toListId、newPosition
- RBAC：用户必须是项目成员

## 5. 部署架构
- docker-compose 运行：
  - backend（8080）
  - frontend（5173 或 nginx:80）
  - postgres
  - redis

## 6. 监控与日志
- Spring Boot Actuator
- 访问日志 + 审计日志

## 7. 扩展规划
- 引入 Yjs 进行文档与白板协同
- 多项目全局搜索
- 企业级权限与审计
