# Optimizing High-Value Technical Interviews: A Constraint Analysis for Engineering Manager Hiring

## Project Abstract

This project provides a data-driven analysis of the Engineering Manager (EM) System Design interview pipeline (EMEA, 2025). I identified and quantified a critical **workload disparity** among our Staff Engineers due to operational constraints (limited availability and rigid scheduling). The analysis confirmed that this bottleneck resulted in $\approx 65\%$ of the total workload being handled by just two engineers. This compromised the statistical reliability of the performance data and risked interviewer burnout. This analysis provides actionable recommendations to stabilize workload and improve data quality for high-stakes hiring decisions.

[![Tool](https://img.shields.io/badge/SQL-Advanced-blue.svg)](https://www.mysql.com/)
[![Tool](https://img.shields.io/badge/BI_Tool-Looker%20Studio-F8A229.svg)](https://lookerstudio.google.com/)
[![Skill](https://img.shields.io/badge/Skill-Business_Justification-green.svg)](#business-recommendation)

---

## 1. Project Overview & Context

### Background and Objectives

The objective of this analysis was to assess the efficiency and consistency of the crucial **System Design** interview stage for the **Engineering Manager** role within the EMEA region in 2025. This stage serves as a high-fidelity predictor of candidate success, making consistency vital.

The analysis was initiated to quantify the impact of known operational constraints—namely, **reduced availability of two Staff Engineers (due to paternity leave/part-time status)** and a **fixed three-day-per-week onsite interview schedule**—on core hiring metrics.

### Project Goals

1.  **Quantify Workload Disparity:** Measure the total number of System Design scorecards submitted by each assigned Staff Engineer.
2.  **Assess Performance Reliability:** Calculate the System Design pass rate for each interviewer to assess if sample sizes are sufficient for reliable performance benchmarking.
3.  **Identify Bottlenecks:** Locate the source of the scheduling constraints and propose solutions to balance the interview load.

### Key Conclusion

The primary conclusion of this analysis is that **operational constraints have a measurable negative impact on data quality and resource management**. The priority is to implement flexible scheduling to solve the bottleneck before making performance judgments based on unreliable sample sizes.

---

## 2. Key Findings & Data Analysis

The data below is anonymized to protect individual performance data but maintains the relative structure of the findings.

| Interviewer | Total Scorecards Submitted | Passing Scorecards | Pass Rate (%) | Data Reliability |
| :--- | :--- | :--- | :--- | :--- |
| **Interviewer A** | 104 | 24 | 23.08% | **Highest Workload.** Risk of fatigue and a high bar. |
| **Interviewer B** | 72 | 22 | 30.56% | High volume. |
| **Interviewer C** | 60 | 24 | **40.00%** | Medium Volume / Smaller Sample. |
| **Interviewer D** | 36 | 12 | 33.33% | **Lowest Volume / Unreliable Sample.** |

### Workload Disparity: The Bottleneck

The operational constraint created a severe workload skew: Interviewers A and B handled 176 out of 272 total interviews ($\approx 65\%$ of the total load), confirming the constraint severely compromised resource distribution.

---

## 3. Looker Studio Data Visualisation


**Looker Studio Dashboard Link:** (https://lookerstudio.google.com/reporting/9bcf989c-3052-4c69-a1e0-d6a1883b6781)

---

## 4. Methodology: The Working SQL Query

The analysis required filtering on several tables (`recruiting_scorecards`, `recruiting_applications`) and creating robust conditional logic to identify passing grades.

### Core Query (System Design Pass Rate)

```sql
WITH InterviewerScorecards AS (
    -- Filters for EMEA, Engineering Manager role, System Design stage, and 2025 activity.
    SELECT
        RS.interviewer_name,
        RS.overall_recommendation AS final_recommendation,
        RS.scorecard_created_at
    FROM
        recruiting_scorecards AS RS
    JOIN
        recruiting_applications AS RA ON RS.application_id = RA.application_id
    WHERE
        RA.region = 'EMEA'
        AND RA.job_name ILIKE '%Engineering Manager%'
        AND RS.interview_name = 'System Design' 
        AND YEAR(RS.scorecard_created_at) = 2025
        AND (
            RS.interviewer_name ILIKE '%Interviewer A%' OR
            RS.interviewer_name ILIKE '%Interviewer B%' OR
            RS.interviewer_name ILIKE '%Interviewer C%' OR
            RS.interviewer_name ILIKE '%Interviewer D%'
        )
)
SELECT
    interviewer_name,
    COUNT(*) AS total_scorecards_submitted,
    -- CRITICAL FIX: Using ILIKE '%yes%' to handle inconsistent data entry (e.g., 'Yes' vs. 'Strong Yes').
    SUM(CASE 
        WHEN final_recommendation ILIKE '%yes%' THEN 1 
        ELSE 0 
    END) AS passing_scorecards,
    CASE
        WHEN COUNT(*) = 0 THEN 0.00
        ELSE (SUM(CASE WHEN final_recommendation ILIKE '%yes%' THEN 1 ELSE 0 END) * 1.0 / COUNT(*)) * 100
    END AS system_design_pass_rate_percent
FROM
    InterviewerScorecards
GROUP BY
    interviewer_name
ORDER BY
    total_scorecards_submitted DESC;
```

### 5. Recommendations:
The successful quantification of the workload disparity leads directly to the following set of actionable recommendations. The priority is to solve the bottleneck before making performance judgments.

1.  **Utilize Interviewers C and D's non-onsite availability (paternity leave, part-time status) with recorded, asynchronous interviews or flexible remote evening slots.**
    --> Expected Impact: Immediately reduces Interviewer A and B’s collective load by $\approx$ 30-40% and speeds up candidate time-to-hire.
2. **Initiate a mandatory calibration session focused on Interviewer A (highest volume, lowest rate) and Interviewer C (medium volume, highest rate).**
    --> Ensures a standardized bar is applied across the highest and lowest performers, reducing inconsistency risk.
4.  **Enforce a clean, non-text based input (e.g., a mandatory dropdown with 'Hire' / 'No Hire') on the scorecard platform.**
    --> Eliminates reliance on complex SQL logic (ILIKE) and improves data integrity for future analysis.
5. **Implement weekly tracking of Interviews Per Week per Interviewer and correlate this with their pass rate trend.**
    --> Provides early signals for burnout risk before it impacts candidate assessment quality.
