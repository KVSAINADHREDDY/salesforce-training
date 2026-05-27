# 📅 Week 2, Day 3 (Day 10): The Integrated College Management Mini Project

Welcome to **Day 10** of the Salesforce Summer Program! Today is the most critical architectural milestone of the curriculum. We transition from learning isolated technologies to building a fully-integrated, enterprise-grade application: the **College Management Mini Project**. 

This document details the unified architecture, data modeling, automated process flows, server controllers, responsive LWC layouts, scaling models, and answers to the day's core revision questions.

---

## 🎯 Goal for Today
By the end of today, you should understand how to connect every layer of the Salesforce platform into a single, cohesive application:
1.  **Database Relational Layer**: Custom CRM objects, Fields, Relational schemas.
2.  **Data Integrity Layer**: Native Validation Rules and Rollup summaries.
3.  **Process Automation Layer**: Declarative Record-Triggered Flows and automated alerts.
4.  **Programmatic Logic Layer**: Highly efficient Apex Controllers, SOQL queries, and bulkified Triggers.
5.  **User Experience Layer**: Modular, responsive Lightning Web Components (LWC) for Student, Faculty, and Admin dashboards.
6.  **Architectural Event Layer**: Decoupled, asynchronous Platform Events.

---

## 📚 Study Modules

### 1️⃣ Deep Learning Modules (IMPORTANT)
*   **Build a Simple LWC Application**
    *   *Focus*: Building and deploying multi-file LWC bundles, connecting user forms directly to server controller APIs, managing client state, and loading spinners.
*   **Lightning Web Components and Salesforce Data**
    *   *Focus*: Wire service adapters, imperative Apex calls, caching strategies, and automatic UI state refreshes.

### 2️⃣ Light Completion Modules
*   **Visualforce & Aura Basics (Overview)**
    *   *Focus*: Maintaining historical context on how legacy frameworks pass page state and handle client interaction compared to modern LWCs.

---

## 🎥 Video Resources (Verified Links)

