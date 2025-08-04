# SaaS LMS Database Schema

This document describes the **full database schema** for a **multi-tenant LMS (Learning Management System)** with **SaaS-ready architecture**, including:

1. Authentication & Tenant Management
2. Courses & Lessons
3. Tests & Assessments
4. Programs & Orders
5. Payments & Subscriptions

Each table includes **column purposes** for easier developer understanding.

---

## 1️⃣ Authentication & Tenant Management

### **tenants**
| Column        | Type         | Purpose                               |
|---------------|-------------|---------------------------------------|
| id            | SERIAL PK   | Unique tenant/business ID              |
| name          | VARCHAR(255)| Tenant or business name                |
| logo_url      | TEXT        | Business logo                          |
| location      | TEXT        | Main business location                 |
| contact_email | VARCHAR(255)| Contact email for tenant               |
| contact_phone | VARCHAR(50) | Contact phone                          |
| is_active     | BOOLEAN     | Enable/disable tenant                  |
| created_at    | TIMESTAMP   | Creation timestamp                     |

---

### **tenant_branches**
| Column        | Type         | Purpose                               |
|---------------|-------------|---------------------------------------|
| id            | SERIAL PK   | Unique branch ID                       |
| tenant_id     | INT FK      | Parent tenant                          |
| name          | VARCHAR(255)| Branch name                            |
| location      | TEXT        | Branch location                        |
| contact_email | VARCHAR(255)| Branch email                           |
| contact_phone | VARCHAR(50) | Branch phone                           |
| is_active     | BOOLEAN     | Branch enabled/disabled                |
| created_at    | TIMESTAMP   | Creation timestamp                     |

---

### **users**
| Column        | Type         | Purpose                                  |
|---------------|-------------|------------------------------------------|
| id            | SERIAL PK   | Unique user ID                            |
| tenant_id     | INT FK      | Tenant ID for staff (NULL for admin/student) |
| email         | VARCHAR(255)| Unique login email                        |
| phone         | VARCHAR(50) | Optional contact number                   |
| password_hash | TEXT        | Hashed password                           |
| full_name     | VARCHAR(255)| User's full name                          |
| avatar_url    | TEXT        | Profile picture URL                       |
| user_type     | VARCHAR(20) | `student` / `tenant_staff` / `admin`      |
| is_active     | BOOLEAN     | User active status                        |
| last_login    | TIMESTAMP   | Last login timestamp                      |
| created_at    | TIMESTAMP   | Record creation                           |
| updated_at    | TIMESTAMP   | Record update                             |

---

### **staff_roles**
| Column           | Type         | Purpose                              |
|------------------|-------------|--------------------------------------|
| id               | SERIAL PK   | Role ID                               |
| tenant_id        | INT FK      | Tenant that owns this role            |
| name             | VARCHAR(100)| Role name (Tenant Admin, Instructor)  |
| is_branch_level  | BOOLEAN     | True if role applies to branch level  |
| created_at       | TIMESTAMP   | Record creation                       |

---

### **staff_permissions**
| Column         | Type         | Purpose                                 |
|----------------|-------------|-----------------------------------------|
| id             | SERIAL PK   | Permission ID                            |
| role_id        | INT FK      | Role to which permission applies         |
| permission_key | VARCHAR(100)| Key like `manage_courses`, `view_reports`|
| allowed        | BOOLEAN     | True = allowed                           |

---

### **tenant_staff**
| Column         | Type         | Purpose                                 |
|----------------|-------------|-----------------------------------------|
| id             | SERIAL PK   | Staff ID                                 |
| user_id        | INT FK      | Linked user                              |
| tenant_id      | INT FK      | Parent tenant                            |
| branch_id      | INT FK      | Optional branch for branch-level staff   |
| role_id        | INT FK      | Staff role                               |
| is_branch_admin| BOOLEAN     | True if branch admin                     |
| is_active      | BOOLEAN     | Staff active status                      |
| created_at     | TIMESTAMP   | Creation timestamp                       |

---

### **branch_student_enrollments**
| Column      | Type         | Purpose                                 |
|-------------|-------------|-----------------------------------------|
| id          | SERIAL PK   | Enrollment ID                            |
| branch_id   | INT FK      | Branch where student is enrolled         |
| student_id  | INT FK      | User with `user_type='student'`          |
| enrolled_at | TIMESTAMP   | Enrollment timestamp                     |

---

### **branch_course_access**
| Column      | Type         | Purpose                                 |
|-------------|-------------|-----------------------------------------|
| id          | SERIAL PK   | Record ID                                |
| branch_id   | INT FK      | Branch that can access this course       |
| course_id   | INT FK      | Course assigned to branch                |
| is_active   | BOOLEAN     | Access status                            |
| activated_at| TIMESTAMP   | Activation timestamp                     |

---

## 2️⃣ Courses & Lessons

### **courses**
| Column        | Type         | Purpose                               |
|---------------|-------------|---------------------------------------|
| id            | SERIAL PK   | Course ID                              |
| tenant_id     | INT FK      | Owner tenant                           |
| title         | VARCHAR(255)| Course title                           |
| description   | TEXT        | Detailed course description            |
| thumbnail_url | TEXT        | Course image                           |
| created_at    | TIMESTAMP   | Record creation                        |

---

