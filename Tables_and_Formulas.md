
# 📊 Master Job Finance Dashboard – Excel Formula Reference

## **1. dim_job**

| Column                | Formula                                                                                                                           | Explanation                                                                    |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `JobID`               | `=tbl_stgJobList[@JOBID]`                                                                                                         | Direct pull from Job List. Unique identifier for each job (SAI File No).       |
| `BID`                 | `=tbl_stgJobList[@BID]`                                                                                                           | Building ID, pulled from Job List.                                             |
| `Project_Description` | `=tbl_stgJobList[@[Project Description]]`                                                                                         | Short description of the project.                                              |
| `ServiceID`           | `=tbl_stgWA[@[Service ID]]`                                                                                                       | Lookup from Work Authorization. Key for joining other tables.                  |
| `WA_Number`           | `=XLOOKUP([@ServiceID], tbl_stgWA[Service ID], tbl_stgWA[WA #], "")`                                                              | Finds Work Authorization number based on Service ID. Returns blank if missing. |
| `WA_Priority`         | `=XLOOKUP([@ServiceID], tbl_stgWA[Service ID], tbl_stgWA[Priority], "")`                                                          | Pulls priority from WA. Useful to flag emergencies or extra billing.           |
| `Primary_Hygienist`   | `=XLOOKUP([@ServiceID], tbl_stgWA[Service ID], tbl_stgWA[Primary Hygienist], "")`                                                 | Who to contact for approvals; pulled from WA.                                  |
| `Job_Start_Date`      | `=MINIFS(tbl_stgReoccupancy[START_DATE], tbl_stgReoccupancy[JobID], [@JobID])`                                                    | Earliest reoccupancy start date for the job.                                   |
| `Job_End_Date`        | `=IF(COUNTIFS(tbl_stgReoccupancy[JobID],[@JobID])=0,"",MAXIFS(tbl_stgReoccupancy[END_DATE], tbl_stgReoccupancy[JobID],[@JobID]))` | Latest end date. Returns blank if no reoccupancy exists.                       |

> 💡 **Tip:** All lookups are keyed to **Service ID** to ensure uniqueness.

---

## **2. dim_technician**

| Column            | Formula / Logic                     | Explanation                              |
| ----------------- | ----------------------------------- | ---------------------------------------- |
| `Technician_Name` | `=tbl_stgPayroll[@[EMPLOYEE NAME]]` | Pulls name from payroll.                 |
| `Is_Active`       | Manual / Y/N                        | Indicates if technician is still active. |

> This is a simple “who” table; roles are encoded in payroll codes, no need to add a Role column.

---

## **3. dim_technician_code**

| Column            | Formula / Logic | Explanation                                                                  |
| ----------------- | --------------- | ---------------------------------------------------------------------------- |
| `Technician_Code` | 00,01,02,03,... | Each code indicates job type: 00 Admin, 01 Project Monitor, 03 Investigator. |

---

## **4. dim_payroll_rate**

| Column             | Formula / Logic                                                              | Explanation                                           |
| ------------------ | ---------------------------------------------------------------------------- | ----------------------------------------------------- |
| `EmpKey`           | `=LOWER(CONCAT(tbl_stgPayroll[@EmpName], TEXT(tbl_stgPayroll[@Code],"00")))` | Unique key for each technician & code combo.          |
| `Cost_Hourly_Rate` | Pull from payroll rate table                                                 | Hourly cost for the technician.                       |
| `SCA_Hourly_Fee`   | Pull from payroll rate table                                                 | What SCA is billed per hour.                          |
| `Rate_Start_Date`  | Date when this rate became active                                            | Needed for historical calculations.                   |
| `Rate_End_Date`    | `=IF(ISBLANK(end_date_cell),TODAY(), end_date_cell)`                         | Avoids blank cells; ensures calculations are bounded. |
| `Rate_Active`      | Y/N                                                                          | Marks the current active rate.                        |

---

## **5. fact_payroll / fact_labor**

