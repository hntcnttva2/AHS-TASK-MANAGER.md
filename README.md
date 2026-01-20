# Task Manager Specification - Đặc Tả Tính Năng Quản Lý Công Việc

## 1. Tổng Quan

Tính năng **Task Manager** (Quản lý công việc) được tích hợp vào hệ thống quản lý nhân sự để tự động hóa việc tạo và quản lý các công việc cần thực hiện. Hệ thống sẽ tự động tạo task dựa trên các hành động (action) trong các module khác và tự động đánh dấu hoàn thành khi hành động tương ứng được thực hiện.

## 2. Mục Tiêu

### 2.1. Mục Tiêu Chính

- Mỗi người dùng có thể xem danh sách công việc cần làm trong ngày hôm nay
- Tự động sinh task mới khi có sự kiện xảy ra trong hệ thống (ví dụ: nhân viên tạo leave request)
- Tự động đánh dấu task là hoàn thành khi hành động tương ứng được thực hiện (ví dụ: quản lý approve leave request)
- Tạo task theo workflow template với các bước tuần tự
- Quản lý dependencies giữa các task: task sau chỉ được thực hiện khi task trước hoàn thành
- Tự động tạo task tiếp theo khi task hiện tại hoàn thành
- Task Template: mẫu định nghĩa sẵn cho một task đơn lẻ (title, description, type, action, role được gán, SLA/due date offset, priority) để tái sử dụng

## 3. Database

### 3.1. Enum Types

#### 3.1.1. task_status_enum

**Các giá trị:**

- `PENDING`: Chờ xử lý (mặc định)
- `IN_PROGRESS`: Đang thực hiện
- `COMPLETED`: Đã hoàn thành
- `CANCELLED`: Đã hủy

#### 3.1.2. task_type_enum

**Các giá trị:**

- `LEAVE_APPROVAL`: Phê duyệt đơn nghỉ phép
- _(Có thể mở rộng thêm các loại khác trong tương lai)_
#### 3.1.3. task_action_enum

**Các giá trị:**

- `APPROVE_LEAVE_REQUEST`: Phê duyệt đơn nghỉ phép
- `REJECT_LEAVE_REQUEST`: Từ chối đơn nghỉ phép
- _(Có thể mở rộng thêm các action khác trong tương lai)_
### 3.2. Bảng task

#### 3.2.1. Cấu Trúc Bảng

| Tên Cột               | Kiểu Dữ Liệu     | Nullable | Mặc Định             | Mô Tả                                         |
| --------------------- | ---------------- | -------- | -------------------- | --------------------------------------------- |
| `id`                  | SERIAL           | NO       | AUTO_INCREMENT       | Primary key, tự động tăng                     |
| `title`               | varchar(255)     | NO       | -                    | Tiêu đề của task                              |
| `description`         | text             | YES      | NULL                 | Mô tả chi tiết của task                       |
| `assigned_to_id`      | int              | NO       | -                    | ID người được giao task (FK → user.id)        |
| `status`              | task_status_enum | NO       | 'PENDING'            | Trạng thái của task                           |
| `type`                | task_type_enum   | NO       | -                    | Loại task                                     |
| `action`              | task_action_enum | NO       | -                    | Hành động cần thực hiện                       |
| `related_entity_type` | varchar(100)     | YES      | NULL                 | Loại entity liên quan (ví dụ: "LeaveRequest") |
| `related_entity_id`   | int              | YES      | NULL                 | ID của entity liên quan                       |
| `workflow_instance_id` | int          | YES      | NULL     | FK → workflow_instance.id                       |
| `workflow_step_order`  | int          | YES      | NULL     | Thứ tự bước trong workflow                      |
| `depends_on_task_id`   | int          | YES      | NULL     | FK → task.id (task phụ thuộc)                   |
| `can_start`            | boolean      | NO       | true     | Task có thể bắt đầu chưa (phụ thuộc task trước) |
| `due_date`            | date             | YES      | NULL                 | Ngày hết hạn của task                         |
| `completed_at`        | date             | YES      | NULL                 | Ngày hoàn thành task                          |
| `created_at`          | timestamp(6)     | NO       | CURRENT_TIMESTAMP(6) | Ngày tạo task                                 |
| `updated_at`          | timestamp(6)     | NO       | CURRENT_TIMESTAMP(6) | Ngày cập nhật task                            |

