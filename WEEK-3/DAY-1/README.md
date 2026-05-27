# 📅 Week 3, Day 1 (Day 15): Data Management & Data Quality

Welcome to **Day 15** of the Salesforce Summer Program! Today, we transition to a fundamental law of software engineering: **"Enterprise systems are only as good as their data."** 

No matter how advanced our Apex code, Lightning Web Components, or automated flows are, a database polluted with dirty, duplicated, or inconsistent records will cause business failure. Today, we focus on **Data Management**, **Bulk Migrations**, **Duplicate Prevention**, and **Enterprise Data Governance** using standard tools like the **Salesforce Data Loader**, mapped specifically to our **College Admission System** schema.

---

## 🏛️ System Schema: College Admission
Our architectural data model is mapped to standard Salesforce CRM objects:
*   **Account**: Represents a **College or educational institution** partner.
*   **Contact**: Represents a **Student, parent, or staff member** associated with the college.
*   **Lead**: Represents a **Student inquiry or application** showing active interest in admission.
*   **Opportunity**: Represents the **Admission opportunity** (the chance of converting the interested student into a successfully enrolled student).

---

## 🎯 Goal for Today
By the end of today, you should understand:
*   How to perform bulk imports, updates, upserts, and exports using **Salesforce Data Loader**.
*   The business and operational consequences of **dirty database records**.
*   How to plan a massive **Data Migration** from legacy systems (like Excel) to Salesforce.
*   How to establish programmatic and declarative **Duplicate Prevention** guardrails.
*   Why **Data Governance** is a critical engineering requirement for enterprise stability.

---

## 📚 Modules To Complete

### 1️⃣ Data Management
*   **Focus**: Importing and exporting data, mapping CSV fields to custom Salesforce objects, understanding bulk API limits, matching records using External IDs, and migration execution thinking.

### 2️⃣ Data Quality
*   **Focus**: Establishing duplicate rules and matching rules, validating data formats, enforcing mandatory fields, ensuring relational data consistency, and designing system-wide data governance rules.

---

## 🎥 Videos (Properly Verified Working Links)