### **modules**
| Column          | Type         | Purpose                               |
|-----------------|-------------|---------------------------------------|
| id              | SERIAL PK   | Module ID                              |
| course_id       | INT FK      | Parent course                          |
| title           | VARCHAR(255)| Module title                           |
| description     | TEXT        | Module description                     |
| module_order    | INT         | Order in course                        |
| duration_minutes| INT         | Estimated duration                     |
| created_at      | TIMESTAMP   | Record creation                        |

---

### **lessons**
| Column           | Type         | Purpose                               |
|------------------|-------------|---------------------------------------|
| id               | SERIAL PK   | Lesson ID                              |
| module_id        | INT FK      | Parent module                          |
| title            | VARCHAR(255)| Lesson title                           |
| content          | TEXT        | Optional text/HTML                     |
| video_type       | VARCHAR(20) | `url` / `upload`                       |
| video_url        | TEXT        | URL if type is `url`                   |
| video_file_path  | TEXT        | Path if uploaded                       |
| lesson_order     | INT         | Lesson order                           |
| duration_minutes | INT         | Duration estimate                      |
| created_at       | TIMESTAMP   | Record creation                        |

---

### **user_course_progress**
| Column             | Type         | Purpose                             |
|--------------------|-------------|-------------------------------------|
| id                 | SERIAL PK   | Progress ID                          |
| user_id            | INT FK      | Student                              |
| course_id          | INT FK      | Course                               |
| current_module_id  | INT FK      | Current module                       |
| current_lesson_id  | INT FK      | Current lesson                       |
| progress_percentage| REAL        | 0-100%                               |
| status             | VARCHAR(20) | in_progress / completed              |
| last_accessed      | TIMESTAMP   | Last access                          |
| created_at         | TIMESTAMP   | Record creation                      |
| updated_at         | TIMESTAMP   | Last update                          |

---

### **user_lesson_progress**
| Column       | Type         | Purpose                                |
|--------------|-------------|----------------------------------------|
| id           | SERIAL PK   | Record ID                               |
| user_id      | INT FK      | Student                                 |
| lesson_id    | INT FK      | Lesson                                  |
| started_at   | TIMESTAMP   | When started                            |
| completed_at | TIMESTAMP   | When completed                          |
| status       | VARCHAR(20) | in_progress / completed / skipped        |

---

## 3️⃣ Tests & Assessments

### **tests**
| Column       | Type         | Purpose                                 |
|--------------|-------------|-----------------------------------------|
| id           | SERIAL PK   | Test ID                                  |
| tenant_id    | INT FK      | Tenant owner                             |
| title        | VARCHAR(255)| Test name                                |
| description  | TEXT        | Description                              |
| max_score    | INT         | Total max score                          |
| scoring_type | VARCHAR(20) | fixed / per_question / per_section / conversion / custom |
| created_at   | TIMESTAMP   | Record creation                          |

---

### **test_sections**
| Column        | Type         | Purpose                                 |
|---------------|-------------|-----------------------------------------|
| id            | SERIAL PK   | Section ID                               |
| test_id       | INT FK      | Parent test                              |
| title         | VARCHAR(255)| Section name                             |
| max_score     | INT         | Optional if per_section scoring          |
| max_time_seconds | INT      | Optional time limit                      |
| ordering      | INT         | Display order                            |

---

### **test_questions**
| Column        | Type         | Purpose                                |
|---------------|-------------|----------------------------------------|
| id            | SERIAL PK   | Question ID                             |
| section_id    | INT FK      | Section                                 |
| question_text | TEXT        | Question text                           |
| type          | VARCHAR(20) | mcq / essay                             |
| score_value   | DECIMAL(5,2)| Score if per_question scoring           |
| ordering      | INT         | Question order                          |

---

*(Follow the same pattern for `test_options`, `test_attempts`, `test_attempt_answers`, `test_scores`.)*

---

## 4️⃣ Programs & Orders

### **programs**
| Column    | Type         | Purpose                          |
|-----------|-------------|----------------------------------|
| id        | SERIAL PK   | Program ID                       |
| name      | VARCHAR(100)| Program name                      |
| parent_id | INT FK      | Optional sub-program hierarchy    |

---

### **orders**
| Column       | Type         | Purpose                              |
|--------------|-------------|--------------------------------------|
| id           | SERIAL PK   | Order ID                              |
| user_id      | INT FK      | Student or buyer                      |
| type         | VARCHAR(20) | purchase / subscription               |
| status       | VARCHAR(20) | pending / paid / completed / cancelled|
| total_amount | DECIMAL(10,2)| Total amount                         |
| created_at   | TIMESTAMP   | Record creation                       |
| updated_at   | TIMESTAMP   | Record update                         |

---

### **payments**
| Column       | Type         | Purpose                              |
|--------------|-------------|--------------------------------------|
| id           | SERIAL PK   | Payment ID                            |
| order_id     | INT FK      | Linked order                          |
| method       | VARCHAR(20) | bank_transfer / gateway               |
| gateway_name | VARCHAR(50) | Gateway name (optional)               |
| status       | VARCHAR(20) | pending / confirmed / failed          |
| amount       | DECIMAL(10,2)| Payment amount                       |
| paid_at      | TIMESTAMP   | When paid                             |
| created_at   | TIMESTAMP   | Record creation                       |

---

# ✅ Notes

- **tenant_id** in each table ensures **multi-tenant isolation**.
- **branch_course_access** controls **branch-level course visibility**.
- **staff_roles + staff_permissions** enable **RBAC (Role-Based Access Control)**.
- **user_type** distinguishes **student / staff / admin**.

---

End of Schema