| Column          | Formula / Logic                                                                                                     | Explanation                                                                    |
| --------------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `Hours_Worked`  | `=SUMIFS(tbl_fact_payroll[Hours_Worked], tbl_fact_payroll[JobID], [@JobID])`                                        | Total hours per job. Make sure JobID matches between dim_job and fact_payroll. |
| `Labor_Cost`    | `=SUMIFS(tbl_fact_payroll[Hours_Worked]*tbl_dim_payroll_rate[Cost_Hourly_Rate], tbl_fact_payroll[JobID], [@JobID])` | Multiply hours by hourly rate.                                                 |
| `Labor_SCA_Fee` | `=SUMIFS(tbl_fact_payroll[Hours_Worked]*tbl_dim_payroll_rate[SCA_Hourly_Fee], tbl_fact_payroll[JobID], [@JobID])`   | What SCA is billed for labor.                                                  |

---

## **6. dim_lab**

| Column            | Formula / Logic                                                                                                                                                                   | Explanation                                 |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| `Lab_Facility`    | `=UNIQUE(tbl_stgLabReports[FACILITY])`                                                                                                                                            | Pull distinct labs from reports.            |
| `Lab_Client_Type` | `=UNIQUE(tbl_stgLabReports[CLIENT_TYPE])`                                                                                                                                         | SCA, PRIVATE, NYCHA, DOE, etc.              |
| `Lab_Sample_Type` | `=UNIQUE(tbl_stgLabReports[SAMPLE TYPE])`                                                                                                                                         | PCM, TEM, DAF, etc.                         |
| `Lab_Rate`        | `excel =INDEX(tbl_stgLab_Rates[[LTS]:[ATLAS]], MATCH([@Lab_Sample_Type], tbl_stgLab_Rates[SAMPLE_TYPE],0), MATCH([@Lab_Facility], {"LTS","ALLAB","ATC","KAM","EMSL","ATLAS"},0))` | Pulls numeric rate for lab type & facility. |
| `Lab_SCA_Fee`     | `=XLOOKUP([@Lab_Sample_Type], tbl_stgLab_Rates[SAMPLE_TYPE], tbl_stgLab_Rates[SCA_FEES],0)`                                                                                       | What SCA is billed for this sample type.    |
| `Lab_Key`         | `=LOWER([@Lab_Facility]&"_"&[@Lab_Client_Type]&"_"&[@Lab_Sample_Type])`                                                                                                           | Unique key for lab joins.                   |
| `Is_Active`       | Y/N                                                                                                                                                                               | Flags active labs.                          |

---

## **7. fact_labs**

> ⚠️ **Warning:** fact_labs currently is messy. Issues include:

* Duplicate rows for same sample/job
* No normalization to link with dim_lab
* Rate calculations are inconsistent (some pulled from `stg_lab_rates`, some hard-coded)
* Mixing client types

**✅ Recommendation:** Do a **do-over** for fact_labs:

1. Pull **unique JobID + Lab_Facility + Sample Type + Client Type** from `stg_lab_reports`.
2. Use **dim_lab** for Lab_Rate and Lab_SCA_Fee.
3. Calculate `Total_Cost = Quantity * Lab_Rate` and `Total_SCA_Fee = Quantity * Lab_SCA_Fee`.
4. Include `Date_of_Report` and `Batch_No` for time-series tracking.
5. Make it a proper **fact table** with all joins keyed to `JobID` and `Lab_Key`.

---

✅ **Extra Notes**

* Always key lookups to **Service ID (for jobs)** or **EmpKey (for technicians)** or **Lab_Key (for labs)**.
* For integrity, consider an **Integrity Check sheet** using formulas like:

```excel
=IF(AND([@[WA_Number]]<>"",[@ProjectID]<>"",[@WA_Priority]<>""),"OK","Missing WA")
```

* Keep `Rate_End_Date` populated (use today for active rates) to avoid miscalculations.
* Quantity in labs comes from `stg_lab_reports`.