*   [🎥 Introduction to Data Loader in Salesforce](https://www.youtube.com/watch?v=2KRGa86nbOg)
    *   *Focus*: Installing Salesforce Data Loader, desktop interface overview, logging in, and basic bulk import mechanics.
*   [🎥 How to Use Data Loader in Salesforce](https://www.youtube.com/watch?v=8xtJ_Ft5nVQ)
    *   *Focus*: Structuring CSV data files, mapping columns to database fields, and performing insert vs. update actions.
*   [🎥 Salesforce Data Loader Tutorial](https://www.youtube.com/watch?v=TBHjqvIaBeA)
    *   *Focus*: Deep dive into Bulk API mode, exporting massive datasets, deleting records, and handling complex relational mappings.

---

## 🏛️ Core Task 1: Bad Data Scenarios (10 Critical Failures)

To protect the operational integrity of our **College Admission System**, let's analyze **10 distinct examples of bad data problems** and details the serious business damage they cause:

| # | Bad Data Scenario | Core Cause | Business / Operational Damage (What Happens) |
| :--- | :--- | :--- | :--- |
| **1** | **Duplicate Student Profiles (Contacts)** | Lack of matching rules during online inquiry submissions. | **Double billing error** where tuition fees are split across two profiles, leading to incorrect financial balances and audit flags. |
| **2** | **Missing Email Addresses (Leads)** | Mandatory validation checkboxes omitted in inquiry forms. | **Notification blackout**; prospective students fail to receive qualification updates, admissions decisions, or event alerts. |
| **3** | **Wrong College Allocation (Accounts)** | Manual typos in relational dropdown keys during bulk import. | **Recruitment mismatch** where students are assigned to campuses or territories they did not apply for, ruining the applicant experience. |
| **4** | **Invalid Conversion Timestamps (Opportunities)**| System allows future timestamps during stage changes. | **Analytics corruption**; conversion metrics, cohort reports, and territorial speed-to-lead statistics are rendered inaccurate. |
| **5** | **Duplicate active applications (Opportunities)**| Lack of constraints blocking multi-opportunity generation. | **Wasted seat capacity** where a single student reserves multiple seats in the same term, distorting enrollment counts. |
| **6** | **Orphaned Admission Tracks (Opportunities)** | Deletion of student Contact without cascading cleanup. | **Pipeline inflation** where the system reports open pipeline values for students that do not exist, distorting forecasting. |
| **7** | **Inconsistent Contact Formatting** | Absence of telephone and zip code regex validation rules. | **Direct mail delivery failures** and failed emergency text message broadcasts during campus recruitment campaigns. |
| **8** | **Negative Scholarship Amounts on Opportunities**| Validation rules bypassed in Data Loader uploads. | **Financial ledger corruption** where negative scholarships distort endowment accounts and trigger billing errors. |
| **9** | **Orphaned Contact Profiles** | Importing students without matching billing Account IDs. | **Financial reporting gaps** where the institution is unable to trace student billing accounts, stalling enrollment. |
| **10**| **Mismatched Recruiter Assignments (Opportunities)**| Bulk updating Opportunities with legacy, outdated owner IDs. | **Support breakdown** where student inquiries are routed to deactivated employee mailboxes, leaving applicants unassisted. |

---

## 👥 Core Task 2: Data Migration Thinking (Excel $\rightarrow$ Salesforce)

Suppose our college decides to retire its legacy database of **Excel sheets** and migrate all historical applicant records into the new **Salesforce College Admission System** schema.

```
┌──────────────────────────────────────┐          ┌──────────────────────────────────────┐
│        Legacy Excel Spreadsheets     │          │         Salesforce Database          │
│                                      │          │                                      │
│ - Duplicate rows (John & Jon Doe)    │   Data   │ - Enforced unique Matching Rules     │
│ - Missing student email columns      ├─────────►│ - Validation: "Mandatory Fields"     │
│ - Text strings in Date fields        │ Loader   │ - Strong Relational Constraints      │
│ - Legacy Partner IDs that don't exist│  Upsert  │ - Clean Opportunity-Contact lookups  │
└──────────────────────────────────────┘          └──────────────────────────────────────┘
```

### The Primary Migration Challenges:

*   **Duplicate Records (Lead/Contact Pollution)**:
    *   *Challenge*: Legacy spreadsheets usually contain duplicate rows for the same student (e.g., "John Doe" and "Jon Doe" with varying emails).
    *   *Solution*: Run pre-migration scripts in Python/Excel to clean lists, and configure Salesforce Matching Rules on Leads and Contacts to block duplicate imports.
*   **Missing and Incomplete Data (Data gaps)**:
    *   *Challenge*: Spreadsheets frequently contain rows with empty fields—like applications (Opportunities) without a related student Contact ID.
    *   *Solution*: Configure Salesforce schema validation (Required Fields) to reject incomplete rows, forcing legacy owners to supply the missing keys before import.
*   **Inconsistent Data Formats (Format Mismatch)**:
    *   *Challenge*: Dates, phone numbers, and tuition figures are entered as unstructured text in Excel (e.g. "January 10th", "10-01-2026", "2026/01/10").
    *   *Solution*: Build standardized CSV maps in Excel to convert dates into standard ISO formats (`YYYY-MM-DD`) and phone numbers to numerical formats.
*   **Invalid and Orphaned Relationships (Relational drift)**:
    *   *Challenge*: Excel rows reference Partner High Schools (Accounts) or program paths that have been retired or do not exist in the new database schema.
    *   *Solution*: Populate custom external key fields in Salesforce (`External_ID__c`) on the Account object to allow the Data Loader to automatically link student Contacts to the correct accredited colleges.

---

## ⚙️ Core Task 3: Data Governance Reflection

> [!NOTE]
> *Data is the foundation of corporate intelligence. Clean data is not a database preference; it is a critical requirement for business survival.*

### Why Clean and Reliable Data is Critical for Enterprise Systems:
1.  **Trustworthy Corporate Analytics**: College directors make strategic admissions decisions based on automated charts. If data is dirty, dashboards display incorrect metrics, leading to failed recruiting investments.
2.  **Compliance and Regulatory Safety (FERPA)**: Educational institutions must report enrollment audits to state agencies. Polluted or inaccurate records can result in massive auditing fines, loss of institutional funding, or regulatory litigation.
3.  **Automated System Efficiency**: Automations rely on data inputs. Clean fields ensure flows, Apex scripts, and billing integrations execute correctly, reducing support ticket backlogs and manual review costs.

---

## 🚀 Core Task 4: Enterprise Thinking (Disaster Scenario Analysis)

Suppose an administrator executes an unchecked Data Loader import, and **50,000 student inquiry and Contact records are imported incorrectly** with mismatched email and ID values. 

```
                                [ DATA LOADER UPLOAD ]
                                          │
                     ┌────────────────────┴────────────────────┐
                     ▼                                         ▼
            [ Mismatched Emails ]                     [ Legacy ID Matches ]
                     │                                         │
                     ▼                                         ▼
           [ Mass Notifications ]                    [ Mismatched Enrollment ]
- Admission offers sent to strangers.      - Tuition pricing profiles cross-polluted.
- Privacy breach violation (FERPA).        - Student grades mapped to wrong Contacts.
```

### The Catastrophic Impact of this Error:
*   **Mass Communication Breaches (Data Privacy Violations)**:
    *   Mismatched emails mean automated system triggers will email admissions offers, financial statements, and Contact data to random recipients, triggering massive FERPA/GDPR compliance fines.
*   **Corrupted Enrollment and Tracking Logs**:
    *   Applications (Opportunities) are logged to the wrong students, leading to false rejection triggers, student disputes, and ruined institutional reputations.
*   **Financial Discrepancies & Invoice Failures**:
    *   Tuition balances and scholarships are allocated to incorrect profiles, preventing students from paying fees on time and causing financial bookkeeping chaos.
*   **Loss of Database Trust**:
    *   When records are corrupted, staff lose faith in the system, forcing them to revert to offline Excel sheets, defeating the purpose of the Salesforce platform.

---

## 📁 GitHub Submission
To submit your work, create the directory `/day15-data-management/` in your repository. The `README.md` inside must include:
*   **Data Quality Problems**: Highlighting the 10 bad data failures in a college admission database.
*   **Migration Discussion**: Detailing standard processes for converting legacy spreadsheets into relational CRM objects.
*   **Duplicate Prevention Ideas**: Outlining how to set up Matching Rules and Duplicate Rules for Contacts and Leads.
*   **Enterprise Risks of Bad Data**: Analyzing the privacy, regulatory, and auditing consequences of corrupt bulk data imports.
*   **Reflection**: Your insights on why database hygiene is an essential engineering discipline.

---

## ✍️ Revision Questions & Answers

### 1. Why is clean data important?
Clean data guarantees that automations execute reliably, strategic metrics reflect reality, billing calculations are precise, and regulatory audits comply with government privacy guidelines.

### 2. What problems happen because of duplicate records?
Duplicates cause billing splits, waste storage capacity, pollute analytics reports, generate duplicate emails to customers, and result in administrative confusion when staff edit different profiles for the same client.

### 3. Why is data migration difficult?
Migrations require mapping unstructured legacy inputs into structured databases. The difficulty lies in sanitizing incomplete fields, correcting inconsistent formats, and validating relational dependencies.

### 4. What is Data Loader used for?
Data Loader is a desktop tool used for executing bulk operations (Insert, Update, Upsert, Delete, Export) on massive datasets of up to millions of Salesforce records.

### 5. Why should enterprises validate imported data?
Validation prevents corrupted data from entering active databases. This protects background triggers from throwing exceptions, secures relational lookups, and keeps active user dashboards correct.

### 6. Why are CSV formats important?
CSV (Comma-Separated Values) is a universal, lightweight text format supported by all database engines. It simplifies massive data transfers by bypassing proprietary software boundaries.

### 7. What risks happen during bulk import?
Bulk imports carry risks of overriding valid data, triggering recursion loops in Apex codes, exceeding server computing capacities (CPU timeouts), and generating thousands of erroneous email notifications.

### 8. Why is governance important in data management?
Governance defines who can access, edit, import, or purge records. It enforces security guidelines, maintains database hygiene, and guarantees accountability throughout the data lifecycle.

---

## 🎓 End of Day Outcome
Students should now clearly understand:
*   **Enterprise Data Management**: Managing bulk operations with Salesforce Data Loader safely.
*   **Data Migration Challenges**: Structuring legacy sheets into relational Salesforce database formats.
*   **Duplicate Prevention**: Establishing rules to prevent profile pollution.
*   **Importance of Clean Data**: Safeguarding the foundation of corporate dashboards and compliance.
*   **Governance and Reliability Thinking**: Securing system reliability throughout the data lifecycle.
