# SyncoBoard 详细设计文档（DDD）

## 1. 领域对象设计
### 1.1 User
- id, email, passwordHash, name, avatar, createdAt

### 1.2 Project
- id, name, description, ownerId, createdAt

### 1.3 Board/List/Card
- List：id, projectId, title, position
- Card：id, listId, title, description, priority, dueDate, assigneeId, position, createdAt, updatedAt

### 1.4 ChatMessage
- id, projectId, userId, content, createdAt

### 1.5 Activity
- id, projectId, actorId, type, payloadJson, createdAt

## 2. 服务层设计
### 2.1 AuthService
- register(email, password, name)
- login(email, password)
- refresh(token)
- logout(token)

### 2.2 ProjectService
- createProject(ownerId, name, description)
- generateInviteToken(projectId)
- addMember(projectId, userId, role)
- listMembers(projectId)

### 2.3 BoardService
- createList(projectId, title, position)
- updateList(listId, title)
- reorderList(listId, newPosition)
- createCard(listId, payload)
- updateCard(cardId, payload)
- moveCard(cardId, fromListId, toListId, newPosition)

### 2.4 ChatService
- listMessages(projectId, before, limit)
- createMessage(projectId, userId, content)

### 2.5 ActivityService
- record(projectId, actorId, type, payload)
- listActivities(projectId, page, size)

### 2.6 DashboardService
- getProjectKpi(projectId)
- getProjectCharts(projectId, range)

## 3. 关键流程
### 3.1 卡片拖拽流程
1. 前端拖拽完成
2. 调用 REST `reorder/move` 接口保存位置
3. 后端更新数据库
4. 后端广播 WebSocket `CARD_MOVED`
5. 在线成员 UI 同步更新

### 3.2 聊天消息流程
1. 前端发送消息
2. 后端保存消息
3. 广播 `MESSAGE_CREATED`

## 4. WebSocket 事件 payload 设计（示例）
```json
{
  "event": "CARD_MOVED",
  "projectId": "p1",
  "cardId": "c1",
  "fromListId": "l1",
  "toListId": "l2",
  "newPosition": 3000,
  "actorId": "u1",
  "timestamp": "2025-01-01T10:00:00Z"
}
```

## 5. 排序策略
- 位置字段采用整数间隔（默认 1000）
- 插入时取相邻位置中位数；必要时触发局部重排

## 6. 权限设计
- 项目访问：必须是 project_members
- 管理员角色可访问 `/app/admin/users`

## 7. 错误码设计（示例）
- 400：参数校验失败
- 401：未登录
- 403：无权限或非项目成员
- 404：资源不存在

## 8. 日志与审计
- 业务事件写入 activities
- 关键操作写入审计日志