### 3.3. Bảng Workflow Template

#### 3.3.1. Cấu Trúc Bảng workflow_template

| Tên Cột        | Kiểu Dữ Liệu | Nullable | Mặc Định | Mô Tả                          |
| -------------- | ------------ | -------- | -------- | ------------------------------ |
| id             | SERIAL       | NO       | AUTO     | Primary key                    |
| name           | varchar(255) | NO       | -        | Tên workflow template          |
| description    | text         | YES      | NULL     | Mô tả workflow                 |
| created_by_id  | int          | NO       | -        | FK → user.id (người tạo)       |
| department_id  | int          | YES      | NULL     | Phòng ban sở hữu workflow      |
| is_active   | boolean      | NO       | true     | Trạng thái active của template |
| created_at     | timestamp(6) | NO       | NOW()    | Ngày tạo                       |
| updated_at     | timestamp(6) | NO       | NOW()    | Ngày cập nhật                  |


#### 3.3.2. Cấu Trúc Bảng workflow_step

| Tên Cột                | Kiểu Dữ Liệu     | Nullable | Mặc Định | Mô Tả                                             |
| ---------------------- | ---------------- | -------- | -------- | ------------------------------------------------- |
| `id`                   | SERIAL           | NO       | AUTO     | Primary key                                       |
| `workflow_template_id` | int              | NO       | -        | FK → workflow_template.id                         |
| `step_order`           | int              | NO       | -        | Thứ tự bước trong workflow (1, 2, 3...)           |
| `step_name`            | varchar(255)     | NO       | -        | Tên bước (ví dụ: "Code", "Review Code")           |
| `task_template_id`     | int              | YES      | NULL     | FK → task_template.id (nếu dùng task template)    |
| `task_type`            | task_type_enum   | YES      | NULL     | Loại task (fallback nếu không dùng task template) |
| `task_action`          | task_action_enum | YES      | NULL     | Action (fallback nếu không dùng task template)    |
| `assigned_to_role`     | varchar(100)     | YES      | NULL     | Role được gán task                                         |
| `is_required`          | boolean          | NO       | true     | Bước này có bắt buộc không                        |
| `can_skip`             | boolean          | NO       | false    | Có thể bỏ qua bước này không                      |
| `created_at`           | timestamp(6)     | NO       | NOW()    | Ngày tạo                                          |

#### 3.3.3. Cấu Trúc Bảng workflow_instance

| Tên Cột                | Kiểu Dữ Liệu | Nullable | Mặc Định | Mô Tả                                    |
| ---------------------- | ------------ | -------- | -------- | ---------------------------------------- |
| `id`                   | SERIAL       | NO       | AUTO     | Primary key                              |
| `workflow_template_id` | int          | NO       | -        | FK → workflow_template.id                |
| `current_step_order`   | int          | NO       | 1        | Bước hiện tại đang thực hiện             |
| `status`               | varchar(50)  | NO       | 'ACTIVE' | Trạng thái: ACTIVE, COMPLETED, CANCELLED |
| `related_entity_type`  | varchar(100) | YES      | NULL     | Loại entity liên quan                    |
| `related_entity_id`    | int          | YES      | NULL     | ID entity liên quan                      |
| `created_by_id`        | int          | NO       | -        | FK → user.id (người tạo workflow)        |
| `created_at`           | timestamp(6) | NO       | NOW()    | Ngày tạo                                 |
| `updated_at`           | timestamp(6) | NO       | NOW()    | Ngày cập nhật                            |
| `completed_at`         | timestamp(6) | YES      | NULL     | Ngày hoàn thành workflow                 |

### 3.4. Bảng Task Template

| Tên Cột                   | Kiểu Dữ Liệu     | Nullable | Mặc Định | Mô Tả                                                 |
| ------------------------- | ---------------- | -------- | -------- | ----------------------------------------------------- |
| `id`                      | SERIAL           | NO       | AUTO     | Primary key                                           |
| `name`                    | varchar(255)     | NO       | -        | Tên task template                                     |
| `description`             | text             | YES      | NULL     | Mô tả template                                        |
| `task_type`               | task_type_enum   | NO       | -        | Loại task                                             |
| `task_action`             | task_action_enum | NO       | -        | Action                                                |
| `default_role`            | varchar(100)     | YES      | NULL     | Role mặc định được gán task                           |
| `default_due_offset_days` | int              | YES      | NULL     | Số ngày cộng thêm để tính due date (tính từ ngày tạo) |
| `priority`                | varchar(50)      | YES      | NULL     | Ưu tiên (ví dụ: HIGH, MEDIUM, LOW)                    |
| `is_active`               | boolean          | NO       | true     | Trạng thái active                                     |
| `created_at`              | timestamp(6)     | NO       | NOW()    | Ngày tạo                                              |
| `updated_at`              | timestamp(6)     | NO       | NOW()    | Ngày cập nhật                                         |

