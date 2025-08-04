# 📘 SaaS LMS Application Documentation (Full)

---

## 1️⃣ Application Overview

The **SaaS LMS** is a **multi-tenant, multi-branch learning management system** designed for:

- **Training organizations**, **corporate academies**, and **online course providers**  
- **Subscription-based SaaS model** with **tenant isolation**  
- **Support for modular courses, programs with classes, and auto-assignment via pre-tests**  
- **Multi-channel notifications** for class schedules and updates  

**Key Highlights:**

- **Multi-tenant & multi-branch architecture**  
- **RBAC (Role-Based Access Control)** for staff and admins  
- **Branch-specific enrollments and course visibility**  
- **Program/Class system with max capacity and auto-assignment**  
- **Multi-channel notifications (Email, WhatsApp, Telegram)**  

---

## 2️⃣ Feature Modules

### 2.1 Tenant & Branch Management

- Multi-tenant with **logo, contact info, and activation status**  
- **Branch management** for localized operations  
- **Staff management** with RBAC:
  - `tenant_staff` roles (e.g., Instructor, Branch Admin)
  - Assign **branch-level or tenant-level permissions**
- **Student enrollment per branch** via `branch_student_enrollments`  

---

### 2.2 Authentication & Users

- **Secure authentication** with password hashing  
- **User types**:
  - `admin` → Global SaaS super admin
  - `tenant_staff` → Instructor / Manager
  - `student` → Learner
- **Profiles** with avatar, last login, and role mapping  
- **RBAC enforcement** with `staff_roles` & `staff_permissions`  

---

### 2.3 Courses & Learning Content

**Structure:**  
**Course → Modules → Lessons**

**Lesson Content Supports:**

- Text/HTML content  
- Video (`url` or `upload`)  
- PDF (`pdf_file_path`) for downloadable material  

**Features:**

- Modular learning paths with **progress tracking**  
- **Pre-test / Post-test** optional per **course or program**  
- **Completion triggers** when all lessons completed  

### 2.4 Tests & Assessments

- **Multi-section tests** with MCQ and Essay support  
- **Flexible scoring** (per question, per section, fixed, or custom)  
- **Pre-test & Post-test** support:
  - **Pre-test** for **program placement / class auto-assignment**  
  - **Post-test** for **final evaluation / certification**  
- **Test attempt tracking**, **auto-grading** for MCQs, and **manual grading** for essays  

---

### 2.5 Programs, Classes & E-Commerce

#### Programs

- Bundle **multiple courses**  
- Purchase via **one-time payment** or **subscription**  
- Optional **pre-test / post-test** configuration per tenant  

#### Classes

- **Programs contain many classes**  
- **Each class has max student capacity**  
- **Students auto-assigned based on pre-test score**  
  - Score thresholds determine **level/class** (Beginner/Intermediate/Advanced)  
- **Dynamic class creation** if all existing classes for a level are full  

#### Class Schedule

- **Date & Time**  
- **Day of Week** (for recurring classes)  
- **Mode**:
  - **Physical** → Address/Room  
  - **Online** → Zoom/Google Meet link  

#### Enrollment & Notification

- After **successful enrollment and class assignment**, student receives:
  - **Email confirmation** with schedule and location/Zoom link  
  - **WhatsApp / Telegram message** (if connected)  
  - **Optional iCal calendar file**  

## 4️⃣ User Roles & Permissions

| Type          | Scope           | Example Permissions                  |
|---------------|-----------------|--------------------------------------|
| Admin         | Global SaaS     | Manage tenants, billing, reports     |
| Tenant Staff  | Tenant / Branch | Manage courses, classes, students    |
| Student       | Branch / Tenant | Access courses, take tests           |

**RBAC Highlights:**

- **Tenant-level and branch-level** permissions  
- **Sample keys**:  
  - `manage_courses`  
  - `manage_classes`  
  - `assign_students`  
  - `view_reports`  

---

## 5️⃣ Multi-Tenant Architecture

1. **Tenant Isolation** with `tenant_id` on most tables  
2. **Branch-Level Scoping** for courses, students, and reporting  
3. **Dynamic Class Assignment** handled per tenant/program  
4. **Payment Isolation** per tenant for SaaS billing  

---

## 7️⃣ Future Enhancements

- **Certificate generation** after course completion  
- **AI-powered recommendations** for class placement and learning paths  
- **Gamification** (points, badges)  
- **Integrated Chat** (student-student, student-instructor)  
- **Zoom / Video Conferencing Integration** for live classes  
- **White-label SaaS** per tenant with custom domain & branding  
- **Mobile app support** for offline learning  

### Automated Reminders & Smart Scheduling

- **Automated Reminders**  
  - Send class reminders **1 day** and **1 hour before class**  
- **Push Notifications**  
  - Via mobile app for schedules, updates, and reminders  
- **Student Self-Reschedule**  
  - Allow students to **move to another class** if slots are available  
- **Dynamic QR Check-In**  
  - For **physical classes** to verify attendance  
