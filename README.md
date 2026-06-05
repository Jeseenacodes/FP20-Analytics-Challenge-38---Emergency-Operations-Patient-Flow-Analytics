# Code Blue: Emergency Operations & Patient Flow Intelligence
### FP20 Analytics Challenge 38 

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Healthcare](https://img.shields.io/badge/Domain-Healthcare-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

---

## Overview

This Power BI report was built for **FP20 Analytics Challenge 38**, stepping into the role of a **Healthcare Operations Analyst** and **Emergency Department BI Consultant** tasked with evaluating emergency operations, patient flow, staffing efficiency, and financial performance across an 11-hospital NHS network.

The goal: transform raw healthcare operations data into actionable business intelligence that tells the story of how emergency care performance impacts patient outcomes, operational efficiency, and resource coordination.

---

## Network Snapshot

| Metric | Value |
|--------|-------|
| Hospitals analysed | 11 |
| Total patient visits | 9,994 |
| NHS regions covered | 6 |
| Hospitals in Critical pressure | 2 |
| Emergency admission rate | 44% |
| 4-hour breach rate | 1.2% |
| Avg wait time | 61.37 min |
| Avg satisfaction score | 62.50 / 100 |

---

## Report Structure

The report is built across **4 pages** within a 1920×1080 Full HD canvas:

### Page 1 - Introduction
Executive landing page with network snapshot, key insights, and prioritised recommendations. Includes navigation buttons to each analytical page.

### Page 2 - Overview
High-level network performance dashboard answering executive questions about demand, capacity, and risk.

**Visuals:**
- 9 KPI cards - Total Visits, Avg Wait Time, Avg Satisfaction, Bed Occupancy Rate, Readmission Rate, Avg Burnout Index, Mortality Rate, 4-Hour Breach Rate
- Monthly admissions trend with Bed Occupancy Rate and Rolling 3M Avg Wait
- Total Visits by Admission Type donut chart
- RAG status tiles - Wait Status and Satisfaction Status
- Hospital performance matrix with conditional color coding
- Hospitals by Pressure Category bar chart
- Avg Burnout Index by Hospital bar chart
- Total Visits by Day of Week column chart

### Page 3 - Patient Flow
Deep dive into how patients move through the care pathway, where delays occur, and which factors drive poor outcomes.

**Visuals:**
- 5 KPI cards - Emergency Admission Rate, High Severity Rate, Avg Treatment Delay, Avg LOS Hours, Patient Experience Index
- Wait Time and Treatment Delay by Severity Level (clustered bar)
- LOS Hours by Diagnosis Category (sorted bar)
- Outcomes by Admission Type (stacked bar)
- Patient Risk Groups vs Readmission Rate (line and clustered bar)
- Hospital flow detail table - Avg Wait Time, Avg LOS Hours, 4-Hour Breach Rate
- Seasonal demand trend with average reference line

### Page 4 - Staffing & Finance
Analysis of workforce pressure, financial sustainability, and operational risk across the network.

**Visuals:**
- 5 KPI cards - Staffing Efficiency Score, Staff Absence Rate, Total Overtime Hours, Avg Profit Margin, Hospital Risk Score
- Hospital Risk Score ranking (horizontal bar)
- Overtime trend by month with Avg Burnout Index overlay
- Revenue vs Operational Cost by Hospital
- Region underperformance comparison
- Hospital staffing detail table
- Burnout vs Mortality scatter (Q8 - factors connected to outcomes)

---

## Key Insights

- 🔴 **Nuffield and Sandwell** score 73+ on the Hospital Risk Index — highest in the network and above the Critical threshold
- 🔴 **Severity 1 patients wait 149 minutes** on average — longer than all other severity levels, indicating a triage prioritisation gap
- 🟡 **December overtime spikes to 4.4K hours** — 44% above monthly average, signalling a recurring winter staffing crisis
- 🟡 **Critical risk patients show 27% readmission** rate despite lowest ED volume — post-discharge care pathway gap identified
- 🟢 **Bristol Royal and Norfolk achieve 0%** 4-hour breach — only hospitals meeting the NHS standard, representing best practice to replicate

---

## 💡 Recommendations

| Priority | Recommendation |
|----------|---------------|
| 🔴 High | Conduct urgent operational review at Nuffield Health Woking and Sandwell General — both exceed risk score 70 |
| 🔴 High | Investigate triage prioritisation — Severity 1 patients waiting 149 min while lower severity patients are seen faster |
| 🟡 Medium | Pre-plan winter staffing contingency — December overtime consistently 44% above average across all hospitals |
| 🟡 Medium | Address Critical risk patient discharge planning — 27% readmission rate suggests care pathway gaps post-discharge |
| 🟢 Quick win | Study Bristol Royal and Norfolk protocols — replicate what makes them the only 0% breach hospitals in the network |

---

## Data Model

The report is built on a **star schema** with 3 fact tables and 7 dimension tables:

```
Fact Tables:
├── Fact_Patient_Visits     (29 columns — core clinical & operational data)
├── Fact_Staffing           (16 columns — shift-level staffing data)
└── Fact_Financials         (26 columns — monthly hospital financials)

Dimension Tables:
├── Dim_Hospital            (27 columns — hospital attributes & benchmarks)
├── Dim_Region              (7 columns  — regional demographics)
├── Dim_Department          (4 columns  — department types)
├── Dim_Date                (14 columns — full date intelligence)
├── Dim_Patient             (7 columns  — patient demographics & risk)
├── Dim_Doctor              (9 columns  — doctor attributes)
└── Dim_Diagnosis           (8 columns  — diagnosis categories & weights)
```

**14 relationships** — all Many-to-One, single direction filtering.

---

## DAX Measures 

Key measures built for this report:

```dax
-- Operational pressure scoring
Operational Pressure Score = 
    ( [Wait Time Score] * 0.30 )
    + ( [Occupancy Score] * 0.25 )
    + ( [Burnout Score] * 0.25 )
    + ( [Treatment Delay Score] * 0.20 )

-- RAG status classification
RAG Wait Status = 
VAR AvgWait = AVERAGE( Fact_Patient_Visits[Wait_Time_Minutes] )
RETURN
    SWITCH( TRUE(),
        AvgWait > 90,  "🔴 Critical",
        AvgWait > 60,  "🟡 At Risk",
        AvgWait <= 60, "🟢 On Target"
    )

-- Pressure categorisation
Pressure Category = 
SWITCH( TRUE(),
    [Operational Pressure Score] >= 80, "Critical",
    [Operational Pressure Score] >= 60, "High",
    [Operational Pressure Score] >= 40, "Moderate",
    "Stable"
)

-- 4-hour NHS breach standard
4-Hour Breach Rate = 
DIVIDE(
    COUNTROWS( FILTER( Fact_Patient_Visits, Fact_Patient_Visits[Wait_Time_Minutes] > 240 ) ),
    COUNTROWS( Fact_Patient_Visits )
)

-- Patient experience composite index
Patient Experience Index = 
    ( DIVIDE( [Avg Satisfaction], 100 ) * 0.50 )
    + ( ( 1 - [Complaint Rate] ) * 0.30 )
    + ( ( 1 - [Readmission Rate] ) * 0.20 )
```

---

## Report Live link

https://app.powerbi.com/reportEmbed?reportId=78fa66d7-9b9b-45eb-a2c8-d8793cd57301&autoAuth=true&ctid=509eb15f-795b-4782-bd4e-748dc6ed48df

---

## 📁 Repository Structure

```
code-blue-emergency-analytics/
│
├── README.md
├── CodeBlue_EmergencyOps.pbix       # Power BI report file
├── data/
│   ├── Fact_Patient_Visits.csv
│   ├── Fact_Staffing.csv
│   ├── Fact_Financials.csv
│   └── Dim_*.csv                    # Dimension tables
├── screenshots/
│   ├── 00_intro_page.png
│   ├── 01_overview.png
│   ├── 02_patient_flow.png
│   └── 03_staffing_finance.png
└── dax/
    └── measures.dax                 # All DAX measures documented
```

---

## Challenge Details

- **Challenge:** FP20 Analytics Challenge 38
- **Topic:** Emergency Operations & Patient Flow Analytics
- **Organised by:** ZoomCharts & FP20 Analytics
- **Role:** Healthcare Operations Analyst / Emergency Department BI Consultant
- **Pages:** 4 (Introduction, Overview, Patient Flow, Staffing & Finance)
- **Canvas:** 1920 × 1080px Full HD

---

## 📬 Connect

If you found this project useful or have feedback, feel free to connect on [LinkedIn](https://www.linkedin.com/in/jeseena-parveen-k/).

---

*Built with Power BI · DAX · Healthcare Analytics*
