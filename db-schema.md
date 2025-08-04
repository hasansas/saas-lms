# SaaS LMS Database Schema (UUID, Multi-Tenant, RBAC, Flexible, 2025)

This document describes the **full database schema** for a modern, multi-tenant, SaaS Learning Management System (LMS) with:

- All **IDs as UUID**
- **RBAC** with tenant/branch-level roles & permissions
- **Unified tenant preferences** for branding, notifications, etc.
- **Flexible test assignment** via `source_type`/`reference_id`
- **Audit fields** on all tables

---

## Table of Contents

- [1️⃣ Authentication & Users](#1-authentication--users)
- [2️⃣ Tenant, Branch, & Staff Management](#2-tenant-branch--staff-management)
- [3️⃣ Courses & Learning Content](#3-courses--learning-content)
- [4️⃣ Programs, Classes, & Enrollment](#4-programs-classes--enrollment)
- [5️⃣ Tests & Assessments](#5-tests--assessments)
- [6️⃣ E-Commerce, Orders & Payments](#6-e-commerce-orders--payments)
- [7️⃣ Notifications](#7-notifications)
- [8️⃣ RBAC: Roles & Permissions](#8-rbac-roles--permissions)
- [9️⃣ Tenant Preferences](#9-tenant-preferences)
- [🔟 Gamification, Certificates, Bundles, Promotions](#10-gamification-certificates-bundles-promotions)
- [ℹ️ Audit Fields & Conventions](#️audit-fields--conventions)

---

## 1️⃣ Authentication & Users

### users

| Column              | Type    | Purpose                                                     |
| ------------------- | ------- | ----------------------------------------------------------- |
| id (PK)             | UUID    | Unique user ID (UUIDv4)                                     |
| name                | STRING  | User’s full name (required)                                 |
| email               | STRING  | Unique login email (required)                               |
| phone               | STRING  | Unique phone number                                         |
| phone_sanitized     | STRING  | Normalized phone                                            |
| password            | STRING  | Hashed password                                             |
| image               | STRING  | Profile avatar URL                                          |
| is_email_verified   | BOOLEAN | Email verified status                                       |
| is_phone_verified   | BOOLEAN | Phone verified status                                       |
| first_login         | DATE    | First login timestamp                                       |
| active              | BOOLEAN | Active user status                                          |
| ...audit fields     |         |                                                             |

---

### roles

| Column      | Type          | Purpose                                |
|-------------|--------------|----------------------------------------|
| id (PK)     | UUID          | Role ID                                |
| name        | VARCHAR(100)  | Role name (e.g. Admin, Tenant Staff)   |
| description | TEXT          | Role description                       |
| ...audit fields |           |                                        |

---

### user_roles

| Column      | Type         | Purpose                                            |
|-------------|-------------|----------------------------------------------------|
| id (PK)     | UUID         | User role mapping ID                               |
| user_id (FK)| UUID         | Linked user                                       |
| role_id (FK)| UUID         | Linked role                                       |
| assigned_at | TIMESTAMP    | When assigned                                     |
| ...audit fields |          |                                                   |

---

### user_verifications

| Column        | Type    | Purpose                                                        |
| ------------- | ------- | -------------------------------------------------------------- |
| id (PK)       | UUID    | Verification record ID                                         |
| user_id (FK)  | UUID    | User ID                                                        |
| type          | STRING  | 'email', 'phone', '2fa', etc.                                  |
| token         | STRING  | Hashed verification token                                      |
| expires_at    | DATE    | When the token expires                                         |
| verified_at   | DATE    | When verification completed                                    |
| attempts      | INTEGER | Failed attempts (rate limiting)                                |
| ...audit fields |       |                                                                |

---

## 2️⃣ Tenant, Branch & Staff Management

### tenants

| Column        | Type           | Purpose                                    |
|---------------|---------------|--------------------------------------------|
| id (PK)       | UUID           | Tenant/business ID                         |
| user_id (FK)  | UUID           | Owner user (super-admin)                   |
| name          | VARCHAR(255)   | Tenant/business name                       |
| logo_url      | TEXT           | Logo URL                                   |
| location      | TEXT           | Main business location                     |
| contact_email | VARCHAR(255)   | Contact email                              |
| contact_phone | VARCHAR(50)    | Contact phone                              |
| is_active     | BOOLEAN        | Active status                              |
| ...audit fields |              |                                            |

---

### tenant_preferences

| Column        | Type     | Purpose                                                        |
|---------------|----------|----------------------------------------------------------------|
| id (PK)       | UUID     | Preferences record ID                                          |
| tenant_id (FK)| UUID     | Tenant                                                         |
| preferences   | JSONB    | Theme, notification, branding, other tenant settings           |
| ...audit fields |        |                                                                |

---

### tenant_branches

| Column        | Type          | Purpose                                   |
|---------------|--------------|-------------------------------------------|
| id (PK)       | UUID          | Branch ID                                 |
| tenant_id (FK)| UUID          | Parent tenant                             |
| name          | VARCHAR(255)  | Branch name                               |
| location      | TEXT          | Branch location                           |
| contact_email | VARCHAR(255)  | Branch email                              |
| contact_phone | VARCHAR(50)   | Branch phone                              |
| is_active     | BOOLEAN       | Active status                             |
| ...audit fields |             |                                           |

---

### tenant_staff

| Column           | Type          | Purpose                                         |
|------------------|--------------|-------------------------------------------------|
| id (PK)          | UUID          | Staff assignment ID                             |
| user_id (FK)     | UUID          | User ID                                         |
| tenant_id (FK)   | UUID          | Tenant                                          |
| branch_id (FK)   | UUID, null    | Branch (optional)                               |
| staff_role_id (FK)| UUID, null   | Staff role (see RBAC below)                     |
| is_branch_admin  | BOOLEAN       | Is branch admin?                                |
| is_active        | BOOLEAN       | Staff active                                    |
| ...audit fields  |              |                                                 |

---

### branch_student_enrollments

| Column        | Type         | Purpose                        |
|---------------|-------------|--------------------------------|
| id (PK)       | UUID         | Enrollment record              |
| branch_id (FK)| UUID         | Branch                         |
| student_id (FK)| UUID        | Student user                   |
| enrolled_at   | TIMESTAMP    | Enrolled at                    |
| ...audit fields |            |                                |

---

### branch_course_access

| Column         | Type         | Purpose                                |
|----------------|-------------|----------------------------------------|
| id (PK)        | UUID         | Access record                          |
| branch_id (FK) | UUID         | Branch                                 |
| course_id (FK) | UUID         | Course                                 |
| is_active      | BOOLEAN      | Is access active                       |
| activated_at   | TIMESTAMP    | When activated                         |
| ...audit fields |             |                                        |

---

## 3️⃣ Courses & Learning Content

### courses

| Column         | Type           | Purpose                                       |
|----------------|---------------|-----------------------------------------------|
| id (PK)        | UUID           | Course ID                                     |
| tenant_id (FK) | UUID           | Owner tenant                                  |
| title          | VARCHAR(255)   | Course title                                  |
| description    | TEXT           | Description                                   |
| content        | TEXT           | Long form content, syllabus, or HTML/Markdown |
| price          | DECIMAL(10,2)  | Displayed (original) price                    |
| actual_price   | DECIMAL(10,2)  | Current/discount price                        |
| duration_days  | INT, nullable  | Days of access (null or 0 = lifetime)         |
| image_url      | TEXT           | Course thumbnail                              |
| status         | VARCHAR(20)    | 'active', 'inactive', 'draft', etc.           |
| tags           | TEXT[], JSONB  | (Optional) Tags for search/filter             |
| ...audit fields|                |                                               |


---

### modules

| Column         | Type           | Purpose                              |
|----------------|---------------|--------------------------------------|
| id (PK)        | UUID           | Module ID                            |
| course_id (FK) | UUID           | Parent course                        |
| title          | VARCHAR(255)   | Module title                         |
| description    | TEXT           | Description                          |
| module_order   | INT            | Display order                        |
| duration_minutes | INT          | Estimated duration                   |
| ...audit fields |               |                                      |

---

### lessons

| Column           | Type           | Purpose                                |
|------------------|---------------|----------------------------------------|
| id (PK)          | UUID           | Lesson ID                              |
| module_id (FK)   | UUID           | Parent module                          |
| title            | VARCHAR(255)   | Lesson title                           |
| content          | TEXT           | HTML or plain text content             |
| video_type       | VARCHAR(20)    | 'url' or 'upload'                      |
| video_url        | TEXT           | Video URL                              |
| video_file_path  | TEXT           | Uploaded video path                    |
| pdf_file_path    | TEXT           | Uploaded PDF path                      |
| lesson_type      | VARCHAR(20)    | 'video', 'pdf', 'text', etc            |
| lesson_order     | INT            | Display order                          |
| duration_minutes | INT            | Duration in minutes                    |
| ...audit fields  |                |                                        |

---

### user_courses

| Column         | Type           | Purpose                             |
|----------------|---------------|-------------------------------------|
| id (PK)        | UUID           | Enrollment record                   |
| user_id (FK)   | UUID           | User                                |
| course_id (FK) | UUID           | Course                              |
| source_type    | VARCHAR(20)    | 'subscription','voucher','manual',etc.|
| source_id      | UUID           | Source reference                    |
| enrolled_at    | TIMESTAMP      | When enrolled                       |
| ...audit fields |               |                                     |

---

### user_course_progress

| Column             | Type           | Purpose                               |
|--------------------|---------------|---------------------------------------|
| id (PK)            | UUID           | Progress record                       |
| user_id (FK)       | UUID           | User                                  |
| course_id (FK)     | UUID           | Course                                |
| current_module_id (FK) | UUID       | Last module seen                      |
| current_lesson_id (FK) | UUID       | Last lesson seen                      |
| progress_percentage | REAL          | 0–100% complete                       |
| status             | VARCHAR(20)    | 'in_progress', 'completed'            |
| last_accessed      | TIMESTAMP      | Last opened                           |
| ...audit fields    |                |                                       |

---

### user_lesson_progress

| Column         | Type         | Purpose                       |
|----------------|-------------|-------------------------------|
| id (PK)        | UUID         | Lesson progress record        |
| user_id (FK)   | UUID         | User                          |
| lesson_id (FK) | UUID         | Lesson                        |
| started_at     | TIMESTAMP    | When started                  |
| completed_at   | TIMESTAMP    | When completed                |
| status         | VARCHAR(20)  | 'in_progress', 'completed'    |
| ...audit fields|              |                               |

---

## 4️⃣ Programs, Classes & Enrollment

### programs

| Column         | Type           | Purpose                                               |
|----------------|---------------|-------------------------------------------------------|
| id (PK)        | UUID           | Program ID                                            |
| tenant_id (FK) | UUID           | Owner tenant                                          |
| name           | VARCHAR(100)   | Program name                                          |
| description    | TEXT           | Short/long marketing description                      |
| content        | TEXT           | Detailed content (HTML/Markdown, e.g., outcomes, FAQ) |
| suggestion     | TEXT           | Suggestion or recommendation (personalized message, upsell, etc.) |
| price          | DECIMAL(10,2)  | Displayed (original) price                            |
| actual_price   | DECIMAL(10,2)  | Current price after discount (if any)                 |
| duration_days  | INT            | Access duration (in days)                             |
| image_url      | TEXT           | Marketing image/badge                                 |
| parent_id (FK) | UUID, null     | Optional parent program                               |
| status         | VARCHAR(20)    | 'active','inactive','draft', etc.                     |
| ...audit fields|                |                                                      |

---

### class_courses

| Column         | Type         | Purpose                               |
|----------------|-------------|---------------------------------------|
| id (PK)        | UUID         | Mapping ID                            |
| class_id (FK)  | UUID         | Class                                 |
| course_id (FK) | UUID         | Course                                |
| course_order   | INT          | Order in class (optional)             |
| ...audit fields|              |                                       |

---

### classes

| Column           | Type         | Purpose                                |
|------------------|-------------|----------------------------------------|
| id (PK)          | UUID         | Class ID                               |
| program_id (FK)  | UUID         | Parent program                         |
| course_id (FK)   | UUID         | Linked course                          |
| branch_id (FK)   | UUID         | Branch for physical/branch class       |
| name             | VARCHAR(100) | Class name                             |
| level            | VARCHAR(50)  | Level (e.g. Beginner, Intermediate)    |
| max_students     | INT          | Max capacity                           |
| current_capacity | INT          | Current enrolled students              |
| min_score        | FLOAT        | Pre-test min score                     |
| max_score        | FLOAT        | Pre-test max score                     |
| schedule_start   | TIMESTAMP    | Start date/time                        |
| schedule_end     | TIMESTAMP    | End date/time                          |
| day_of_week      | INT[]        | Days of week for recurring             |
| mode             | VARCHAR(20)  | 'online','offline','hybrid'            |
| location         | TEXT         | Address/location                       |
| meeting_link     | TEXT         | Video conference link                  |
| status           | VARCHAR(20)  | 'scheduled','active','completed',etc.  |
| ...audit fields  |              |                                        |

---

### class_levels

| Column         | Type         | Purpose                          |
|----------------|-------------|----------------------------------|
| id (PK)        | UUID         | Class-level threshold            |
| class_id (FK)  | UUID         | Class                            |
| min_score      | FLOAT        | Minimum score for assignment     |
| max_score      | FLOAT        | Maximum score for assignment     |
| ...audit fields|              |                                  |

---

## 5️⃣ Tests & Assessments

### tests

| Column        | Type           | Purpose                                       |
|---------------|---------------|-----------------------------------------------|
| id (PK)       | UUID           | Test/exam ID                                  |
| tenant_id (FK)| UUID           | Owner tenant                                  |
| title         | VARCHAR(255)   | Test title                                    |
| description   | TEXT           | Test description                              |
| max_score     | INT            | Total max score                               |
| scoring_type  | VARCHAR(20)    | 'fixed','per_question','per_section',etc.     |
| scoring_config| JSONB, null    | Custom scoring rules (optional)               |
| source_type   | VARCHAR(20)    | 'course','program','module','lesson',etc (optional) |
| reference_id  | UUID, null     | ID of related entity if `source_type` is set  |
| test_type     | VARCHAR(20)    | quiz, pre-test, post-test, placement, etc     |
| show_section_header| BOOLEAN   | For frontend display                          |
| ...audit fields |              |                                               |

> **Note:** If `source_type` is set, `reference_id` must point to a valid entity (application validation).

---

### test_sections

| Column         | Type           | Purpose                             |
|----------------|---------------|-------------------------------------|
| id (PK)        | UUID           | Section ID                          |
| test_id (FK)   | UUID           | Parent test                         |
| title          | VARCHAR(255)   | Section title                       |
| instructions   | TEXT           | Instructions                        |
| max_score      | INT            | Max score for section               |
| max_time_seconds | INT          | Time limit for section              |
| ordering       | INT            | Section order                       |
| ...audit fields|                |                                     |

---

### questions

| Column         | Type           | Purpose                                   |
|----------------|---------------|-------------------------------------------|
| id (PK)        | UUID           | Question ID                               |
| section_id (FK)| UUID           | Section                                   |
| question_text  | TEXT           | Prompt                                    |
| type           | VARCHAR(20)    | 'mcq','essay','audio','video', etc.       |
| media_type     | VARCHAR(20)    | 'text','audio','video','image'            |
| media_url      | TEXT           | URL for prompt media                      |
| score_value    | DECIMAL(5,2)   | Score value                               |
| ordering       | INT            | Question order in section                 |
| ...audit fields|                |                                           |

---

### test_options

| Column         | Type           | Purpose                      |
|----------------|---------------|------------------------------|
| id (PK)        | UUID           | Option ID                    |
| question_id (FK)| UUID          | Question                     |
| option_text    | TEXT           | Answer text                  |
| is_correct     | BOOLEAN        | Is this option correct?      |
| ...audit fields|                |                              |

---

### test_attempts

| Column          | Type          | Purpose                              |
|-----------------|--------------|--------------------------------------|
| id (PK)         | UUID          | Attempt record                       |
| test_id (FK)    | UUID          | Test                                 |
| student_id (FK) | UUID          | Student                              |
| started_at      | TIMESTAMP     | Start time                           |
| completed_at    | TIMESTAMP     | Completion time                      |
| current_section_id (FK)| UUID   | Section in progress                  |
| current_question_id (FK)| UUID  | Question in progress                 |
| status          | VARCHAR(20)   | 'in_progress','completed','abandoned'|
| ...audit fields |               |                                      |

---

### test_attempt_answers

| Column             | Type           | Purpose                                  |
|--------------------|---------------|------------------------------------------|
| id (PK)            | UUID           | Answer record                            |
| attempt_id (FK)    | UUID           | Test attempt                             |
| question_id (FK)   | UUID           | Question                                 |
| selected_option_id (FK)| UUID       | MCQ option                               |
| text_answer        | TEXT           | Essay/text answer                        |
| answer_type        | VARCHAR(20)    | 'text','audio','video','image'           |
| answer_media_url   | TEXT           | URL for submitted media                  |
| answered_at        | TIMESTAMP      | Time answered                            |
| ...audit fields    |                |                                          |

---

### test_scores

| Column         | Type           | Purpose                              |
|----------------|---------------|--------------------------------------|
| id (PK)        | UUID           | Score summary                        |
| user_id (FK)   | UUID           | Student                              |
| test_id (FK)   | UUID           | Test                                 |
| total_score    | DECIMAL(5,2)   | Final score                          |
| status         | VARCHAR(20)    | 'in_progress','completed', etc.      |
| completed_at   | TIMESTAMP      | When finished                        |
| ...audit fields|                |                                      |

---

### test_section_scores

| Column            | Type           | Purpose                     |
|-------------------|---------------|-----------------------------|
| id (PK)           | UUID           | Section score record        |
| test_score_id (FK)| UUID           | Parent test score           |
| section_id (FK)   | UUID           | Section                     |
| score             | DECIMAL(5,2)   | Score in section            |
| ...audit fields   |                |                             |

---

### test_score_conversions

| Column         | Type         | Purpose                              |
|----------------|-------------|--------------------------------------|
| id (PK)        | UUID         | Score conversion record              |
| test_id (FK)   | UUID         | Test                                 |
| section_id (FK)| UUID         | Section                              |
| raw_score      | INT          | Raw score                            |
| scaled_score   | INT          | Scaled (converted) score             |
| ...audit fields|              |                                      |

---

## 6️⃣ E-Commerce, Orders & Payments

### orders

| Column        | Type          | Purpose                            |
|---------------|--------------|------------------------------------|
| id (PK)       | UUID          | Order ID                           |
| user_id (FK)  | UUID          | Buyer                              |
| type          | VARCHAR(20)   | 'purchase','subscription'          |
| status        | VARCHAR(20)   | Order status                       |
| total_amount  | DECIMAL(10,2) | Order total                        |
| ...audit fields|              |                                    |

---

### order_items

| Column         | Type          | Purpose                                |
|----------------|--------------|----------------------------------------|
| id (PK)        | UUID          | Line item                              |
| order_id (FK)  | UUID          | Parent order                           |
| item_type      | VARCHAR(20)   | 'course','program','package'           |
| item_id        | UUID          | Entity ID                              |
| price          | DECIMAL(10,2) | Price                                  |
| ...audit fields|               |                                        |

---

### payments

| Column        | Type           | Purpose                                  |
|---------------|---------------|------------------------------------------|
| id (PK)       | UUID           | Payment record                           |
| order_id (FK) | UUID           | Order                                    |
| method        | VARCHAR(20)    | 'bank_transfer','gateway'                |
| gateway_name  | VARCHAR(50)    | e.g., 'xendit','midtrans'                |
| status        | VARCHAR(20)    | 'pending','confirmed','failed'           |
| amount        | DECIMAL(10,2)  | Paid amount                              |
| paid_at       | TIMESTAMP      | Payment timestamp                        |
| ...audit fields|               |                                          |

---

### payment_gateway_transactions

| Column         | Type           | Purpose                               |
|----------------|---------------|---------------------------------------|
| id (PK)        | UUID           | Gateway transaction log               |
| payment_id (FK)| UUID           | Payment                               |
| gateway_name   | VARCHAR(50)    | Gateway                               |
| transaction_id | VARCHAR(100)   | Transaction ID                        |
| status         | VARCHAR(50)    | Gateway status                        |
| raw_response   | JSONB          | Full response                         |
| ...audit fields|                |                                       |

---

### payment_proofs

| Column        | Type          | Purpose                                  |
|---------------|--------------|------------------------------------------|
| id (PK)       | UUID          | Proof record                             |
| payment_id (FK)| UUID         | Payment                                  |
| proof_url     | TEXT          | Uploaded proof (image, PDF)              |
| uploaded_at   | TIMESTAMP     | When uploaded                            |
| reviewed_by (FK)| UUID        | Reviewer                                 |
| reviewed_at   | TIMESTAMP     | Review time                              |
| review_status | VARCHAR(20)   | 'approved','rejected'                    |
| ...audit fields|              |                                          |

---

### payment_webhook_logs

| Column         | Type          | Purpose                              |
|----------------|--------------|--------------------------------------|
| id (PK)        | UUID          | Log record                           |
| gateway_name   | VARCHAR(50)   | Gateway                              |
| transaction_id | VARCHAR(100)  | Transaction (optional)               |
| event_type     | VARCHAR(50)   | e.g. 'invoice.paid'                  |
| payload        | JSONB         | Full webhook payload                 |
| status         | VARCHAR(20)   | Processing status                    |
| error_message  | TEXT          | Error info                           |
| ...audit fields|               |                                      |

---

### packages

| Column         | Type           | Purpose                                            |
|----------------|---------------|----------------------------------------------------|
| id (PK)        | UUID           | Package ID                                         |
| tenant_id (FK) | UUID           | Owner tenant                                       |
| name           | VARCHAR(100)   | Package name                                       |
| description    | TEXT           | Marketing description                              |
| content        | TEXT           | Detailed info, FAQ, or HTML/Markdown               |
| price          | DECIMAL(10,2)  | Recurring/subscription price (monthly/annual/etc.) |
| actual_price   | DECIMAL(10,2)  | Discounted/current price                           |
| duration_days  | INT            | Billing cycle length (30, 365, etc)                |
| image_url      | TEXT           | Promo image or badge                               |
| status         | VARCHAR(20)    | 'active','inactive','draft', etc.                  |
| ...audit fields|                |                                                    |

---

### package_courses

| Column         | Type         | Purpose                     |
|----------------|-------------|-----------------------------|
| id (PK)        | UUID         | Mapping ID                  |
| package_id (FK)| UUID         | Parent package              |
| course_id (FK) | UUID         | Included course             |
| course_order   | INT          | (optional)                  |
| ...audit fields|              |                             |

---

### subscriptions

| Column        | Type          | Purpose                                        |
|---------------|--------------|------------------------------------------------|
| id (PK)       | UUID          | Subscription record                            |
| user_id (FK)  | UUID          | Subscriber                                     |
| type          | VARCHAR(20)   | 'single_course','program','package'            |
| reference_id  | UUID          | ID of course/program/package                   |
| started_at    | TIMESTAMP     | When started                                   |
| expires_at    | TIMESTAMP     | When expires                                   |
| status        | VARCHAR(20)   | 'active','expired'                             |
| ...audit fields|              |                                                |

---

### subscription_plans

| Column        | Type          | Purpose                                |
|---------------|--------------|----------------------------------------|
| id (PK)       | UUID          | Plan ID                                |
| name          | VARCHAR(100)  | Plan name                              |
| monthly_price | DECIMAL(10,2) | Price                                  |
| max_students  | INT           | Plan quota                             |
| max_staff     | INT           | Plan quota                             |
| max_storage_gb| INT           | Storage GB                             |
| features      | JSONB         | Feature flags/config                   |
| description   | TEXT          | Plan description                       |
| ...audit fields|              |                                        |

---

### tenant_subscriptions

| Column        | Type          | Purpose                                    |
|---------------|--------------|--------------------------------------------|
| id (PK)       | UUID          | Record ID                                  |
| tenant_id (FK)| UUID          | Tenant                                     |
| plan_id (FK)  | UUID          | Plan                                       |
| started_at    | TIMESTAMP     | Subscription start                         |
| expires_at    | TIMESTAMP     | Subscription expiry                        |
| status        | VARCHAR(20)   | active, cancelled, etc.                    |
| renewal_type  | VARCHAR(20)   | monthly, annual, custom                    |
| ...audit fields|              |                                            |

---

### commission_fees

| Column        | Type          | Purpose                                    |
|---------------|--------------|--------------------------------------------|
| id (PK)       | UUID          | Commission record                          |
| tenant_id (FK)| UUID          | Tenant                                     |
| order_id (FK) | UUID          | Order                                      |
| enrollment_id (FK)| UUID      | Enrollment (if per enrollment)             |
| commission_type | VARCHAR(20) | percent, fixed                             |
| commission_value| DECIMAL(10,2)| Value                                     |
| applied_at    | TIMESTAMP     | When applied                               |
| ...audit fields|              |                                            |

---

## 7️⃣ Notifications

### notifications

| Column             | Type           | Purpose                                  |
|--------------------|---------------|------------------------------------------|
| id (PK)            | UUID           | Notification record                      |
| user_id (FK)       | UUID           | Recipient user                           |
| tenant_id (FK)     | UUID           | Tenant                                   |
| notification_type  | VARCHAR(50)    | Type (class_schedule, payment, etc.)     |
| channel            | VARCHAR(20)    | 'email','whatsapp','telegram','push'     |
| title              | VARCHAR(255)   | Notification title                       |
| message            | TEXT           | Content/body                             |
| sent_at            | TIMESTAMP      | When sent                                |
| status             | VARCHAR(20)    | 'pending','sent','failed'                |
| ...audit fields    |                |                                          |

---

## 8️⃣ RBAC: Roles & Permissions

### staff_roles

| Column          | Type        | Purpose                      |
|-----------------|------------|------------------------------|
| id (PK)         | UUID        | Role ID                      |
| tenant_id (FK)  | UUID        | Tenant                       |
| branch_id (FK)  | UUID, null  | Branch, if branch-specific   |
| name            | VARCHAR(100)| e.g. Instructor, Branch Admin|
| description     | TEXT        | Description                  |
| ...audit fields |             |                              |

---

### staff_permissions

| Column              | Type        | Purpose                    |
|---------------------|------------|----------------------------|
| id (PK)             | UUID        | Permission record          |
| staff_role_id (FK)  | UUID        | Staff role                 |
| permission_key      | VARCHAR(100)| e.g. manage_courses        |
| allowed             | BOOLEAN     | Allowed or not             |
| ...audit fields     |             |                            |

---

## 9️⃣ Tenant Preferences

- See `tenant_preferences` above. Store all tenant-level branding, theme, and notification preferences here (as JSONB).

---

## 🔟 Gamification, Certificates, Bundles, Promotions

- Tables such as `certificates`, `badges`, `user_badges`, `points`, `user_points`, `bundles`, `bundle_courses`, `promotions` as per your feature roadmap and prior suggestions.

---

## ℹ️ Audit Fields & Conventions

**All tables include**:

- `created_at` (TIMESTAMP)
- `created_by` (UUID, nullable)
- `updated_at` (TIMESTAMP)
- `updated_by` (UUID, nullable)
- `deleted_at` (TIMESTAMP, nullable)
- `deleted_by` (UUID, nullable)

---

## Notes

- All `id`, `(FK)` columns are **UUID**.
- Use **application validation** for logic like `tests.source_type`/`reference_id`.
- You may add additional tables for chat/discussion, advanced analytics, or other modules as needed.
- **External analytics (e.g., Google Analytics)** recommended for event and engagement tracking.

---

**End of Schema**
