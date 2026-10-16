# 任务逾期提醒 — 代码证据档

> 虚构示例：仓库、路径、行号和代码均为虚构。

生成时间：2026-10-16
代码访问级别：L2
检索方式：本地 `rg` 关键词检索 + 源码回读
代码来源：`~/code/board`

## 一、检索范围

| 仓库 | 分支或提交 | 说明 | 覆盖的链路 |
|---|---|---|---|
| board-server | main @ 本地快照 | 任务与通知后端 | 判定、存储、投递 |
| board-web | main @ 本地快照 | Web 前端 | 前端展示 |

## 二、确证证据

### 任务截止时间与状态

- `board-server/src/tasks/task.entity.ts:42` `dueAt: Date | null`
  核对结论：截止时间可以为空；为空的任务不存在逾期。
- `board-server/src/tasks/task-status.ts:3` `enum TaskStatus { Todo, InProgress, Done, Archived }`
  核对结论：共四种状态，没有独立的“已删除”状态，删除为物理删除。

### 现有通知能力

- `board-server/src/notifications/notification.service.ts:88` `send(userId, type, payload, channels)`
  核对结论：已支持站内通知和 App 推送两个渠道，调用方指定渠道；推送通过现有推送通道异步发送。
- `board-server/src/notifications/preferences.ts:15` `NOTIFICATION_TYPES = ['mention', 'assigned', 'comment']`
  核对结论：用户可以按类型关闭通知，目前没有“逾期提醒”类型。

### 前端逾期判断

- `board-web/src/features/tasks/TaskRow.tsx:57` `const isOverdue = task.dueAt && task.dueAt < now && task.status !== 'done'`
  核对结论：前端判断逾期时只排除了“已完成”，没有排除“已归档”。已归档但未完成的任务在列表中仍显示为逾期。

## 三、待确认项

1. 已归档任务是否算逾期：前端口径（算）与本需求的默认口径（不算）不一致，需产品与研发确认后统一。
2. 截止时间只精确到日期时的判定时刻：`task.entity.ts` 只存储了时间戳，没有找到“仅日期”的标记字段。日期型截止时间在入库时如何换算，链路未读通。

## 四、证据边界

- 已覆盖：任务实体、状态枚举、通知服务、通知偏好、Web 列表的逾期展示。
- 未覆盖：iOS/Android App 端代码（本地无仓库）；推送通道内部的重试逻辑。
- 已剔除的误判：`board-server/src/reports/overdue-report.ts` 关键词命中，但只是统计报表，不在提醒链路上。
- 确证与方向：第二节均为代码级确证；第三节第 2 项仅为方向。
- 时效：基于本地快照，可能与线上分支存在差异。
