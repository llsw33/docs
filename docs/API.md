# SyncoBoard API 接口文档

## 1. 通用约定
- Base URL：`/api`
- 认证：Authorization: Bearer <JWT>
- 返回结构：
```json
{
  "code": 0,
  "message": "ok",
  "data": {}
}
```
- 分页参数（通用）：`page`、`size` 或 `before`、`limit`
- 时间格式：ISO 8601（UTC）

## 2. 认证接口
### 2.1 注册
- POST `/api/auth/register`
- 请求：`{ "email": "", "password": "", "name": "" }`
- 响应：用户信息 + JWT

### 2.2 登录
- POST `/api/auth/login`
- 请求：`{ "email": "", "password": "" }`
- 响应：JWT + refresh token
- 备注：refresh token 建议存于 HttpOnly Cookie

### 2.3 刷新
- POST `/api/auth/refresh`
- 请求：`{ "refreshToken": "" }`

### 2.4 退出
- POST `/api/auth/logout`

## 3. 项目接口
### 3.1 项目列表
- GET `/api/projects`

### 3.2 创建项目
- POST `/api/projects`
- 请求：`{ "name": "", "description": "" }`

### 3.3 项目详情
- GET `/api/projects/{id}`

### 3.4 更新项目
- PUT `/api/projects/{id}`

### 3.5 删除项目
- DELETE `/api/projects/{id}`

### 3.6 邀请链接
- POST `/api/projects/{id}/invite`
- 响应：`{ "inviteToken": "" }`

### 3.7 加入项目
- POST `/api/projects/{id}/members`
- 请求：`{ "inviteToken": "" }`

### 3.8 成员列表
- GET `/api/projects/{id}/members`
- 响应示例：
```json
{
  "data": [
    {
      "userId": 1,
      "name": "Alice",
      "memberRole": "owner",
      "joinedAt": "2025-01-01T10:00:00Z",
      "lastReadAt": "2025-01-02T12:00:00Z"
    }
  ]
}
```

## 4. 看板接口
### 4.1 列列表
- GET `/api/projects/{id}/board/lists`

### 4.2 创建列
- POST `/api/projects/{id}/board/lists`
- 请求：`{ "title": "", "position": 1000 }`

### 4.3 更新列
- PUT `/api/projects/{id}/board/lists/{listId}`

### 4.4 列排序
- POST `/api/projects/{id}/board/lists/{listId}/reorder`
- 请求：`{ "position": 3000 }`

### 4.5 卡片 CRUD
- GET `/api/projects/{id}/board/cards?listId=...`
- POST `/api/projects/{id}/board/cards`
- PUT `/api/projects/{id}/board/cards/{cardId}`
- DELETE `/api/projects/{id}/board/cards/{cardId}`

### 4.6 卡片移动/拖拽
- POST `/api/projects/{id}/board/cards/{cardId}/move`
- 请求：`{ "fromListId": 1, "toListId": 2, "position": 3500 }`
- 响应：更新后的卡片对象

## 5. 聊天接口
### 5.1 获取消息
- GET `/api/projects/{id}/chat/messages?before=timestamp&limit=50`

### 5.2 发送消息
- POST `/api/projects/{id}/chat/messages`
- 请求：`{ "content": "" }`

## 6. Dashboard 接口
- GET `/api/projects/{id}/dashboard`
- 响应示例：
```json
{
  "kpi": {
    "totalCards": 120,
    "overdueCards": 5,
    "activeMembers": 8
  },
  "charts": {
    "cardsCreatedLast7Days": [1,2,3,5,8,13,21],
    "cardsByStatus": [
      {"name": "Todo", "value": 30},
      {"name": "Doing", "value": 50},
      {"name": "Done", "value": 40}
    ]
  }
}
```

## 7. WebSocket 事件规范
### 7.1 看板
- topic：`/topic/projects/{projectId}/board`
- events：CARD_CREATED, CARD_UPDATED, CARD_MOVED, CARD_DELETED, LIST_CREATED, LIST_REORDERED

### 7.2 聊天
- topic：`/topic/projects/{projectId}/chat`
- events：MESSAGE_CREATED

### 7.3 在线状态
- topic：`/topic/projects/{projectId}/presence`
- events：USER_ONLINE, USER_OFFLINE

### 7.4 通用事件结构
```json
{
  "event": "CARD_MOVED",
  "projectId": 1,
  "actorId": 2,
  "payload": {}
}
```

## 8. 错误码规范（示例）
- 400 参数错误
- 401 未登录
- 403 无权限
- 404 资源不存在