*   [🎥 LWC Full Course for Beginners](https://www.youtube.com/watch?v=IHezETiVWPo)
    *   *Focus*: In-depth walkthrough of templates, ES6 classes, property reactivity, and browser sandboxes.
*   [🎥 Salesforce LWC Project Tutorial](https://www.youtube.com/watch?v=SygSlOlT7fU)
    *   *Focus*: Creating responsive grids, handling server returns, database updates, and UI styling.
*   [🎥 LWC Communication Explained](https://www.youtube.com/watch?v=2Kk2m0wKk9E)
    *   *Focus*: Custom DOM event bubbling, parameters, and pub-sub architectures.
*   [🎥 Aura vs LWC Explained](https://www.youtube.com/watch?v=9m4r2xZlK0U)
    *   *Focus*: Comparing architectural benchmarks, memory usage, framework weight, and render speeds.

---

## 🏛️ Core Mini Project: Unified Architecture Outline

Below is the conceptual blueprint of the **College Management System**, illustrating how each technical layer connects:

```
                  ┌────────────────────────────────────────────────────────┐
                  │                 LWC Presentation Layer                 │
                  │  [Student Dashboard] [Faculty Dashboard] [Registration] │
                  └──────────────────────────┬─────────────────────────────┘
                                             │ (Imperative Apex Calls)
                  ┌──────────────────────────▼─────────────────────────────┐
                  │                Apex Logic Controller                   │
                  │   - EnrollmentService.cls                              │
                  │   - SOQL Prerequisite Queries & DML execution          │
                  └──────────────────────────┬─────────────────────────────┘
                                             │ (DML Database Insert/Update)
                  ┌──────────────────────────▼─────────────────────────────┐
                  │              Trigger / Validation Layer                │
                  │   - EnrollmentTrigger.trigger                          │
                  │   - Validation Rules: Seats limit, email formats       │
                  └──────────────────────────┬─────────────────────────────┘
                                             │ (Database Commits)
                  ┌──────────────────────────▼─────────────────────────────┐
                  │                    Data Schema Layer                   │
                  │  [Student__c] ◄──(Junction: Enrollment__c)──► [Course__c]│
                  │       ▲                                            ▲   │
                  │       └─────────── [Department__c] ────────────────┘   │
                  └──────────────────────────┬─────────────────────────────┘
                                             │ (Post-Commit Transactions)
                  ┌──────────────────────────▼─────────────────────────────┐
                  │               Flow & Event Automation                  │
                  │   - Auto Confirmation Email Flow                       │
                  │   - Platform Event: low attendance, course capacity    │
                  └────────────────────────────────────────────────────────┘
```

---

## 📊 1. Data Schema & Relational Model

We define four primary custom objects and one key relational junction object to capture the university's data:

### Object Dictionary
1.  **`Department__c`**: The academic organizational unit (e.g., Computer Science, Mathematics).
2.  **`Course__c`**: Represents structured catalog items offered by a department.
3.  **`Faculty__c`**: Represents instructors assigned to teach courses under departments.
4.  **`Student__c`**: Represents active students registered in the university.
5.  **`Enrollment__c` *(Junction Object)***: Models the many-to-many relationship between `Student__c` and `Course__c`.

```
                    ┌─────────────────┐
                    │  Department__c  │
                    └─┬─────────────┬─┘
                      │             │
             (Lookup) │             │ (Lookup)
                      ▼             ▼
               ┌──────────┐     ┌───────────┐
               │Course__c │     │Faculty__c │
               └────┬─────┘     └───────────┘
                    │ (Master-Detail)
                    ▼
            ┌────────────────┐
            │ Enrollment__c  │◄────── (Master-Detail) ── Student__c
            └────────────────┘
```

### Relational Fields & Constraints
*   **`Course__c.Department__c`**: Lookup relationship to `Department__c`.
*   **`Faculty__c.Department__c`**: Lookup relationship to `Department__c`.
*   **`Enrollment__c.Student__c`**: Master-Detail relationship to `Student__c`. Enforces cascading deletion and security sharing rules.
*   **`Enrollment__c.Course__c`**: Master-Detail relationship to `Course__c`. Enforces relational integrity.
*   **`Course__c.Filled_Seats__c`**: Roll-up Summary field. Function: `COUNT` of `Enrollment__c` records where `Status__c = 'Active'`.
*   **`Course__c.Remaining_Seats__c`**: Formula Field. Calculation: `Max_Seats__c - Filled_Seats__c`.
*   **`Enrollment__c.Attendance_Percentage__c`**: Formula Field. Calculation: `(Classes_Attended__c / Total_Classes__c)`.

---

## 🛡️ 2. Data Integrity: Validation Rules

We write rules to prevent corrupted or logically inconsistent records from entering the database:

### A. Mandatory & Formatted Contact Information
*   **Target Object**: `Student__c`
*   **Rule Name**: `Enforce_Valid_Student_Email`
*   **Error Condition Formula**:
    ```formula
    OR(
        ISBLANK(Email__c),
        NOT(REGEX(Email__c, "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,4}$"))
    )
    ```
*   **Error Message**: "A valid email address is mandatory for all student profiles."

### B. Enforced Course Capacity Limit
*   **Target Object**: `Enrollment__c`
*   **Rule Name**: `Prevent_Over_Enrollment`
*   **Error Condition Formula**:
    ```formula
    AND(
        ISNEW(),
        Course__r.Remaining_Seats__c <= 0
    )
    ```
*   **Error Message**: "Cannot enroll student. This course has reached its maximum seat capacity limit."

---

## ⚡ 3. Declarative Automation: Process Flows

We configure two high-performance record-triggered flows in the Flow Builder:

1.  **Auto Confirmation Email (`Enrollment__c` - Record-Triggered, After Save)**:
    *   *Trigger Context*: Runs when an `Enrollment__c` record is created.
    *   *Execution*: Uses an Action element to send a templated HTML email confirmation to the student (`Student__r.Email__c`) welcoming them to the class and listing the schedule.
2.  **Low Attendance Alert (`Enrollment__c` - Record-Triggered, After Save/Update)**:
    *   *Trigger Context*: Runs when `Enrollment__c` is updated and `Attendance_Percentage__c` drops below `0.75` (75%).
    *   *Execution*: Creates a Salesforce Task assigned to the Student's Faculty advisor requesting an academic intervention, and logs a warning record inside the system.

---

## 💻 4. Programmatic Logic: Apex & Triggers

To check complex rules, run calculations, and support the user interface, we deploy custom Apex classes and triggers:

### A. Apex Service Class: Enrollment Service
This controller manages complex validation checks (such as prerequisite checks and GPA evaluations) before initiating enrollments:

```apex
public with sharing class EnrollmentService {
    
    @AuraEnabled
    public static String registerStudent(Id studentId, Id courseId) {
        try {
            // 1. Fetch Student & Course records
            Student__c student = [SELECT Id, GPA__c FROM Student__c WHERE Id = :studentId LIMIT 1];
            Course__c course = [SELECT Id, Prerequisite_Course__c, Remaining_Seats__c FROM Course__c WHERE Id = :courseId LIMIT 1];
            
            // 2. Complex Check: Verify Student GPA
            if (student.GPA__c < 2.0) {
                return 'Denied: Student GPA is below 2.0. Academic probation restricts enrollment.';
            }
            
            // 3. Complex Check: Verify Prerequisite completion
            if (course.Prerequisite_Course__c != null) {
                List<Enrollment__c> completed = [
                    SELECT Id FROM Enrollment__c 
                    WHERE Student__c = :studentId 
                    AND Course__c = :course.Prerequisite_Course__c 
                    AND Grade__c IN ('A', 'B', 'C')
                    LIMIT 1
                ];
                if (completed.isEmpty()) {
                    return 'Denied: Prerequisite course is not completed.';
                }
            }
            
            // 4. Register
            Enrollment__c newEnrollment = new Enrollment__c(
                Student__c = studentId,
                Course__c = courseId,
                Status__c = 'Active',
                Classes_Attended__c = 0,
                Total_Classes__c = 1
            );
            insert as user newEnrollment; // Secure USER mode execution
            
            return 'Success';
        } catch (Exception e) {
            throw new AuraHandledException('Enrollment failed: ' + e.getMessage());
        }
    }
}
```

### B. Trigger Event Thinking: Course Alerts & Capacity Rules
We write an Apex Trigger on `Enrollment__c` to manage real-time platform event triggers when critical changes happen:

```apex
trigger EnrollmentTrigger on Enrollment__c (after insert, after update) {
    
    if (Trigger.isAfter) {
        Set<Id> courseIds = new Set<Id>();
        for (Enrollment__c enroll : Trigger.new) {
            courseIds.add(enroll.Course__c);
        }
        
        // Query affected courses and check remaining seats
        List<Course__c> affectedCourses = [SELECT Id, Name, Remaining_Seats__c, Faculty__c FROM Course__c WHERE Id IN :courseIds];
        List<Course_Alert__e> alertsToPublish = new List<Course_Alert__e>();
        
        for (Course__c course : affectedCourses) {
            if (course.Remaining_Seats__c == 0 && course.Faculty__c != null) {
                // Instantiates a decoupled Platform Event
                alertsToPublish.add(new Course_Alert__e(
                    Course_Id__c = course.Id,
                    Faculty_Id__c = course.Faculty__c,
                    Alert_Message__c = 'The course ' + course.Name + ' is full.'
                ));
            }
        }
        
        if (!alertsToPublish.isEmpty()) {
            // Asynchronously publishes events to the system bus
            EventBus.publish(alertsToPublish);
        }
    }
}
```

---

## 🎨 5. User Interface (LWC Designs)

We design three interfaces tailored to our primary users:

1.  **Student Dashboard**:
    *   A personalized page where students view active courses, trace their radial attendance indicator (binds to `Enrollment__c.Attendance_Percentage__c`), and review GPA trends.
2.  **Faculty Dashboard**:
    *   A professor portal displays active courses. Selecting a course renders an editable grid of students where grades and attendance checkboxes can be updated in bulk.
3.  **Course Registration Screen**:
    *   A responsive multi-step wizard where students search available courses, view seats, and click "Enroll". It displays loading spinners during registration and triggers a Toast Notification when complete.

---

## 🔄 System Thinking Task 1: Complete Data Flow

Let's trace how data flows through our integrated College Management architecture:

```
[ LWC UI Screen ] ──── (1. Click Register) ───► [ JS Local Controller ]
                                                        │
[ Validation Rule ] ◄─ (3. DML Pre-Checks) ◄── [ Apex Controller ] (2. Imperative Call)
        │
        ▼ (Database Insert Trigger)
[ Apex Trigger ] ───── (4. Seat check / Event) ─► [ Database Layer ]
                                                        │
[ Dynamic UI Toast] ◄── (6. Refresh Event) ◄── [ Flow Automation ] (5. Confirmation Email)
```

1.  **Frontend Interaction**: A student logs into the Portal, browses electives, and clicks **"Enroll in Computer Science 101"**.
2.  **Client-Side Check**: The LWC JavaScript controller intercepts the event, displays a loading spinner, and sends an imperative Apex query to `EnrollmentService.registerStudent`.
3.  **Server Controller Rules**: The Apex method verifies the student's GPA and prerequisite completions. If valid, it fires a DML operation `insert newEnrollment`.
4.  **Database & Security Rules**: The Salesforce database intercepts the DML.
    *   *Validation Rules* confirm that `Remaining_Seats__c > 0`.
    *   *Apex Trigger* intercepts the save, notices that remaining seats are now `0`, and publishes a `Course_Alert__e` Platform Event.
5.  **Post-Commit Flows**: The record is saved to the database.
    *   An *After-Save Flow* reads the new record and dispatches an automated HTML confirmation email.
6.  **UI Event Resolution**: The server returns `Success` to the browser. The LWC controller hides the spinner, pops a success `Toast` notification, and uses `getRecordNotifyChange` to dynamically refresh the student dashboard widgets.

---

## 🏢 System Thinking Task 2: Architecture Thinking

In enterprise engineering, developers must combine UI, Backend, Databases, and Events into a single, cohesive architecture:

*   **UI/Frontend (Responsiveness)**: Manages layout renders, handles local user states, and runs validation rules to keep the app feeling fast and responsive.
*   **Backend (Business Logic)**: Securely processes business calculations, runs access checks (CRUD/FLS), and coordinates external integrations where the client browser cannot be trusted.
*   **Database (Persistence & Safety)**: Keeps student records consistent and auditable, enforces relational rules, and guarantees transaction security (ACID rules).
*   **Automation (Agility)**: Allows administrators to configure simple business workflows (like sending email alerts or assigning tasks) without writing custom code.
*   **Events (Decoupling & Scale)**: Processes secondary tasks (like logging logs or messaging other systems) in the background, keeping the user interface fast and responsive.

---

## 📈 System Thinking Task 3: Scaling Thinking (50,000 Concurrent Students)

Scaling an application to support 50,000 active students introduces several system challenges:

| Risk Area | Scaling Challenge | Technical Mitigation Strategy |
| :--- | :--- | :--- |
| **Performance & Limits** | Apex CPU Timeout limits and SOQL query limits (`101 queries`) are quickly hit if code is run inside loops. | Enforce strict Apex **Trigger Bulkification**. Ensure all queries are selective (using indexed fields), replace loops with Maps, and use **Batch Apex** for heavy calculations. |
| **Data Consistency** | **Record Locking**: Hundreds of students clicking "Enroll" at the same instant on a popular course causes database concurrency conflicts. | Use Apex SOQL **`FOR UPDATE`** lock queries on the parent Course record during registration. Implement an optimistic UI seat reservation queue pattern to manage checkout steps cleanly. |
| **System Notifications** | Exceeding Salesforce's daily email transaction limits. | Offload high-volume student email notifications to external transactional delivery engines (such as SendGrid or AWS SES) triggered asynchronously via Platform Events. |
| **System Security** | Visual screens exposed to DOM hijacking or client-side manipulation of records. | Enforce **Lightning Web Security (LWS)** on all UI modules. Run all backend Apex queries with `with sharing` or **`WITH USER_MODE`** to block unauthorized access. |

---

## 🧠 System Thinking Task 4: Reflection

> [!NOTE]
> Learning Salesforce changes how developers think about enterprise systems. 

Enterprise development is not just about writing clean lines of code; it is about choosing the right balance between **clicks** (declarative configurations) and **code** (programmatic customizations) while ensuring data integrity.

Platforms like Salesforce show the value of a metadata-driven runtime. Because database fields, validation configurations, automated flows, and LWC layouts all run on a single metadata engine, they communicate seamlessly. The ultimate goal of an enterprise developer is to design systems that are secure, scale efficiently, and remain easy to modify over time.

---

## ✍️ Revision Questions & Answers

### 1. Why do enterprise systems need modular architecture?
Modular designs prevent isolated bugs in one component from taking down the entire application. It also lets development teams build, test, and deploy separate modules in parallel without running into code conflicts.

### 2. Why are database relationships important?
Relationships define how entities connect, enforce database referential integrity, support rollup summary fields, cascade sharing permissions, and allow robust queries (SOQL) across joined data models.

### 3. Why are Flows insufficient for some cases?
Flows are declarative. For high-volume processing, complex integrations requiring custom encryption, specific database locking rules (`FOR UPDATE`), or complex multi-loop collections parsing, custom Apex code is required to keep performance high and respect platform limits.

### 4. Why do systems need event-driven behavior?
It decouples secondary tasks (like sending emails or auditing changes). By processing these tasks in the background, the main application thread can respond to the user immediately, resulting in faster load times.

### 5. Why is UI/backend separation important?
It separates visual design from database logic. You can redesign or update visual screens without risking database corruption, and it keeps critical business logic secure on the server where users cannot modify it.

### 6. Why do enterprise systems require testing?
Enterprise applications have hundreds of moving parts. Comprehensive testing (Apex unit tests, LWC Jest tests) ensures that adding new code or making updates does not cause regression bugs in active features.

### 7. Why is reusable UI architecture powerful?
It saves developers massive amounts of time. Creating a single flexible component (like a course card or date-picker) guarantees that styling, functional logic, and accessibility rules (WCAG) are instantly applied across the entire app.

### 8. What problems happen when systems scale?
Scaling introduces database lock conflicts, slow search queries, CPU execution timeouts, API rate limits, trigger recurrences, and visual loading delays in UI screens.

### 9. Why should automation be designed carefully?
Poorly designed automation can trigger endless loops, lock database records, hit Governor Limits (like CPU time or query limits), and degrade overall application performance.

### 10. How do all Salesforce concepts integrate together?
Salesforce integrates all layers using a unified **Metadata Engine**. The database fields feed into validation rules. Database DML events trigger Apex Triggers and Process Flows. LWC UI components consume these structures via Apex controller APIs, and Platform Events tie the layers together asynchronously.

---

## 🎓 End of Day Outcomes
You should now clearly understand:
*   How to build an integrated Salesforce application.
*   How data propagates from an interactive LWC layout down to database triggers.
*   How to design scalable data models and secure business rules.
*   How to write efficient, bulk-ready programmatic logic on a shared cloud environment.
