# Patient Waiting Time Project
## About
The **Patient Waiting Time** project aims to analyze patient waiting times at a healthcare facility. The dataset includes critical information such as **Doctor Type**, **Patient Type**, **Entry Time**, and **Completion Time**. This analysis will help identify factors affecting waiting times, determine staffing needs, and suggest potential improvements to enhance patient experience.

## Table of Contents
- [Introduction](#introduction)
- [Data Preparation](#data-preparation)
- [Analysis Questions](#analysis-questions)
   - [Does the patient type affect the waiting time?](#1-does-the-patient-type-affect-the-waiting-time)
   - [Average Wait Time by Doctor Type](#2-average-wait-time-by-doctor-type)
   - [Total Revenue by Financial Class](#3-total-revenue-by-financial-class)
   - [Does the financial class of patients affect total expenses?](#4-does-the-financial-class-of-patients-affect-total-expenses)
   - [Top 5 Most Frequent Days for Entries](#5-top-5-most-frequent-days-for-entries)
   - [Correlation Between Entry Time and Wait Time](#6-correlation-between-entry-time-and-wait-time)
   - [Documentation for Patient Wait Time Trends Over Time](#7-documentation-for-patient-wait-time-trends-over-time)
   - [Revenue Vs. Wait Time](#8-revenue-vs-wait-time)
- [Anomalies and Inconsistencies](#anomalies-and-inconsistencies)
- [Summary of Key Findings](#summary-of-key-findings)
- [Visualizations](#visualizations)
- [Conclusion](#conclusion)

## Introduction

#### Brief Overview of The Project
The Patient Waiting Time project focuses on analyzing the duration patients wait to see a doctor, identifying trends, and understanding the factors that contribute to varying waiting times.

#### Purpose of The Project
This project analyzes patient waiting times to uncover patterns and identify the underlying factors contributing to delays. Healthcare facilities can implement strategies to enhance patient experience, optimize resource allocation, and improve overall operational efficiency by understanding these dynamics. 

#### Dataset Source
The dataset used for this project is titled **Hospital Patient Data** and can be accessed on Kaggle at the following link: [Hospital Patient Data](https://www.kaggle.com/datasets/abdulqaderasiirii/hospital-patient-data). This dataset encompasses a comprehensive collection of data from patients entering the hospital until their exit. It was updated 2 years ago by **Abdulqader Asiirii**.

---

## Data Preparation
#### Loading Data
   - Database Setup: Created a database named patient_db in MySQL (later renamed hospital_db).
   - Table Creation: Define the patients' table using the appropriate data types for each column.
   - Data Import: Imported approximately 5,436 rows from the original dataset. Due to size constraints, only a portion of the full dataset was loaded.
   - File Splitting: Large dataset handling is planned using Excel to split files if additional data is required.

#### Exploring Raw Data
   - Column Overview: Inspected column types and sample entries, confirming that certain columns, like lab_cost, contain non-numeric characters ($) in most rows, which need further cleaning.
   - Data Types Review: Adjusted column data types to align with the dataset (e.g., treating patient_id as a VARCHAR due to mixed text and numeric values).

#### Cleaning Data
   - **Objective**: Prepare the dataset for analysis by addressing any inconsistencies, standardizing formats, and handling missing or placeholder values.

   - **Steps Taken**:

     i. **Handling Null or Placeholder Values**:
         - The `lab_cost` column contained rows where entries had only the `$` symbol, which was replaced with `NULL` to ensure data consistency.
         - Used the following SQL query for this transformation:
     ```sql
           UPDATE patients
           SET lab_cost = NULL
           WHERE lab_cost = '$';
     ```

     ii. **Standardizing Date and Time Formats**:
         - Ensured `entry_date` and time-related columns (`entry_time`, `post_consultation_time`, `completion_time`) were stored in valid date and time formats.
         - Queried for any entries that didn’t match standard date or time formats using:

        ```sql
           SELECT *
           FROM patients
           WHERE entry_date NOT REGEXP '^[0-9]{4}-[0-9]{2}-[0-9]{2}$'
              OR entry_time NOT REGEXP '^[0-9]{2}:[0-9]{2}:[0-9]{2}$'
              OR post_consultation_time NOT REGEXP '^[0-9]{2}:[0-9]{2}:[0-9]{2}$'
              OR completion_time NOT REGEXP '^[0-9]{2}:[0-9]{2}:[0-9]{2}$';
        ```
> [!NOTE]  
> I changed these formats in **Excel** before importing into MySQL to ensure everything was intact before beginning the analysis. This was done to ensure proper data interpretation during import, especially for date and time values.

 iii. **Removing Duplicates**:
   - Checked for duplicate records based on the combination of `patient_id`, `entry_date`, and `completion_time`.
   - Identified duplicates using:
     ```sql
        SELECT patient_id, entry_date, completion_time, COUNT(*)
        FROM patients
        GROUP BY patient_id, entry_date, completion_time
        HAVING COUNT(*) > 1;
     ```

This **Data Cleaning** process was essential to prepare a consistent, analysis-ready dataset that would yield accurate insights in the subsequent analysis steps.

---

## Analysis Questions
#### 1. Does the patient type affect the waiting time?
In this dataset, the **patient type** does not affect the waiting time because all the patients are **OUTPATIENTS**. Outpatients are people who come in for medical care but don’t need to stay overnight in the hospital. Since everyone in the dataset is the same type of patient (outpatient), there is no difference in waiting times based on patient type in this case.

#### 2. Average Wait Time by Doctor Type
**Purpose**:  
Calculate the average wait time by doctor type to identify potential delays in patient care.

**Findings**:
- **Anchor**: In retail or healthcare settings, the _anchor_ generally refers to a professional (e.g., a pharmacist or manager) who has a primary, stable location or store as their main assignment. They are the consistent, primary point of contact for that location. **838 hours, 59 minutes**
- **Locum**: This term is commonly used in healthcare, especially for doctors or pharmacists, to describe a temporary or substitute professional who fills in for someone on leave. A _locum_ practitioner is hired on a short-term basis to maintain continuity of care or service. **535 hours, 56 minutes**
- **Floating**: This describes a role that involves rotating or moving between multiple locations or departments as needed. A _floater_ typically covers shifts across different locations to provide support where needed, such as when there are staff shortages. **75 hours, 21 minutes**

**Interpretation**:  
Locum and Anchor doctors have significantly longer wait times than Floating doctors, suggesting potential inefficiencies or higher patient loads.

**Recommendation**:  
Review staffing and scheduling to balance the workload more effectively.

#### 3. Total Revenue by Financial Class
**Purpose:**
This query calculates the total revenue generated by each financial class and explores whether there is any relationship between financial class and patient wait times.

**Interpretation:**
The query reveals which financial classes contribute the most revenue, and by comparing this with wait time data, it helps identify whether higher-revenue classes also face longer wait times. If a correlation exists, it could suggest that patients from higher-revenue classes experience longer wait times due to increased service demands.

**Recommendation:**
Consider analyzing the relationship between financial class and wait times. If certain classes consistently have higher wait times, adjustments in scheduling or staffing may be needed to balance service delivery.

#### 4. Does the financial class of patients affect total expenses?
Patients who pay with **INSURANCE** have the highest combined revenue for both medication and consultation, totalling **$210,831.98**. This suggests that insurance-covered patients are generating the highest revenue for the healthcare providers compared to the other financial classes, such as **HMO**, **CORPORATE**, and **MEDICARE**.

The total revenue generated from medication and consultation combined are as follows:
- **INSURANCE**: $210,831.98
- **HMO**: $111,232.43
- **CORPORATE**: $104,079.89
- **MEDICARE**: $21,932.11 (Lowest revenue)

So, **insurance** patients are the biggest contributors in total revenue generated from medication and consultation, which could be an important insight for healthcare providers when analyzing financial performance by patient type.

#### 5. Top 5 Most Frequent Days for Entries
**Purpose:**
This query identifies the top 5 days of the week with the highest number of patient entries, providing insight into which days are busiest.

**Interpretation:**  
- **Monday** has the highest number of entries at **1,297**.
- **Tuesday** follows with **1,056** entries, while **Friday** sees **890** entries.
- **Wednesday** and **Saturday** have comparatively fewer entries, with **762** and **545** respectively.

This information can help the hospital prepare for busier days by adjusting staffing levels and resources to better manage the patient load.

**Recommendation:**
Allocate additional resources on **Monday** and **Tuesday** to handle the higher patient volume and ensure shorter wait times.

#### 6. Correlation Between Entry Time and Wait Time
**Purpose:**  
This query examines the correlation between patients' entry times and their corresponding wait times. It helps determine whether patients entering the hospital at specific times experience longer or shorter wait times.

**Interpretation:** 
The following table shows the average wait time for patients based on their entry time:

| Entry Time (Hour) | Average Wait Time (Minutes) |
|-------------------|:-----------------------------:|
| 9                 | 55.60                       |
| 8                 | 48.35                       |
| 10                | 47.64                       |
| 13                | 39.67                       |
| 11                | 38.87                       |

This suggests that patients who enter at **9 AM** experience the longest wait times, while those entering later, such as at **1 PM** and **11 AM**, tend to have shorter wait times. Wait times are generally shorter later in the day, potentially due to more efficient processing or fewer patients entering at those times.

**Recommendation:** 
Consider adjusting patient scheduling or resource allocation to reduce wait times during busier morning hours, such as **9 AM**.

#### 7. Documentation for **Patient Wait Time Trends Over Time**
**Purpose:**
This query identifies trends in patient wait times over a period, helping to spot peak times or issues with wait times that could suggest operational inefficiencies.

**Interpretation:**
The average wait times for patients over the period are as follows:
- **November 1st**: 40.839 minutes
- **November 2nd**: 44.097 minutes
- **November 3rd**: 33.435 minutes
- **November 4th**: 44.729 minutes
- **November 5th**: 41.706 minutes
- **November 6th**: 41.552 minutes
- **November 7th**: 38.813 minutes
- **November 8th**: 40.557 minutes
- **November 9th**: 41.726 minutes
- **November 10th**: 31.162 minutes
- **November 11th**: 51.581 minutes
- **November 12th**: 41.171 minutes
- **November 13th**: 46.718 minutes

The data shows some variation in wait times, with **November 10th** having the shortest wait time of **31.162 minutes** and **November 11th** the longest at **51.581 minutes**. Other dates show relatively consistent average wait times, hovering between **38** and **46 minutes**.

**Recommendation:**  
Investigate what factors contributed to the spikes in wait times on **November 11th** and **November 13th**. These outliers might indicate periods of high patient volume or staffing challenges that could be addressed to improve efficiency.

#### 8. Revenue Vs. Wait Time
This code is designed to help analyze patient data by looking at how waiting times relate to patient expenses. It aims to understand whether patients with higher expenses tend to have longer waiting times or if there’s any noticeable difference across expense categories.

#### Explanation of Code Steps
1. **Grouping Patients by Expense Level**:
   - First, calculate each patient’s total expense by combining their `medication_revenue` and `consultation_revenue`.
   - Based on these total expenses, each patient is classified into one of three categories: **High Expenses**, **Middle Expenses**, or **Low Expenses**. This allows us to easily compare waiting times across different financial classes.

In the context of categorizing expenses, a threshold is a cutoff value that we set to define boundaries between different categories. For example, if we're dividing patients into *High*, *Middle*, and *Low Expense* categories based on their total expenses, we need specific thresholds to determine what counts as "high," "middle," and "low."

2. **Calculating Average Waiting Time**:
   - For each expense category, we calculate the average waiting time in minutes. This is done by finding the time difference between when each patient entered the hospital (`entry_time`) and when their consultation was completed (`post_consultation_time`). 
   - The time difference is then averaged within each expense group, giving a summary of waiting times per category.

3. **Sorting and Displaying Results**:
   - The results are shown in a simple table, ordered by expense category. This makes it easy to see if there’s any correlation between higher expenses and waiting times.

#### Results

The output from this analysis looks like this:

| Expense Category | Average Wait Time (minutes)   |
|------------------|:-----------------------------:|
| High Expenses    | 42.15                         |
| Middle Expenses  | 49.58                         |
| Low Expenses     | 42.40                         |

These findings show that **patients with middle-range expenses had the highest average waiting time**, whereas high and low-expense groups had slightly shorter average waits. This result suggests that expense levels alone might not be the main factor driving wait times. 

### Anomalies and Inconsistencies

Throughout the analysis of the **Hospital Patient Data** dataset, several anomalies and inconsistencies were observed that may need attention for further investigation and cleaning:

1. **Outliers in Wait Times**:
   - **November 11th** recorded the highest average wait time at **51.58 minutes**, which is significantly higher than the average of most other days (which are generally between 38 and 46 minutes). This spike in wait times may suggest that there was either a staffing issue, a higher number of patients, or inefficiencies on this particular day. Further investigation is required to identify the exact cause.
   - **November 10th**, on the other hand, showed the shortest wait time at **31.16 minutes**, which could be due to fewer patients or better resource allocation on that day.

2. **Inconsistent Lab Cost Data**:
   - The **Lab Cost** column contains **$** symbols without any numeric values in approximately 26,566 rows. This inconsistency needs to be addressed either by removing the rows with invalid entries or by converting the values to a consistent numeric format to enable meaningful analysis.

3. **Financial Class and Revenue**:
   - The **financial class** data indicated that some categories (e.g., **HMO**, **Insurance**, **Corporate**) reported identical total revenue figures of **838:59:59**.. This could be due to data entry errors or missing information in the **revenue** field that was not captured correctly.

4. **Missing or Inconsistent Patient Type Data**:
   - Although the **patient type** field is generally consistent with only **Outpatients** recorded, certain records may lack specific designations, potentially skewing the analysis of patient wait times and revenues. Missing **patient type** data could lead to incomplete results when analyzing the relationship between patient type and other variables.

### Recommendations for Addressing Anomalies:
- **Investigate High Wait Times**: Perform a deeper dive into dates with outliers, such as **November 11th**, to identify operational or staffing challenges that need resolution.
- **Cleanse Lab Cost Data**: Address rows with invalid **Lab Cost** entries to ensure meaningful financial analysis.
- **Standardize Revenue Data**: Resolve inconsistencies in revenue reporting to avoid misleading financial conclusions.
- **Optimize Scheduling**: Investigate the causes behind longer wait times during peak hours (e.g., **9 AM**) and optimize patient entry scheduling to balance patient load more evenly throughout the day.
- **Handle Missing Data**: Implement strategies to fill in or handle missing **patient type** data to maintain the accuracy and integrity of analyses.

By addressing these anomalies, the dataset's quality will be improved, leading to more accurate insights and better decision-making.

---

### Summary of Key Findings 

The analysis of patient waiting times revealed critical insights that highlight areas for improvement and opportunities to enhance operational efficiency in the hospital. Below is a summary of the key findings:  

1. **Average Waiting Time**:  
   The average patient waiting time was calculated to be approximately 42.43 minutes, indicating potential inefficiencies in patient flow and resource allocation.  

2. **Longest and Shortest Waiting Times**:  
   - The **longest waiting times** were observed on specific days, particularly **Mondays**, suggesting that these days experience higher patient volumes or staffing challenges.  
   - **Sundays**, on the other hand, recorded the **shortest waiting times**, likely due to reduced patient visits or streamlined processes.  

3. **Impact of Financial Class on Waiting Time**:  
   Patients categorized under different **financial classes** exhibited varying waiting times. This suggests that financial status may influence service prioritization or resource allocation strategies.  

4. **Doctor Types and Consultation Efficiency**:  
   - Analysis of **doctor types** revealed significant differences in the duration of consultations and their impact on overall waiting times.  
   - Certain specialities, such as **specialist doctors**, tended to have longer consultation times compared to general practitioners, impacting the flow of patients.  

5. **Revenue Insights**:  
   - The **total revenue** from medications and consultations was categorized by financial class, providing a clear understanding of income sources and areas that might require strategic focus for improved profitability.  

6. **Day-wise Trends**:  
   The day-of-week analysis showed distinct patterns in waiting times, which can be leveraged to optimize scheduling and staffing. For example, increasing staff or resources on Mondays may help reduce peak-time delays.  

---

## Visualizations

This project features a range of insightful visualizations created in Power BI to uncover trends and patterns in patient waiting times. Highlights include a bar chart showing total revenue by financial class, a column chart for average waiting times by day of the week, and KPIs displaying total patients and revenue. These visuals provide a clear, data-driven perspective to support decision-making. For a detailed view, check out the [Power BI Visualization](https://drive.google.com/file/d/1TwIsagBo-a6vUQv9xQAHy69uzewpoA3-/view?usp=sharing) and [PowerPoint Slides](https://drive.google.com/file/d/1xEaGZa2TTb6-bQsYSSAmRtSaAfM8ZbXd/view?usp=sharing).

---

### Conclusion
These findings underline the need for targeted interventions, such as enhanced scheduling practices, balanced resource allocation, and patient-centered workflows, to improve service delivery and patient satisfaction.