### 3.5. Quan Hệ với Các Bảng Khác

**User Table**

- `task.assigned_to_id → user.id` (Many-to-One)
- `workflow_instance.created_by_id → user.id` (Many-to-One)
- Một user có thể có nhiều task
- Một task chỉ thuộc về một user

**Workflow Template**

- `workflow_step.workflow_template_id → workflow_template.id` (Many-to-One)
- `workflow_instance.workflow_template_id → workflow_template.id` (Many-to-One)
- Một template có nhiều steps
- Một template có thể có nhiều instances

**Workflow Instance**

- `task.workflow_instance_id → workflow_instance.id` (Many-to-One)
- Một workflow instance có nhiều tasks
- Một task chỉ thuộc về một workflow instance (hoặc không thuộc workflow nào)

**Task Dependencies**

- `task.depends_on_task_id → task.id` (Self-referencing)
- Task có thể phụ thuộc vào một task khác
- Khi task phụ thuộc hoàn thành, task hiện tại mới có thể bắt đầu

## 4. Luồng Hoạt Động

### 4.1. Luồng Tạo Task Tự Động

```
1. Sự kiện xảy ra trong hệ thống
   ↓
2. Module tương ứng gọi TaskService.createTaskByTypeAndAction()
   ↓
3. TaskService tìm người được gán task (ví dụ: tìm manager)
   ↓
4. Tạo task mới với status = PENDING
   ↓
5. Task được lưu vào database
```

### 4.2. Luồng Hoàn Thành Task Tự Động

```
1. Người dùng thực hiện hành động (ví dụ: approve leave request)
   ↓
2. Module tương ứng gọi TaskService.completeTaskByRelatedEntity()
   ↓
3. TaskService tìm task liên quan dựa trên relatedEntityType, relatedEntityId, action
   ↓
4. Cập nhật task: status = COMPLETED, completedAt = now()
   ↓
5. Task được lưu vào database
```

### 4.3. Luồng Xem Task Hôm Nay

```
1. User gọi API GET /tasks/today
   ↓
2. TaskService.getTodayTasks(userId)
   ↓
3. Lọc task theo:
   - assignedToId = userId
   - dueDate = hôm nay
   - status = PENDING
   - canStart = true (chỉ lấy task có thể bắt đầu)
   ↓
4. Trả về danh sách task
```

### 4.4. Luồng Tạo Task Theo Workflow

```
1. User tạo workflow instance từ template
   ↓
2. WorkflowService.createInstance(templateId, relatedEntity)
   ↓
3. Tạo workflow_instance với current_step_order = 1
   ↓
4. Tạo task đầu tiên từ step đầu tiên:
   - depends_on_task_id = NULL (không phụ thuộc)
   - can_start = true
   - workflow_instance_id = instance.id
   - workflow_step_order = 1
   ↓
5. Các task tiếp theo được tạo nhưng:
   - depends_on_task_id = task trước đó
   - can_start = false (chờ task trước hoàn thành)
```

### 4.5. Luồng Hoàn Thành Task Trong Workflow

```
1. User hoàn thành task hiện tại (status = COMPLETED)
   ↓
2. TaskService.updateTaskStatus() được gọi
   ↓
3. Kiểm tra xem task có thuộc workflow không:
   - Nếu có: Kiểm tra task phụ thuộc (depends_on_task_id)
   - Nếu task phụ thuộc đã completed: Set can_start = true cho task hiện tại
   ↓
4. Tìm task tiếp theo trong workflow:
   - workflow_instance_id = task.workflow_instance_id
   - workflow_step_order = task.workflow_step_order + 1
   ↓
5. Nếu có task tiếp theo:
   - Kiểm tra depends_on_task_id = task vừa completed
   - Set can_start = true cho task tiếp theo
   - Cập nhật workflow_instance.current_step_order
   ↓
6. Nếu không còn task nào:
   - Cập nhật workflow_instance.status = COMPLETED
   - Set workflow_instance.completed_at = now()
```

