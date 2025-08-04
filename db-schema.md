# SaaS LMS Database Schema

This document describes the **full database schema** for a **multi-tenant LMS (Learning Management System)** with **SaaS-ready architecture**, including:

1. Authentication & Tenant Management
2. Courses & Learning Content
3. Programs & Classes
4. Tests & Assessments
5. Enrollment, Orders & Payments
6. Notifications

---

## Table of Contents

- [1️⃣ Authentication & Tenant Management](#1-authentication--tenant-management)
- [2️⃣ Courses & Learning Content](#2-courses--learning-content)
- [3️⃣ Programs & Classes](#3-programs--classes)
- [4️⃣ Tests & Assessments](#4-tests--assessments)
- [5️⃣ Enrollment, Orders & Payments](#5-enrollment-orders--payments)
- [6️⃣ Notifications](#6-notifications)

---

## 1️⃣ Authentication & Tenant Management

### tenants

| Column        | Type          | Purpose                                    |
|---------------|--------------|--------------------------------------------|
| id (PK)       | SERIAL        | Unique tenant/business ID                  |
| name          | VARCHAR(255)  | Tenant/business name                       |
| logo_url      | TEXT          | Logo URL                                   |
| location      | TEXT          | Main business location                     |
| contact_email | VARCHAR(255)  | Contact email                              |
| contact_phone | VARCHAR(50)   | Contact phone number                       |
| is_active     | BOOLEAN       | Enable/disable tenant                      |
| created_at    | TIMESTAMP     | Creation timestamp                         |

---

### tenant_branches

| Column        | Type          | Purpose                                   |
|---------------|--------------|-------------------------------------------|
| id (PK)       | SERIAL        | Unique branch ID                          |
| tenant_id (FK)| INT           | Parent tenant                             |
| name          | VARCHAR(255)  | Branch name                               |
| location      | TEXT          | Branch location                           |
| contact_email | VARCHAR(255)  | Branch email                              |
| contact_phone | VARCHAR(50)   | Branch phone number                       |
| is_active     | BOOLEAN       | Enable/disable branch                     |
| created_at    | TIMESTAMP     | Creation timestamp                        |

---

### users

| Column        | Type           | Purpose                                       |
|---------------|---------------|-----------------------------------------------|
| id (PK)       | SERIAL         | Unique user ID                                |
| tenant_id (FK)| INT            | Tenant for staff (null for admin/student)     |
| email         | VARCHAR(255)   | Unique login email                            |
| phone         | VARCHAR(50)    | Contact number                                |
| password_hash | TEXT           | Password (hashed)                             |
| full_name     | VARCHAR(255)   | User's full name                              |
| avatar_url    | TEXT           | Profile picture                               |
| user_type     | VARCHAR(20)    | 'student', 'tenant_staff', 'admin'            |
| is_active     | BOOLEAN        | User active status                            |
| last_login    | TIMESTAMP      | Last login time                               |
| created_at    | TIMESTAMP      | Created                                       |
| updated_at    | TIMESTAMP      | Updated                                       |

---

### staff_roles

| Column           | Type           | Purpose                            |
|------------------|---------------|------------------------------------|
| id (PK)          | SERIAL         | Role ID                            |
| tenant_id (FK)   | INT            | Owner tenant                       |
| name             | VARCHAR(100)   | Role name (Admin, Instructor)      |
| is_branch_level  | BOOLEAN        | Role is for branch level?          |
| created_at       | TIMESTAMP      | Created                            |

---

### staff_permissions

| Column            | Type           | Purpose                                  |
|-------------------|---------------|------------------------------------------|
| id (PK)           | SERIAL         | Permission ID                            |
| role_id (FK)      | INT            | Associated role                          |
| permission_key    | VARCHAR(100)   | e.g. 'manage_courses', 'view_reports'    |
| allowed           | BOOLEAN        | Is allowed?                              |

---

### tenant_staff

| Column           | Type          | Purpose                                         |
|------------------|--------------|-------------------------------------------------|
| id (PK)          | SERIAL        | Staff assignment ID                             |
| user_id (FK)     | INT           | User ID                                         |
| tenant_id (FK)   | INT           | Tenant                                          |
| branch_id (FK)   | INT           | Branch (optional)                               |
| role_id (FK)     | INT           | Staff role                                      |
| is_branch_admin  | BOOLEAN       | Is branch admin?                                |
| is_active        | BOOLEAN       | Staff active                                    |
| created_at       | TIMESTAMP     | Assigned at                                     |

---

### branch_student_enrollments

| Column        | Type         | Purpose                        |
|---------------|-------------|--------------------------------|
| id (PK)       | SERIAL       | Enrollment record              |
| branch_id (FK)| INT          | Branch                         |
| student_id (FK)| INT         | Student user                   |
| enrolled_at   | TIMESTAMP    | Enrolled at                    |

---

### branch_course_access

| Column         | Type         | Purpose                                |
|----------------|-------------|----------------------------------------|
| id (PK)        | SERIAL       | Access record                          |
| branch_id (FK) | INT          | Branch                                 |
| course_id (FK) | INT          | Course                                 |
| is_active      | BOOLEAN      | Is access active                       |
| activated_at   | TIMESTAMP    | When activated                         |

---

### tenant_plans

| Column           | Type          | Purpose                                  |
|------------------|--------------|------------------------------------------|
| id (PK)          | SERIAL        | Tenant plan record                       |
| tenant_id (FK)   | INT           | Tenant                                   |
| plan_name        | VARCHAR(50)   | Plan name (Free/Pro/Enterprise)          |
| max_students     | INT           | Max students allowed                     |
| max_staff        | INT           | Max staff allowed                        |
| max_storage_mb   | INT           | Storage limit in MB                      |
| price_per_month  | DECIMAL(10,2) | Plan price per month                     |
| active           | BOOLEAN       | Is plan active                           |
| started_at       | TIMESTAMP     | Start date                               |
| expires_at       | TIMESTAMP     | End date                                 |
| created_at       | TIMESTAMP     | Created                                  |

---

### tenant_usage

| Column             | Type          | Purpose                                   |
|--------------------|--------------|-------------------------------------------|
| id (PK)            | SERIAL        | Usage record                              |
| tenant_id (FK)     | INT           | Tenant                                    |
| active_students    | INT           | Count of active students                  |
| total_enrollments  | INT           | Total enrollments this period             |
| total_storage_used_mb | INT        | Storage used (MB)                         |
| report_month       | DATE          | Month of report                           |
| created_at         | TIMESTAMP     | Created                                   |

---

### tenant_billing

| Column             | Type          | Purpose                                   |
|--------------------|--------------|-------------------------------------------|
| id (PK)            | SERIAL        | Billing statement                         |
| tenant_id (FK)     | INT           | Tenant                                    |
| billing_month      | DATE          | Billing period                            |
| subscription_fee   | DECIMAL(10,2) | Fixed fee for subscription                |
| usage_fee          | DECIMAL(10,2) | Variable usage/commission fee             |
| total_fee          | DECIMAL(10,2) | Total billed (auto-sum)                   |
| status             | VARCHAR(20)   | Billing status                            |
| created_at         | TIMESTAMP     | Created                                   |

---

## 2️⃣ Courses & Learning Content

### courses

| Column        | Type           | Purpose                                   |
|---------------|---------------|-------------------------------------------|
| id (PK)       | SERIAL         | Course ID                                 |
| tenant_id (FK)| INT            | Owner tenant                              |
| title         | VARCHAR(255)   | Course title                              |
| description   | TEXT           | Description                               |
| thumbnail_url | TEXT           | Course thumbnail                          |
| created_at    | TIMESTAMP      | Created                                   |

---

### modules

| Column         | Type           | Purpose                              |
|----------------|---------------|--------------------------------------|
| id (PK)        | SERIAL         | Module ID                            |
| course_id (FK) | INT            | Parent course                        |
| title          | VARCHAR(255)   | Module title                         |
| description    | TEXT           | Description                          |
| module_order   | INT            | Display order                        |
| duration_minutes | INT          | Estimated duration                   |
| created_at     | TIMESTAMP      | Created                              |

---

### lessons

| Column           | Type           | Purpose                                |
|------------------|---------------|----------------------------------------|
| id (PK)          | SERIAL         | Lesson ID                              |
| module_id (FK)   | INT            | Parent module                          |
| title            | VARCHAR(255)   | Lesson title                           |
| content          | TEXT           | Lesson content (HTML or plain text)    |
| video_type       | VARCHAR(20)    | 'url' or 'upload'                      |
| video_url        | TEXT           | Video URL                              |
| video_file_path  | TEXT           | Uploaded video path                    |
| pdf_file_path    | TEXT           | Uploaded PDF path                      |
| lesson_type      | VARCHAR(20)    | 'video', 'pdf', 'text', etc            |
| lesson_order     | INT            | Display order in module                |
| duration_minutes | INT            | Duration in minutes                    |
| created_at       | TIMESTAMP      | Created                                |

---

### user_courses

| Column         | Type           | Purpose                             |
|----------------|---------------|-------------------------------------|
| id (PK)        | SERIAL         | Enrollment record                   |
| user_id (FK)   | INT            | User                                |
| course_id (FK) | INT            | Course                              |
| source_type    | VARCHAR(20)    | 'subscription','voucher','manual',etc.|
| source_id      | INT            | Source reference                    |
| enrolled_at    | TIMESTAMP      | When enrolled                       |

---

### user_course_progress

| Column             | Type           | Purpose                               |
|--------------------|---------------|---------------------------------------|
| id (PK)            | SERIAL         | Progress record                       |
| user_id (FK)       | INT            | User                                  |
| course_id (FK)     | INT            | Course                                |
| current_module_id (FK) | INT        | Last module seen                      |
| current_lesson_id (FK) | INT        | Last lesson seen                      |
| progress_percentage | REAL          | 0–100% complete                       |
| status             | VARCHAR(20)    | 'in_progress', 'completed'            |
| last_accessed      | TIMESTAMP      | Last opened                           |
| created_at         | TIMESTAMP      | Created                               |
| updated_at         | TIMESTAMP      | Updated                               |

---

### user_lesson_progress

| Column         | Type         | Purpose                       |
|----------------|-------------|-------------------------------|
| id (PK)        | SERIAL       | Lesson progress record        |
| user_id (FK)   | INT          | User                          |
| lesson_id (FK) | INT          | Lesson                        |
| started_at     | TIMESTAMP    | When started                  |
| completed_at   | TIMESTAMP    | When completed                |
| status         | VARCHAR(20)  | 'in_progress', 'completed'    |

---

## 3️⃣ Programs & Classes

### programs

| Column        | Type           | Purpose                            |
|---------------|---------------|------------------------------------|
| id (PK)       | SERIAL         | Program bundle ID                  |
| tenant_id (FK)| INT            | Owner tenant                       |
| name          | VARCHAR(100)   | Program name                       |
| parent_id (FK)| INT            | Optional parent program            |

---

### program_courses

| Column         | Type         | Purpose                             |
|----------------|-------------|-------------------------------------|
| id (PK)        | SERIAL       | Mapping record                      |
| program_id (FK)| INT          | Program                             |
| course_id (FK) | INT          | Course                              |
| course_order   | INT          | Display order in program            |

---

### classes

| Column           | Type         | Purpose                                |
|------------------|-------------|----------------------------------------|
| id (PK)          | SERIAL       | Class ID                               |
| program_id (FK)  | INT          | Parent program                         |
| course_id (FK)   | INT          | Linked course                          |
| branch_id (FK)   | INT          | Branch for physical/branch class       |
| name             | VARCHAR(100) | Class name                             |
| level            | VARCHAR(50)  | Level (e.g. Beginner, Intermediate)    |
| max_students     | INT          | Max capacity                           |
| min_score        | FLOAT        | Pre-test min score                     |
| max_score        | FLOAT        | Pre-test max score                     |
| schedule_start   | TIMESTAMP    | Start date/time                        |
| schedule_end     | TIMESTAMP    | End date/time                          |
| day_of_week      | INT[]        | Days of week for recurring             |
| mode             | VARCHAR(20)  | 'online','offline','hybrid'            |
| location         | TEXT         | Address/location                       |
| meeting_link     | TEXT         | Video conference link                  |
| created_at       | TIMESTAMP    | Created                                |

---

### class_levels

| Column         | Type         | Purpose                          |
|----------------|-------------|----------------------------------|
| id (PK)        | SERIAL       | Class-level threshold            |
| class_id (FK)  | INT          | Class                            |
| min_score      | FLOAT        | Minimum score for assignment     |
| max_score      | FLOAT        | Maximum score for assignment     |

---

## 4️⃣ Tests & Assessments

### tests

| Column        | Type           | Purpose                                  |
|---------------|---------------|------------------------------------------|
| id (PK)       | SERIAL         | Test/exam ID                             |
| tenant_id (FK)| INT            | Owner tenant                             |
| title         | VARCHAR(255)   | Test title                               |
| description   | TEXT           | Test description                         |
| max_score     | INT            | Total max score                          |
| scoring_type  | VARCHAR(20)    | 'fixed','per_question','per_section',etc.|
| created_at    | TIMESTAMP      | Created                                  |

---

### test_sections

| Column         | Type           | Purpose                             |
|----------------|---------------|-------------------------------------|
| id (PK)        | SERIAL         | Section ID                          |
| test_id (FK)   | INT            | Parent test                         |
| title          | VARCHAR(255)   | Section title                       |
| instructions   | TEXT           | Instructions                        |
| max_score      | INT            | Max score for section               |
| max_time_seconds | INT          | Time limit for section              |
| ordering       | INT            | Section order                       |

---

### questions

| Column         | Type           | Purpose                                   |
|----------------|---------------|-------------------------------------------|
| id (PK)        | SERIAL         | Question ID                               |
| section_id (FK)| INT            | Section                                   |
| question_text  | TEXT           | Question prompt                           |
| type           | VARCHAR(20)    | 'mcq','essay','audio','video', etc.       |
| media_type     | VARCHAR(20)    | 'text','audio','video','image'            |
| media_url      | TEXT           | URL for prompt media                      |
| score_value    | DECIMAL(5,2)   | Score value                               |
| ordering       | INT            | Question order in section                 |

---

### test_options

| Column         | Type           | Purpose                      |
|----------------|---------------|------------------------------|
| id (PK)        | SERIAL         | Option ID                    |
| question_id (FK)| INT           | Question                     |
| option_text    | TEXT           | Answer text                  |
| is_correct     | BOOLEAN        | Is this option correct?      |

---

### test_attempts

| Column          | Type          | Purpose                              |
|-----------------|--------------|--------------------------------------|
| id (PK)         | SERIAL        | Attempt record                       |
| test_id (FK)    | INT           | Test                                 |
| student_id (FK) | INT           | Student                              |
| started_at      | TIMESTAMP     | Start time                           |
| completed_at    | TIMESTAMP     | Completion time                      |
| current_section_id (FK)| INT    | Section in progress                  |
| current_question_id (FK)| INT   | Question in progress                 |
| status          | VARCHAR(20)   | 'in_progress','completed','abandoned'|

---

### test_attempt_answers

| Column             | Type           | Purpose                                  |
|--------------------|---------------|------------------------------------------|
| id (PK)            | SERIAL         | Answer record                            |
| attempt_id (FK)    | INT            | Test attempt                             |
| question_id (FK)   | INT            | Question                                 |
| selected_option_id (FK)| INT        | MCQ option                               |
| text_answer        | TEXT           | Essay or text answer                     |
| answer_type        | VARCHAR(20)    | 'text','audio','video','image'           |
| answer_media_url   | TEXT           | URL for submitted media                  |
| answered_at        | TIMESTAMP      | Time answered                            |

---

### test_scores

| Column         | Type           | Purpose                              |
|----------------|---------------|--------------------------------------|
| id (PK)        | SERIAL         | Score summary                        |
| user_id (FK)   | INT            | Student                              |
| test_id (FK)   | INT            | Test                                 |
| total_score    | DECIMAL(5,2)   | Final score                          |
| status         | VARCHAR(20)    | 'in_progress','completed', etc.      |
| completed_at   | TIMESTAMP      | When finished                        |

---

### test_section_scores

| Column            | Type           | Purpose                     |
|-------------------|---------------|-----------------------------|
| id (PK)           | SERIAL         | Section score record        |
| test_score_id (FK)| INT            | Parent test score           |
| section_id (FK)   | INT            | Section                     |
| score             | DECIMAL(5,2)   | Score in section            |

---

### test_score_conversions

| Column         | Type         | Purpose                              |
|----------------|-------------|--------------------------------------|
| id (PK)        | SERIAL       | Score conversion record              |
| test_id (FK)   | INT          | Test                                 |
| section_id (FK)| INT          | Section                              |
| raw_score      | INT          | Raw score                            |
| scaled_score   | INT          | Scaled (converted) score             |

---

## 5️⃣ Enrollment, Orders & Payments

### orders

| Column        | Type          | Purpose                            |
|---------------|--------------|------------------------------------|
| id (PK)       | SERIAL        | Order ID                           |
| user_id (FK)  | INT           | Buyer                              |
| type          | VARCHAR(20)   | 'purchase','subscription'          |
| status        | VARCHAR(20)   | Order status                       |
| total_amount  | DECIMAL(10,2) | Order total                        |
| created_at    | TIMESTAMP     | Created                            |
| updated_at    | TIMESTAMP     | Updated                            |

---

### order_items

| Column         | Type          | Purpose                                |
|----------------|--------------|----------------------------------------|
| id (PK)        | SERIAL        | Line item                              |
| order_id (FK)  | INT           | Parent order                           |
| item_type      | VARCHAR(20)   | 'course','program','package'           |
| item_id        | INT           | Entity ID                              |
| price          | DECIMAL(10,2) | Price                                  |

---

### payments

| Column        | Type           | Purpose                                  |
|---------------|---------------|------------------------------------------|
| id (PK)       | SERIAL         | Payment record                           |
| order_id (FK) | INT            | Order                                    |
| method        | VARCHAR(20)    | 'bank_transfer','gateway'                |
| gateway_name  | VARCHAR(50)    | e.g., 'xendit','midtrans'                |
| status        | VARCHAR(20)    | 'pending','confirmed','failed'           |
| amount        | DECIMAL(10,2)  | Paid amount                              |
| paid_at       | TIMESTAMP      | Payment timestamp                        |
| created_at    | TIMESTAMP      | Created                                  |

---

### payment_gateway_transactions

| Column         | Type           | Purpose                               |
|----------------|---------------|---------------------------------------|
| id (PK)        | SERIAL         | Gateway transaction log               |
| payment_id (FK)| INT            | Payment                               |
| gateway_name   | VARCHAR(50)    | Gateway                               |
| transaction_id | VARCHAR(100)   | Transaction ID                        |
| status         | VARCHAR(50)    | Gateway status                        |
| raw_response   | JSONB          | Full response                         |
| created_at     | TIMESTAMP      | Created                               |

---

### payment_proofs

| Column        | Type          | Purpose                                  |
|---------------|--------------|------------------------------------------|
| id (PK)       | SERIAL        | Proof record                             |
| payment_id (FK)| INT          | Payment                                  |
| proof_url     | TEXT          | Uploaded proof (image, PDF)              |
| uploaded_at   | TIMESTAMP     | When uploaded                            |
| reviewed_by (FK)| INT         | Reviewer                                 |
| reviewed_at   | TIMESTAMP     | Review time                              |
| review_status | VARCHAR(20)   | 'approved','rejected'                    |

---

### payment_webhook_logs

| Column         | Type          | Purpose                              |
|----------------|--------------|--------------------------------------|
| id (PK)        | SERIAL        | Log record                           |
| gateway_name   | VARCHAR(50)   | Gateway                              |
| transaction_id | VARCHAR(100)  | Transaction (optional)               |
| event_type     | VARCHAR(50)   | e.g. 'invoice.paid'                  |
| payload        | JSONB         | Full webhook payload                 |
| status         | VARCHAR(20)   | Processing status                    |
| error_message  | TEXT          | Error info                           |
| created_at     | TIMESTAMP     | Created                              |

---

### subscriptions

| Column        | Type          | Purpose                                  |
|---------------|--------------|------------------------------------------|
| id (PK)       | SERIAL        | Subscription record                      |
| user_id (FK)  | INT           | Subscriber                               |
| type          | VARCHAR(20)   | 'single_course','program','package'      |
| reference_id  | INT           | Entity subscribed to                     |
| started_at    | TIMESTAMP     | When started                             |
| expires_at    | TIMESTAMP     | When expires                             |
| status        | VARCHAR(20)   | 'active','expired'                       |

---

## 6️⃣ Notifications

### notifications

| Column             | Type           | Purpose                                  |
|--------------------|---------------|------------------------------------------|
| id (PK)            | SERIAL         | Notification record                      |
| user_id (FK)       | INT            | Recipient user                           |
| tenant_id (FK)     | INT            | Tenant                                   |
| notification_type  | VARCHAR(50)    | Type (class_schedule, payment, etc.)     |
| channel            | VARCHAR(20)    | 'email','whatsapp','telegram','push'     |
| title              | VARCHAR(255)   | Notification title                       |
| message            | TEXT           | Content/body                             |
| sent_at            | TIMESTAMP      | When sent                                |
| status             | VARCHAR(20)    | 'pending','sent','failed'                |
