# Task Manager Module

## 1. Tổng quan

Module Task Manager cho phép quản lý công việc và nhiệm vụ trong hệ thống HRM. Module này hỗ trợ:
- Tạo và quản lý tasks
- Phân công tasks cho nhân viên
- Theo dõi tiến độ và trạng thái tasks
- Quản lý projects và nhóm tasks
- Comment và attachment trên tasks
- Thông báo về các sự kiện liên quan đến tasks

## 2. Các tính năng chính

### 2.1. Quản lý Projects
- Tạo, sửa, xóa projects
- Gán project manager và team members
- Thiết lập thời gian bắt đầu/kết thúc dự án
- Quản lý trạng thái dự án (Planning, In Progress, On Hold, Completed, Cancelled)
- Liên kết project với department

### 2.2. Quản lý Tasks
- Tạo tasks với thông tin chi tiết (title, description, priority, due date)
- Phân công tasks cho assignees
- Thiết lập task dependencies
- Quản lý trạng thái task (Todo, In Progress, In Review, Done, Cancelled)
- Gán labels/tags cho tasks
- Ưu tiên tasks (Low, Medium, High, Urgent)
- Ước tính thời gian hoàn thành

### 2.3. Theo dõi tiến độ
- Cập nhật tiến độ task
- Log thời gian làm việc
- Theo dõi thời gian thực tế vs ước tính
- Báo cáo tiến độ theo project/department/user

### 2.4. Comments và Attachments
- Thêm comments vào tasks
- Upload attachments (files, images)
- Mention users trong comments
- Lịch sử thay đổi

### 2.5. Notifications
- Thông báo khi được assign task mới
- Thông báo khi task được cập nhật
- Thông báo khi task sắp đến deadline
- Thông báo khi có comment mới
- Thông báo khi task được hoàn thành

### 2.6. Báo cáo và Dashboard
- Dashboard tổng quan tasks của user
- Báo cáo tasks theo project/department
- Thống kê hiệu suất làm việc
- Báo cáo tasks quá hạn

## 3. Database Schema

### 3.1. Project Entity
| Trường                   | Mô tả                                       |
| ------------------------ | ------------------------------------------- |
| `id`                     | Định danh dự án                             |
| `name`                   | Tên dự án                                   |
| `code`                   | Mã dự án                                    |
| `description`            | Mô tả dự án                                 |
| `department`             | Phòng ban phụ trách                         |
| `projectManager`         | Người quản lý dự án                         |
| `startDate`, `endDate`   | Thời gian thực hiện                         |
| `status`                 | Trạng thái dự án (Planning, In Progress, …) |
| `isActive`               | Đang hoạt động hay không                    |
| `members`                | Danh sách thành viên                        |
| `tasks`                  | Danh sách công việc                         |
| `createdAt`, `updatedAt` | Thời gian tạo và cập nhật                   |


### 3.2. Task Entity
| Trường                                | Mô tả                                                      |
| ------------------------------------- | ---------------------------------------------------------- |
| `id`                                  | Định danh công việc                                        |
| `title`                               | Tên công việc                                              |
| `description`                         | Mô tả chi tiết                                             |
| `project`                             | Dự án liên quan                                            |
| `department`                          | Phòng ban phụ trách                                        |
| `createdBy`                           | Người tạo                                                  |
| `assignedTo`                          | Người chịu trách nhiệm chính                               |
| `assignees`                           | Danh sách người thực hiện                                  |
| `status`                              | Trạng thái (Todo, In Progress, In Review, Done, Cancelled) |
| `priority`                            | Mức độ ưu tiên                                             |
| `startDate`, `dueDate`, `completedAt` | Mốc thời gian                                              |
| `estimatedHours`, `actualHours`       | Thời gian ước tính & thực tế                               |
| `progress`                            | Tiến độ                                                    |
| `labels`                              | Tags phân loại                                             |
| `parentTask`                          | Task cha                                                   |
| `subtasks`                            | Các task con                                               |
| `dependencies`                        | Quan hệ phụ thuộc                                          |
| `comments`                            | Trao đổi công việc                                         |
| `attachments`                         | Tài liệu đính kèm                                          |
| `timeLogs`                            | Nhật ký thời gian                                          |
| `createdAt`, `updatedAt`              | Thời gian tạo và cập nhật                                  |