- **Recording Links for Online Classes**  
  - Provide access to recordings **after session ends**  

---

## 3️⃣ Core Application Flows

### 3.1 Student Learning Flow

```mermaid
flowchart TD
A[Student Login] --> B[View Enrolled Courses/Programs]
B --> C[Start Lesson]
C --> D{Lesson Type?}
D -->|Video| E[Watch Video]
D -->|PDF| F[View/Download PDF]
E --> G[Update Lesson Progress]
F --> G[Update Lesson Progress]
G --> H{Last Lesson Completed?}
H -->|Yes| I[Mark Course Completed]
H -->|No| B
```

### 3.2 Staff Course & Program Management Flow

```mermaid
flowchart TD
A[Staff Login] --> B[Create Program or Course]
B --> C[Add Modules & Lessons]
C --> D[Configure Pre-test/Post-test]
D --> E[Assign Course/Program to Branch]
E --> F[Monitor Student Progress & Reports]
```

### 3.3 Program Purchase & Class Auto-Assignment Flow

```mermaid
flowchart TD
A[Student Buys/Subscribes Program] --> B[Process Payment]
B --> C{Payment Success?}
C -->|No| D[Order Pending/Retry]
C -->|Yes| E[Check Program Pre-test Requirement]

E -->|No| F[Direct Class Assignment to Default/Next Available Class]
E -->|Yes| G[Student Takes Pre-test]
G --> H[Calculate Score & Determine Level]

H --> I{Class Available & Not Full?}
I -->|Yes| J[Assign Student to Class]
I -->|No| K[Create New Class & Assign Student]

J --> L[Enrollment Success]
K --> L[Enrollment Success]

L --> M[Fetch Class Schedule & Delivery Info]
M --> N[Send Multi-Channel Notifications]
```

### 3.4 Class Notification Flow (Programs with Classes Only)

```mermaid
flowchart TD
A[Enrollment Success] --> B[Fetch Class Schedule]
B --> C{Class Mode?}
C -->|Physical| D[Include Location & Room]
C -->|Online| E[Include Zoom/Meet URL]

D --> F[Send Notifications]
E --> F[Send Notifications]

F --> G[Email with Class Details]
F --> H[WhatsApp / Telegram Message]
F --> I[Optional iCal Calendar Event]
```

### 3.5 Student Flow: Program Without Class

```mermaid
flowchart TD
A[Student Buys/Subscribes Program] --> B[Process Payment]
B --> C{Payment Success?}
C -->|No| D[Order Pending/Retry]
C -->|Yes| E[Grant Program Access Immediately]
E --> F[Student Can Access Courses]
F --> G[Student Starts Course]
G --> H[Follow Student Learning Flow]
```

**Flow Notes:**

- No class assignment is needed.  
- Student can start the course immediately after payment.  
- Progress tracking and lesson access follow the **Student Learning Flow**.

### 3.6 Student Flow: Program With Class

```mermaid
flowchart TD
A[Student Buys/Subscribes Program] --> B[Process Payment]
B --> C{Payment Success?}
C -->|No| D[Order Pending/Retry]
C -->|Yes| E[Check Program Pre-Test Requirement]

E -->|No| F[Auto-Class Assignment]
E -->|Yes| G[Student Takes Pre-Test]
G --> H[Determine Class Level & Assign]

H --> I[Check Class Capacity]
I -->|Full| J[Create New Class & Assign Student]
I -->|Available| K[Assign to Existing Class]

J --> L[Enrollment Success]
K --> L[Enrollment Success]

L --> M[Send Multi-Channel Notifications]
M --> N[Student Waits Until Class Start]
N --> O[Student Attends First Class]
O --> P[Student Starts Course]
P --> Q[Follow Student Learning Flow]
```

**Flow Notes:**

- Student cannot start learning until class assignment & schedule notification.
- If pre-test is required, class level is determined by score thresholds.
- Notifications include date, time, location/Zoom link.
- After class start, the Student Learning Flow applies.

### 3.7 Tenant Flow: From Registration to Selling Courses/Classes

```mermaid
flowchart TD
A[Tenant Registers on SaaS LMS] --> B[Admin Approves Tenant (Optional)]
B --> C[Tenant Setup Profile & Branding]
C --> D[Create Branches (Optional)]
D --> E[Invite Staff & Assign Roles]
E --> F[Create Courses & Modules]
F --> G[Optionally Create Programs & Classes]
G --> H[Configure Pre-Test/Post-Test (Optional)]
H --> I[Assign Courses/Programs to Branches]
I --> J[Set Pricing / Subscription Plans]
J --> K[Publish Courses/Programs]

K --> L[Students Buy/Subscribe]
L --> M{Program Has Class?}

M -->|No| N[Student Starts Course Immediately]
N --> O[Follow Student Learning Flow]

M -->|Yes| P[Student Assigned to Class]
P --> Q[Student Receives Schedule & Notifications]
Q --> R[Class Starts or Student Joins First Session]
R --> S[Follow Student Learning Flow]
```