## 5. API Endpoints

### 5.1. Tạo Task 

**Endpoint:** `POST /tasks`

**Request Body:**

- `title`: string (required) - Tiêu đề của task
- `description`: string (optional) - Mô tả chi tiết
- `assignedToId`: number (required) - ID người được giao task
- `type`: TaskType (required) - Loại task
- `action`: TaskAction (required) - Hành động cần thực hiện
- `relatedEntityType`: string (optional) - Loại entity liên quan
- `relatedEntityId`: number (optional) - ID entity liên quan
- `dueDate`: string (optional) - Ngày hết hạn (YYYY-MM-DD)

**Response:** Task object với status = PENDING

### 5.2. Lấy Danh Sách Task

**Endpoint:** `GET /tasks`

**Query Parameters:**

- `status` (optional): Lọc theo trạng thái (PENDING, IN_PROGRESS, COMPLETED, CANCELLED)
- `date` (optional): Lọc theo ngày (YYYY-MM-DD)
- `all` (optional): true để lấy tất cả, false hoặc không có để lấy hôm nay

**Response:** Mảng các Task objects

### 5.3. Lấy Task Hôm Nay

**Endpoint:** `GET /tasks/today`

**Response:** Mảng các Task objects có dueDate = hôm nay và status = PENDING

### 5.4. Lấy Task Theo ID

**Endpoint:** `GET /tasks/:id`

**Response:** Task object

### 5.5. Cập Nhật Trạng Thái Task

**Endpoint:** `PUT /tasks/:id/status`

**Request Body:**

- `status`: TaskStatus (required) - Trạng thái mới

**Response:** Task object đã được cập nhật

**Lưu ý:** Nếu status = COMPLETED, hệ thống tự động set completedAt = now()

### 5.6. Xóa Task

**Endpoint:** `DELETE /tasks/:id`

**Response:** Message xác nhận đã xóa

### 5.7. Workflow Template Endpoints

#### 5.7.1. Tạo Workflow Template

**Endpoint:** `POST /workflows/templates`

**Request Body:**

- `name`: string (required) - Tên workflow template
- `description`: string (optional) - Mô tả
- `steps`: array (required) - Danh sách các bước
  - `stepOrder`: number - Thứ tự bước
  - `stepName`: string - Tên bước
  - `taskType`: TaskType - Loại task
  - `taskAction`: TaskAction - Action
  - `assignedToRole`: string (optional) - Role được gán
  - `isRequired`: boolean - Bắt buộc hay không
  - `canSkip`: boolean - Có thể bỏ qua không

**Response:** WorkflowTemplate object

#### 5.7.2. Lấy Danh Sách Workflow Templates

**Endpoint:** `GET /workflows/templates`

**Query Parameters:**

- `isActive` (optional): boolean - Lọc theo trạng thái active

**Response:** Mảng các WorkflowTemplate objects

#### 5.7.3. Lấy Workflow Template Theo ID

**Endpoint:** `GET /workflows/templates/:id`

**Response:** WorkflowTemplate object với danh sách steps

#### 5.7.4. Cập Nhật Workflow Template

**Endpoint:** `PUT /workflows/templates/:id`

**Request Body:** Tương tự như tạo template

**Response:** WorkflowTemplate object đã cập nhật

#### 5.7.5. Xóa Workflow Template

**Endpoint:** `DELETE /workflows/templates/:id`

**Response:** Message xác nhận đã xóa

### 5.8. Workflow Instance Endpoints

#### 5.8.1. Tạo Workflow Instance

**Endpoint:** `POST /workflows/instances`

**Request Body:**

- `workflowTemplateId`: number (required) - ID của template
- `relatedEntityType`: string (optional) - Loại entity liên quan
- `relatedEntityId`: number (optional) - ID entity liên quan
- `initialData`: object (optional) - Dữ liệu ban đầu cho các task

**Response:** WorkflowInstance object với task đầu tiên đã được tạo

#### 5.8.2. Lấy Danh Sách Workflow Instances

**Endpoint:** `GET /workflows/instances`

**Query Parameters:**