### 3.3. Task Dependency Entity

| Thành phần               | Mô tả                                                        |
| ------------------------ | ------------------------------------------------------------ |
| `id`                     | Định danh quan hệ phụ thuộc                                  |
| `taskId`                 | Task đang xét                                                |
| `dependsOnTaskId`        | Task mà `taskId` phụ thuộc vào                               |
| `type`                   | Loại quan hệ phụ thuộc: `BLOCKS`, `BLOCKED_BY`, `RELATES_TO` |
| `createdAt`, `updatedAt` | Thời gian tạo và cập nhật                                    |


### 3.4. Task Comment Entity
| Thành phần               | Mô tả                                         |
| ------------------------ | --------------------------------------------- |
| `id`                     | Định danh của comment                         |
| `taskId`                 | Task mà comment thuộc về                      |
| `userId`                 | Người tạo comment                             |
| `content`                | Nội dung bình luận                            |
| `mentionedUserIds`       | Danh sách user được mention trong comment     |
| `createdAt`, `updatedAt` | Thời gian tạo và cập nhật                     |


### 3.5. Task Attachment Entity

| Thành phần               | Mô tả                           |
| ------------------------ | ------------------------------- |
| `id`                     | Định danh tệp                   |
| `taskId`                 | Task chứa tệp                   |
| `uploadedById`           | Người upload                    |
| `fileName`               | Tên file                        |
| `fileUrl`                | Đường dẫn lưu trữ               |
| `fileType`               | Loại file (pdf, image, docx, …) |
| `fileSize`               | Dung lượng file                 |
| `createdAt`, `updatedAt` | Thời điểm upload                |


### 3.6. Task Time Log Entity
| Thành phần    | Mô tả              |
| ------------- | ------------------ |
| `id`          | Định danh log      |
| `taskId`      | Task được ghi nhận |
| `userId`      | Người thực hiện    |
| `startTime`   | Thời điểm bắt đầu  |
| `endTime`     | Thời điểm kết thúc |
| `hours`       | Số giờ làm việc    |
| `description` | Ghi chú công việc  |
| `createdAt`   | Thời điểm ghi nhận |


## 4. Enums

### 4.1. ProjectStatus Enum
```typescript
export enum ProjectStatus {
  PLANNING = 'PLANNING',
  IN_PROGRESS = 'IN_PROGRESS',
  ON_HOLD = 'ON_HOLD',
  COMPLETED = 'COMPLETED',
  CANCELLED = 'CANCELLED',
}
```

### 4.2. TaskStatus Enum
```typescript
export enum TaskStatus {
  TODO = 'TODO',
  IN_PROGRESS = 'IN_PROGRESS',
  IN_REVIEW = 'IN_REVIEW',
  DONE = 'DONE',
  CANCELLED = 'CANCELLED',
}
```

### 4.3. TaskPriority Enum
```typescript
export enum TaskPriority {
  LOW = 'LOW',
  MEDIUM = 'MEDIUM',
  HIGH = 'HIGH',
  URGENT = 'URGENT',
}
```

### 4.4. DependencyType Enum
```typescript
export enum DependencyType {
  BLOCKS = 'BLOCKS',           // Task này chặn task khác
  BLOCKED_BY = 'BLOCKED_BY',   // Task này bị chặn bởi task khác
  RELATES_TO = 'RELATES_TO',   // Task này liên quan đến task khác
}
```

## 5. API Endpoints

### 5.1. Project Endpoints

#### POST /projects
Tạo project mới
- **Body**: `CreateProjectDto`
- **Permissions**: Admin, Manager
- **Response**: Project entity

