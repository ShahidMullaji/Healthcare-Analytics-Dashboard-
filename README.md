# Healthcare-Analytics-Dashboard-

# Healthcare Analytics Dashboard (Power BI)

A two-page Power BI report that tracks hospital admissions, bed capacity and length of stay across multiple hospitals.

I built it to answer three simple questions a hospital manager would ask every morning: how many patients are in right now, how full are the beds, and how long do patients stay.

## What's inside

| File | What it is |
| --- | --- |
| `Healthcare_Analytics_Dashboard.pbix` | The Power BI report |
| `admissions.csv` | 8,000 admissions, Jan 2024 to Aug 2026 |
| `patients.csv` | 5,000 patients (age, gender, condition, doctor, insurance, medication) |
| `beds.csv` | 1,025 beds with ward, bed type and current status |
| `BRD` | Business Requirements Document for the dashboard |

## Report pages

**1. Healthcare Overview**
- KPI cards: total admissions, total beds, active admissions, occupied beds, available beds, occupancy rate, average length of stay
- Line chart of active patients by month
- Bed occupancy trend by month
- Admission type split (Elective, Emergency, Urgent)
- Patient load by hospital
- Slicers: hospital, gender, medical condition

**2. Resource Optimization**
- Occupied beds by hospital
- Average length of stay by hospital
- Bed status split (Occupied, Available, Maintenance)
- Hospital summary table
- Slicers: month, hospital, admission type, ward

## Data model

`admissions` sits in the middle and connects to `patients`, `hospitals` and `beds` through `patient_id`, `hospital_id` and `bed_id` (many-to-one). A separate `Date` table is used for the monthly charts.

A blank `discharge_date` means the patient is still admitted. All 974 blank dates have the status "Admitted", so this rule holds across the dataset.

## Key DAX measures

```DAX
Total Patients   = DISTINCTCOUNT(admissions[admission_id])
Total Beds       = SUM(hospitals[total_beds])
Active Admissions =
    CALCULATE(COUNTROWS(admissions), ISBLANK(admissions[discharge_date]))
Available Beds   = [Total Beds] - [Occupied_bed]
Averge LOS       = AVERAGE(admissions[length of stay])

Active Patients =
CALCULATE(
    DISTINCTCOUNT(admissions[patient_id]),
    FILTER(
        admissions,
        admissions[admission_date] <= MAX('Date'[Date]) &&
        (ISBLANK(admissions[discharge_date]) ||
         admissions[discharge_date] >= MAX('Date'[Date]))
    )
)
```

`length of stay` is a calculated column: `DATEDIFF(admission_date, discharge_date, DAY)`. It only exists for discharged patients.

## Known issues (being fixed)

While documenting the model I found a few things that need correcting:

- **Bed Occupancy rate is upside down.** The formula is `Total Beds / Occupied_bed`, so it should be `DIVIDE([Occupied_bed], [Total Beds])`.
- **Occupied_bed overcounts.** It uses the 974 active admissions, but those sit on only 621 distinct beds (353 overlap). The beds table also shows 621 occupied.
- **"Total Patients" counts admissions**, not unique patients. There are 8,000 admissions but 3,951 unique patients.
- **The Date table is not related to admissions**, so plain counts don't change by month. The "Admissions vs Discharges" chart also has no discharge measure yet.

The full list, with fixes, is in the BRD.

## How to open it

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/).
2. Keep the four CSV files in one folder.
3. Open the `.pbix`, then go to **Transform data → Data source settings** and point each source to your folder. The report currently loads from `F:\AAMIR\`.
4. Click **Refresh**.

## Tools

Power BI, DAX, Power Query. The CSVs were profiled with Python (Pandas) to check the model numbers.

## Author

Mohammad Shahid Abdul Ahad Mullaji, Data Analyst
[GitHub](https://github.com/ShahidMullaji) · [LinkedIn](https://linkedin.com/in/mohammad-shahid-7bba45290)
