# SyncoBoard 数据库设计文档

## 1. 数据库选型
- PostgreSQL 作为主数据库
- Redis 用于缓存与 WebSocket session 管理

## 2. ER 关系（文本描述）
- User 与 Project：多对多（project_members）
- Project 与 List：一对多
- List 与 Card：一对多
- Card 与 Comment：一对多
- Project 与 ChatMessage：一对多
- Project 与 Activity：一对多

## 3. 表结构定义
### 3.1 users
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | bigserial | PK | 用户 ID |
| email | varchar(255) | UNIQUE, NOT NULL | 邮箱 |
| password_hash | varchar(255) | NOT NULL | 密码哈希 |
| name | varchar(100) | NOT NULL | 昵称 |
| avatar | varchar(255) | NULL | 头像 |
| created_at | timestamp | NOT NULL | 创建时间 |

### 3.2 roles
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | bigserial | PK | 角色 ID |
| name | varchar(50) | UNIQUE | 角色名称 |

### 3.3 user_roles
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| user_id | bigint | FK users.id | 用户 ID |
| role_id | bigint | FK roles.id | 角色 ID |

### 3.4 projects
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | bigserial | PK | 项目 ID |
| name | varchar(200) | NOT NULL | 项目名 |
| description | text | NULL | 描述 |
| owner_id | bigint | FK users.id | 项目负责人 |
| created_at | timestamp | NOT NULL | 创建时间 |

### 3.5 project_members
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| project_id | bigint | FK projects.id | 项目 ID |
| user_id | bigint | FK users.id | 用户 ID |
| member_role | varchar(50) | NOT NULL | 成员角色 |
| joined_at | timestamp | NOT NULL | 加入时间 |
| last_read_at | timestamp | NULL | 最后读时间 |
| status | varchar(20) | NOT NULL | 成员状态（active/inactive） |

### 3.6 lists
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | bigserial | PK | 列 ID |
| project_id | bigint | FK projects.id | 项目 ID |
| title | varchar(200) | NOT NULL | 列标题 |
| position | bigint | NOT NULL | 排序 |
| created_at | timestamp | NOT NULL | 创建时间 |
| updated_at | timestamp | NOT NULL | 更新时间 |

### 3.7 cards
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | bigserial | PK | 卡片 ID |
| list_id | bigint | FK lists.id | 所属列 |
| title | varchar(200) | NOT NULL | 标题 |
| description | text | NULL | 描述 |
| priority | varchar(20) | NULL | 优先级 |
| due_date | date | NULL | 截止日期 |
| assignee_id | bigint | FK users.id | 指派人 |
| position | bigint | NOT NULL | 排序 |
| created_at | timestamp | NOT NULL | 创建时间 |
| updated_at | timestamp | NOT NULL | 更新时间 |
| status | varchar(20) | NULL | 业务状态（todo/doing/done） |

### 3.8 labels
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | bigserial | PK | 标签 ID |
| project_id | bigint | FK projects.id | 项目 ID |
| name | varchar(100) | NOT NULL | 标签名称 |
| color | varchar(20) | NOT NULL | 色值 |

### 3.9 card_labels
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| card_id | bigint | FK cards.id | 卡片 ID |
| label_id | bigint | FK labels.id | 标签 ID |

### 3.10 comments
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | bigserial | PK | 评论 ID |
| card_id | bigint | FK cards.id | 卡片 ID |
| user_id | bigint | FK users.id | 评论人 |
| content | text | NOT NULL | 评论内容 |
| created_at | timestamp | NOT NULL | 创建时间 |

### 3.11 chat_messages
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | bigserial | PK | 消息 ID |
| project_id | bigint | FK projects.id | 项目 ID |
| user_id | bigint | FK users.id | 发送人 |
| content | text | NOT NULL | 内容 |
| created_at | timestamp | NOT NULL | 创建时间 |
| edited_at | timestamp | NULL | 编辑时间 |

### 3.12 activities
| 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | bigserial | PK | 活动 ID |
| project_id | bigint | FK projects.id | 项目 ID |
| actor_id | bigint | FK users.id | 操作人 |
| type | varchar(50) | NOT NULL | 类型 |
| payload_json | jsonb | NOT NULL | 事件详情 |
| created_at | timestamp | NOT NULL | 创建时间 |

## 4. 索引设计（建议）
- users(email)
- project_members(project_id, user_id)
- lists(project_id, position)
- cards(list_id, position)
- chat_messages(project_id, created_at)
- activities(project_id, created_at)
- cards(assignee_id, due_date)

## 5. 数据一致性与约束
- 删除项目时级联删除 lists、cards、chat_messages、activities
- 删除 list 时级联删除 cards