#### GET /projects
Lấy danh sách projects
- **Query**: `GetProjectsDto` (filters, pagination)
- **Permissions**: All authenticated users
- **Business Logic**:
  - Employee: Chỉ xem projects mình tham gia
  - Manager: Xem projects của department mình quản lý
  - Admin: Xem tất cả projects

#### GET /projects/:id
Lấy chi tiết project
- **Permissions**: Xem theo business logic trên

#### PUT /projects/:id
Cập nhật project
- **Body**: `UpdateProjectDto`
- **Permissions**: Admin, Project Manager của project đó

#### DELETE /projects/:id
Xóa project (soft delete)
- **Permissions**: Admin

#### GET /projects/:id/tasks
Lấy danh sách tasks của project
- **Query**: Filters, pagination
- **Permissions**: Xem theo business logic của project

#### POST /projects/:id/members
Thêm members vào project
- **Body**: `{ userIds: number[] }`
- **Permissions**: Admin, Project Manager

#### DELETE /projects/:id/members/:userId
Xóa member khỏi project
- **Permissions**: Admin, Project Manager

### 5.2. Task Endpoints

#### POST /tasks
Tạo task mới
- **Body**: `CreateTaskDto`
- **Permissions**: Project Member, Project Manager, Admin
- **Response**: Task entity

#### GET /tasks
Lấy danh sách tasks
- **Query**: `GetTasksDto` (filters: status, priority, assignee, project, department, dueDate, pagination)
- **Permissions**: All authenticated users
- **Business Logic**:
  - Employee: Xem tasks được assign cho mình và tasks của projects mình tham gia
  - Manager: Xem tasks của department mình quản lý
  - Admin: Xem tất cả tasks

#### GET /tasks/my
Lấy danh sách tasks của user hiện tại
- **Query**: Filters, pagination
- **Permissions**: All authenticated users

#### GET /tasks/:id
Lấy chi tiết task
- **Permissions**: Xem theo business logic trên

#### PUT /tasks/:id
Cập nhật task
- **Body**: `UpdateTaskDto`
- **Permissions**: Task creator, Assigned user, Project Manager, Admin

#### DELETE /tasks/:id
Xóa task (soft delete)
- **Permissions**: Task creator, Project Manager, Admin

#### PUT /tasks/:id/status
Cập nhật trạng thái task
- **Body**: `{ status: TaskStatus }`
- **Permissions**: Task creator, Assigned user, Project Manager, Admin

#### PUT /tasks/:id/assign
Gán task cho user(s)
- **Body**: `{ assigneeIds: number[] }`
- **Permissions**: Task creator, Project Manager, Admin

#### PUT /tasks/:id/progress
Cập nhật tiến độ task
- **Body**: `{ progress: number }` (0-100)
- **Permissions**: Task creator, Assigned user, Project Manager, Admin

#### POST /tasks/:id/subtasks
Tạo subtask
- **Body**: `CreateTaskDto`
- **Permissions**: Task creator, Assigned user, Project Manager, Admin

#### GET /tasks/:id/dependencies
Lấy danh sách dependencies của task
- **Permissions**: Task creator, Assigned user, Project Manager, Admin

#### POST /tasks/:id/dependencies
Thêm dependency
- **Body**: `{ dependsOnTaskId: number, type: DependencyType }`
- **Permissions**: Task creator, Assigned user, Project Manager, Admin

#### DELETE /tasks/:id/dependencies/:dependencyId
Xóa dependency
- **Permissions**: Task creator, Project Manager, Admin

### 5.3. Task Comment Endpoints

#### POST /tasks/:id/comments
Thêm comment vào task
- **Body**: `{ content: string, mentionedUserIds?: number[] }`
- **Permissions**: Project Member, Project Manager, Admin

#### GET /tasks/:id/comments
Lấy danh sách comments của task
- **Query**: Pagination
- **Permissions**: Project Member, Project Manager, Admin

#### PUT /tasks/:taskId/comments/:commentId
Cập nhật comment
- **Body**: `{ content: string }`
- **Permissions**: Comment owner, Admin

