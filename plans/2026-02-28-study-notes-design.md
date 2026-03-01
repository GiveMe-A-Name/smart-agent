# 学习笔记应用设计方案

## 1. 项目概述

- **项目名称**: StudyNotes
- **项目类型**: Web 应用 (React + Node.js)
- **核心功能**: 程序员学习笔记应用，支持 Markdown、代码高亮、文件夹层级组织
- **目标用户**: 程序员个人使用

## 2. 技术架构

| 层级 | 技术选型 |
|------|----------|
| 前端框架 | React + Vite |
| 后端框架 | Node.js + Express |
| 数据库 | SQLite + Prisma ORM |
| Markdown | react-markdown + react-syntax-highlighter |
| 样式 | CSS Modules / Styled Components |

## 3. 数据模型

### Folder (文件夹)
```prisma
model Folder {
  id        String   @id @default(uuid())
  name      String
  parentId  String?
  parent    Folder?  @relation("FolderToFolder", fields: [parentId], references: [id])
  children  Folder[] @relation("FolderToFolder")
  notes     Note[]
  createdAt DateTime @default(now())
}
```

### Note (笔记)
```prisma
model Note {
  id        String   @id @default(uuid())
  title     String
  content   String   @default("")
  folderId  String
  folder    Folder   @relation(fields: [folderId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

## 4. 核心功能

### 4.1 文件夹管理
- 创建文件夹（支持选择父文件夹）
- 重命名文件夹
- 删除文件夹（同时删除所有子文件夹和笔记）
- 树形结构展示，支持展开/折叠

### 4.2 笔记 CRUD
- 创建笔记（选择所属文件夹）
- 编辑笔记标题和内容
- 删除笔记
- 笔记列表展示（按更新时间排序）

### 4.3 Markdown 编辑器
- 左侧：Markdown 编辑区
- 右侧：实时 HTML 预览
- 支持代码块语法高亮（JavaScript, Python, TypeScript, Go 等）
- 自动保存（或手动保存按钮）

### 4.4 搜索功能
- 搜索笔记标题
- 搜索笔记内容
- 实时搜索结果

## 5. 界面布局

```
┌─────────────────────────────────────────────────────────────────┐
│  Header: App Title + Search Bar                                │
├────────────┬────────────────────┬───────────────────────────────┤
│            │                    │                               │
│  Sidebar   │   Note List        │   Editor + Preview            │
│  (Folder   │   (Notes in        │   ┌─────────┬─────────────┐   │
│   Tree)    │    selected        │   │  Edit   │   Preview   │   │
│            │    folder)         │   │         │             │   │
│            │                    │   │         │             │   │
│            │                    │   └─────────┴─────────────┘   │
│            │                    │                               │
└────────────┴────────────────────┴───────────────────────────────┘
```

- **左侧边栏 (20%)**: 文件夹树形结构
- **中间列表 (25%)**: 当前文件夹下的笔记列表
- **右侧编辑器 (55%)**: Markdown 编辑 + 实时预览

## 6. API 设计

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | /api/folders | 获取所有文件夹 |
| POST | /api/folders | 创建文件夹 |
| PUT | /api/folders/:id | 更新文件夹 |
| DELETE | /api/folders/:id | 删除文件夹 |
| GET | /api/notes | 获取笔记列表 (可选 folderId) |
| GET | /api/notes/:id | 获取单个笔记 |
| POST | /api/notes | 创建笔记 |
| PUT | /api/notes/:id | 更新笔记 |
| DELETE | /api/notes/:id | 删除笔记 |
| GET | /api/search?q=xxx | 搜索笔记 |

## 7. 潜在风险与解决方案

| 风险 | 解决方案 |
|------|----------|
| 删除文件夹误删数据 | 删除前二次确认对话框 |
| Markdown XSS 攻击 | 使用 react-markdown 默认安全防护 |
| 大笔记性能问题 | 考虑分页或虚拟滚动（后续优化） |
| 数据丢失 | SQLite 本地存储，定期备份 |

## 8. 验收标准

- [ ] 可以创建/编辑/删除文件夹
- [ ] 可以创建/编辑/删除笔记
- [ ] Markdown 编辑器左右分栏，实时预览
- [ ] 代码块有语法高亮
- [ ] 搜索可以找到标题和内容匹配的笔记
- [ ] 界面美观，响应式布局