- `status` (optional): string - Lọc theo trạng thái
- `createdById` (optional): number - Lọc theo người tạo
- `relatedEntityType` (optional): string - Lọc theo entity type
- `relatedEntityId` (optional): number - Lọc theo entity ID

**Response:** Mảng các WorkflowInstance objects

#### 5.8.3. Lấy Workflow Instance Theo ID

**Endpoint:** `GET /workflows/instances/:id`

**Response:** WorkflowInstance object với danh sách tasks và trạng thái hiện tại

#### 5.8.4. Hủy Workflow Instance

**Endpoint:** `DELETE /workflows/instances/:id`

**Response:** Message xác nhận đã hủy

**Lưu ý:** Khi hủy workflow instance, tất cả tasks chưa completed sẽ bị cancelled

## 6. Tích Hợp Với Các Module Khác

### 6.1. Tích Hợp với Leave Module

#### 6.1.1. Khi Nhân Viên Tạo Leave Request

**Luồng xử lý:**

1. Nhân viên tạo leave request qua `POST /leaves`
2. `LeaveService.createRequest()` được gọi
3. Sau khi tạo leave request thành công:
   - Tìm manager của nhân viên (hoặc admin nếu không có manager)
   - Gọi `TaskService.createTaskByTypeAndAction()` với:
     - `type`: `LEAVE_APPROVAL`
     - `action`: `APPROVE_LEAVE_REQUEST`
     - `assignedToId`: ID của manager/admin
     - `relatedEntityType`: `"LeaveRequest"`
     - `relatedEntityId`: ID của leave request vừa tạo
     - `title`: "Phê duyệt đơn nghỉ phép của [Tên nhân viên]"
     - `description`: Mô tả chi tiết về đơn nghỉ phép
     - `dueDate`: Ngày bắt đầu nghỉ phép

#### 6.1.2. Khi Quản Lý Approve/Reject Leave Request

**Luồng xử lý:**

1. Quản lý approve/reject leave request qua `PUT /leaves/:id/status`
2. `LeaveService.updateStatus()` được gọi
3. Sau khi cập nhật status thành công:
   - Gọi `TaskService.completeTaskByRelatedEntity()` với:
     - `relatedEntityType`: `"LeaveRequest"`
     - `relatedEntityId`: ID của leave request
     - `action`: `APPROVE_LEAVE_REQUEST` hoặc `REJECT_LEAVE_REQUEST` tùy vào status

### 6.2. Mở Rộng cho Các Module Khác

Để tích hợp task với các module khác, cần:

1. **Thêm TaskType mới** trong enum TaskType (ví dụ: SALARY_APPROVAL, ATTENDANCE_REVIEW)
2. **Thêm TaskAction mới** trong enum TaskAction (ví dụ: APPROVE_SALARY, REVIEW_ATTENDANCE)
3. **Tích hợp vào Service của module tương ứng**:
   - Import `TaskModule` vào module
   - Inject `TaskService` vào service
   - Gọi `createTaskByTypeAndAction()` khi cần tạo task
   - Gọi `completeTaskByRelatedEntity()` khi hoàn thành hành động

## 7. Permissions và Security

### 7.1. Permissions

- **GET /tasks**: EMPLOYEE, MANAGER, ADMIN
- **GET /tasks/today**: EMPLOYEE, MANAGER, ADMIN
- **GET /tasks/:id**: EMPLOYEE, MANAGER, ADMIN
- **POST /tasks**:  MANAGER, ADMIN
- **PUT /tasks/:id/status**: EMPLOYEE, MANAGER, ADMIN (chỉ có thể cập nhật task của chính mình)
- **DELETE /tasks/:id**: EMPLOYEE, MANAGER, ADMIN (chỉ có thể xóa task của chính mình)

### 7.3. Workflow Permissions

- **POST /workflows/templates**: MANAGER, ADMIN
- **GET /workflows/templates**: MANAGER, ADMIN
- **PUT /workflows/templates/:id**: MANAGER, ADMIN
- **DELETE /workflows/templates/:id**: MANAGER, ADMIN
- **POST /workflows/instances**: MANAGER, ADMIN
- **GET /workflows/instances**: EMPLOYEE, MANAGER, ADMIN (chỉ xem instance của mình hoặc được assign)
- **DELETE /workflows/instances/:id**: EMPLOYEE, MANAGER, ADMIN (chỉ xóa instance của mình)