#### DELETE /tasks/:taskId/comments/:commentId
Xóa comment
- **Permissions**: Comment owner, Task creator, Admin

### 5.4. Task Attachment Endpoints

#### POST /tasks/:id/attachments
Upload attachment
- **Body**: Multipart form data (file)
- **Permissions**: Project Member, Project Manager, Admin

#### GET /tasks/:id/attachments
Lấy danh sách attachments
- **Permissions**: Project Member, Project Manager, Admin

#### DELETE /tasks/:taskId/attachments/:attachmentId
Xóa attachment
- **Permissions**: Attachment owner, Task creator, Admin

### 5.5. Task Time Log Endpoints

#### POST /tasks/:id/time-logs
Tạo time log
- **Body**: `{ startTime: Date, endTime?: Date, hours: number, description?: string }`
- **Permissions**: Assigned user, Project Manager, Admin

#### GET /tasks/:id/time-logs
Lấy danh sách time logs của task
- **Permissions**: Project Member, Project Manager, Admin

#### GET /time-logs/my
Lấy time logs của user hiện tại
- **Query**: Date range, pagination
- **Permissions**: Assigned user, Project Manager, Admin

#### PUT /time-logs/:id
Cập nhật time log
- **Body**: `UpdateTimeLogDto`
- **Permissions**: Assigned user, Project Manager, Admin

#### DELETE /time-logs/:id
Xóa time log
- **Permissions**: Assigned user, Project Manager, Admin

### 5.6. Dashboard & Reports Endpoints

#### GET /tasks/dashboard
Dashboard tổng quan tasks của user
- **Permissions**: Project Manager, Admin

#### GET /tasks/reports
Báo cáo tasks
- **Query**: `{ projectId?, departmentId?, userId?, startDate?, endDate? }`
- **Permissions**: Project Manager, Admin


## 6. Business Logic & Rules

### 6.1. Task Assignment Rules
- Một task có thể được gán cho nhiều người (assignees)
- Khi task được gán, gửi notification cho assignees
- Chỉ người tạo task, Admin, hoặc Manager mới có thể gán task

### 6.2. Task Status Flow
```
TODO → IN_PROGRESS → IN_REVIEW → DONE
  ↓         ↓            ↓
CANCELLED  CANCELLED   CANCELLED
```
- Task có thể bị cancel ở bất kỳ trạng thái nào
- Khi task chuyển sang DONE, tự động set `completedAt` và `progress = 100`
- Khi task chuyển sang IN_PROGRESS, tự động set `startDate` nếu chưa có

### 6.3. Task Dependencies
- Task không thể chuyển sang IN_PROGRESS nếu có dependencies chưa hoàn thành (BLOCKED_BY)
- Khi task được hoàn thành, tự động kiểm tra và unlock các tasks bị block bởi nó
- Không cho phép tạo circular dependencies

### 6.4. Time Tracking
- Time logs được tính vào `actualHours` của task
- Khi `actualHours` > `estimatedHours`, có thể cảnh báo
- Time logs có thể được tạo với `endTime` hoặc chỉ `hours`

### 6.5. Subtasks
- Subtask tự động kế thừa project và department từ parent task
- Khi tất cả subtasks hoàn thành, có thể tự động đánh dấu parent task là DONE
- Subtask có thể có subtasks riêng 

### 6.6. Notifications
- Gửi notification khi:
  - Task được assign cho user
  - Task status thay đổi
  - Task được comment (mention users)
  - Task sắp đến deadline (1 ngày trước)
  - Task quá hạn
  - Task dependency được hoàn thành

### 6.7. Project Rules
- Project có thể có nhiều members
- Project Manager có thể quản lý tất cả tasks trong project
- Khi project status = CANCELLED, tất cả tasks chưa hoàn thành tự động chuyển sang CANCELLED
- Khi project status = COMPLETED, chỉ cho phép tạo tasks mới nếu có quyền đặc biệt
