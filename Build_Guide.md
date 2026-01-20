# Build_Guide.md

## Overview

This guide walks through building the **Master Workbook** for SAI job and financial analytics, consolidating multiple workbooks into a single source of truth for dashboards. It includes dimension and fact tables, key calculations, and integrity checks.

---

## Step 1 – Create Master Workbook

1. Open Excel.
2. Create a **new workbook**.
3. Save it as:

   ```
   Master_Job_Finance.xlsx
   ```
4. Create a **sheet** for each staging/source dataset you have:

   * `stg_WA` → copy Work Authorization Sheet1 here
   * `stg_JobList` → copy 2024-2026 Job Lists here
   * `stg_Payroll` → copy Payroll Master here
   * `stg_Lab_Reports` → copy Lab Reports here
   * `stg_Lab_Rates` → copy Lab Rates here
   * `stg_Reoccupancy` → copy Reoccupancy here

**Tip:** Format each as an **Excel Table** (`Ctrl+T`) and give each a clear name (as above). Tables make lookups, formulas, and updates much easier.

---

## Step 2 – Prepare `dim_job`

`dim_job` is the **primary dimension table for jobs**.

**Columns & Data Sources:**

| Column              | Source                                                                       | Notes / Formula                                            |
| ------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------- |
| JobID               | `stg_JobList[JobID]`                                                         | Unique identifier from SAI File Number                     |
| BID                 | `stg_JobList[BID]`                                                           | Building ID                                                |
| Project_Description | `stg_JobList[Project Description]`                                           | Short description of job                                   |
| ServiceID           | `stg_WA[Service ID]`                                                         | Use this as the **one-to-one key** between WA and Job List |
| ProjectID           | `stg_WA[Project ID]`                                                         |                                                            |
| WA_Number           | `XLOOKUP([@ServiceID], stg_WA[Service ID], stg_WA[WA #], "")`                | Returns WA number from Work Authorization                  |
| WA_Priority         | `XLOOKUP([@ServiceID], stg_WA[Service ID], stg_WA[Priority], "")`            | Priority (useful for emergencies)                          |
| Primary_Hygienist   | `XLOOKUP([@ServiceID], stg_WA[Service ID], stg_WA[Primary Hygienist], "")`   |                                                            |
| Job_Start_Date      | `MINIFS(stg_Reoccupancy[START_DATE], stg_Reoccupancy[JobID], [@JobID])`      | Earliest start date of reoccupancies                       |
| Job_End_Date        | `MAXIFS(stg_Reoccupancy[END_DATE], stg_Reoccupancy[JobID], [@JobID])`        | Latest end date of reoccupancies; if blank, leave blank    |
| Contractor          | `XLOOKUP([@JobID], stg_Reoccupancy[JobID], stg_Reoccupancy[Contractor], "")` | If multiple contractors, pick primary or concatenate       |
| Expected_Cost       | `stg_WA[Current WA Total Amt]`                                               | Optional: pull total amount from Work Authorization        |

**Integrity Check:**

Create a helper column:

```excel
=IF(AND([@[WA_Number]]<>"", [@ProjectID]<>"", [@WA_Priority]<>""), "OK", "Missing WA")
```

* Flags missing WA info or priority.

---

## Step 3 – Prepare `dim_technician`

**Purpose:**

* Keep track of **all technicians**, their codes, and whether they are active.
* This will be referenced by payroll and labor fact tables.

**Columns & Data Sources:**

| Column          | Source / Formula                                                                                                     | Notes                                                       |
| --------------- | -------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| Technician_Name | `stg_Payroll[EMPLOYEE NAME]`                                                                                         | Full name, e.g., Patricia Dros                              |
| Technician_Code | `stg_Payroll[CODE]`                                                                                                  | Numeric or alphanumeric code (00, 01, 03…)                  |
| EmpKey          | `=LOWER(SUBSTITUTE([@Technician_Name]," ","") & TEXT([@Technician_Code],"00"))`                                      | Concatenates name and code; no spaces. Maintains uniqueness |
| Is_Active       | Manual or `=IF(MAXIFS(stg_Payroll[DATE WORKED], stg_Payroll[EMPLOYEE NAME],[@Technician_Name])>TODAY()-365,"Y","N")` | Optional: flag if technician has worked in the past year    |

**Notes / Tips:**

* Keep `Technician_Code` as **text**, not number, so codes like 01 don’t become 1.
* `EmpKey` is the **primary key** for linking payroll rates and fact tables.

---

### Step 3a – dim_payroll_rate

`dim_payroll_rate` keeps **hourly rate history** for each technician (SCA_FEE for SCA jobs included).

**Columns:**

| Column           | Source / Formula                      | Notes                                         |
| ---------------- | ------------------------------------- | --------------------------------------------- |
| EmpKey           | Concatenation of Name+Code            | Same as `dim_technician`                      |
| EmpName          | Technician name                       |                                               |
| Technician_Code  | From payroll                          |                                               |
| PayrollType      | From payroll (`SCA`, `Private`, etc.) | Determines rate for type of hours             |
| Cost_Hourly_Rate | From payroll                          | Regular pay rate                              |
| SCA_Hourly_Fee   | From payroll                          | What SCA pays us for this technician          |
| Rate_Active      | `Y` or `N`                            | Y = current rate, N = historical              |
| Rate_Start_Date  | Manual / contract date                | Effective start of rate                       |
| Rate_End_Date    | Manual / contract date                | Effective end; can be future (e.g., 1/1/2100) |

**Tips:**

* Keep multiple rows per technician if rates change over time.
* Use **Rate_Start_Date and Rate_End_Date** to reference the correct rate in fact tables.

---

### Step 3b – dim_technician_code (optional)

If you want a **lookup table** for code → role (for clarity or dashboards):

| Column          | Example                   |
| --------------- | ------------------------- |
| Technician_Code | 00                        |
| Role            | Admin                     |
| Description     | Admin staff / office work |

---

Once `dim_technician` and `dim_payroll_rate` are set up, linking payroll fact table is straightforward using **EmpKey**.

---


