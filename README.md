# 📊 Job Cost vs Revenue Analytics Dashboard (Excel Data Model)

## Project Overview

This project was initiated to solve a **critical operational gap** at my company:  

There was **no centralized, structured system** for tracking job-level cost vs revenue. Handling multiple technicians, lab facilities, and reoccupancies per job was complex, and the company lacked a reliable way to answer key questions such as:
- How much does each job actually cost?
- How much are we billing the client?
- Where are margins strong or under pressure?

I took ownership of building a **clean, scalable Excel-based analytics system** that establishes a **single source of truth**, enables **job-level profitability and operational insights**, and replaces fragmented spreadsheets with a robust data model.

---

## 🎯 Business Objectives

- Establish a **job-centric data model** for analysis
- Track **actual internal costs** (labor + labs)
- Track **client billing amounts** (primary client)
- Enable **technician-level cost analysis**
- Quantify **job duration and operational complexity**
- Lay the foundation for margin and profitability reporting

---

## 🧱 Data Model Structure

The model follows a **star-schema-style design** using Excel Tables and Power Pivot.

### Dimension Table (Single Source of Truth)

#### `tblJobs` (Job List)
One row per job.

**Columns**
- `JobID` – Unique job identifier (e.g., 25S-001)
- `BID` – Building ID

Purpose:
- Central anchor for all analysis
- Drives filtering and relationships across the model

---

### Fact Tables

#### `tblReoccupancy`
Tracks job phases and scope.

**Columns**
- `JobID`
- `BID`
- `Start_Date`
- `End_Date`
- `Location`
- `Amt_ACM` – Indicator of job size/complexity

Purpose:
- Analyze job timelines
- Measure duration and reoccupancy frequency
- Support time-based insights

---

#### `tblLabs`
One row per sample collected.

**Columns**
- `JobID`
- `BID`
- `Code` – Sample code (L01, L02, etc.)
- `Sample_Type`
- `Facility` – Lab used (LTS, ALLAB, ATC, KAM)
- `Date_Collected`
- `Technician`
- `Batch_No` – Unique lab batch identifier
- `Lab Cost` – Actual cost paid by the company (calculated)

**Helper Columns**
- `Code`  
  - Created to standardize sample identification
  - Used as the key to connect lab pricing logic
- `Lab Cost`  
  - Calculated using a matrix lookup based on:
    - Sample code
    - Lab facility
  - Ensures costs are accurate, scalable, and auditable

Purpose:
- Track lab volume and cost
- Support cost analysis by:
  - Job
  - Sample type
  - Facility
  - Technician involvement

---

#### `tblPayroll`
Tracks labor time and cost.

**Columns**
- `Employee_Name`
- `JobID`
- `BID`
- `Date_Worked`
- `Week_Ending_Date`
- `Hours`
- `Hourly_Rate`

Purpose:
- Calculate technician labor cost
- Analyze labor distribution over time
- Support technician-level profitability insights

---

### Pricing / Business Rules Table

#### `tblLabPrices`
Centralized pricing logic (what we pay vs what the primary client is billed).

**Columns**
- `CODE`
- `SAMPLE_TYPE`
- `LTS`
- `ALLAB`
- `ATC`
- `KAM`
- `SCA_FEE_SCHED`

Purpose:
- Acts as a **pricing dimension**
- Separates **business rules** from transactional data
- Enables:
  - Accurate lab cost calculations
  - Clean billing logic for client charges

---

## 🧠 Key Design Decisions

- **Excel Data Model (Power Pivot)** used instead of VLOOKUP-heavy spreadsheets
- Cost and revenue **explicitly separated**
- No hard-coded pricing inside fact tables
- One-to-many relationships enforced from `tblJobs`
- Model designed to scale to:
  - Excel

---

## 📈 Outcomes

- For the first time, my company can:
  - See **true job cost**
  - Compare **actual cost vs billed revenue**
  - Identify margin leakage
  - Understand operational drivers of cost
- This project replaces guesswork with data.
- Yes — it’s overdue.  
- Yes — it’s being built correctly now.

---

## 🚀 Next Enhancements

- DAX measures for:
  - Job-level margin
  - Cost vs revenue variance
- Sample-type billing breakdowns
- Technician utilization metrics


---

## Final Note

This project exists because **the database didn’t** — and because the company had **no visibility into its own economics**.

Now it does.

And it’s only getting better.
