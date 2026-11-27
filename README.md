# SaaS Subscribers Retention Cohort Analysis (Power BI)

## Business Problem
Subscription streaming businesses (such as **MavenFlix**) need to track how well newly acquired subscribers stay active over time rather than relying only on top-level subscriber counts, which can hide early cancellations or shifting churn patterns.

Using subscription records from **September 1, 2022 through September 8, 2023** (**3,069** subscription rows across **2,877** unique `customer_id`s, with **1,065** active records having a null `canceled_date`, a **$39** monthly `subscription_cost`, and `was_subscription_paid` split between **2,936** `Yes` and **133** `No`), this project transforms raw signup and cancellation dates into a monthly tenure structure to measure retention across signup cohorts.

## What the Cohort Matrix Answers
The retention cohort matrix groups subscriptions by **signup month (`Created Date (SOM)`)** on the rows and **tenure month (`Month List`)** on the columns to answer:
* **How much retention drops after Month 1:** Shows what share of each monthly signup cohort continues into Month 2 and beyond, highlighting early-lifecycle churn.
* **How newer cohorts compare to older cohorts:** Lets leadership compare retention trajectories across monthly signup waves on a consistent tenure scale (`1, 2, 3...`).
* **How right-censoring affects recent cohorts:** Because the dataset ends on **September 8, 2023**, earlier cohorts have longer observed tenure (up to **13** months for September 2022), while recent cohorts have fewer completed months.

## Cohort Retention Matrix Preview
![SaaS Subscribers Retention Cohort in Power BI]([viz/cohort_retention_matrix.png](https://github.com/Egitto-Data/Subscriber_Retention_Cohort/blob/main/viz/Capture.PNG))

## Repository Structure
```text
├── dataset/                                        # Source CSV dataset
├── SaaS_Subscribers_Retention_Cohort.Report/         # Power BI report definitions
├── SaaS_Subscribers_Retention_Cohort.SemanticModel/  # Data model, relationships, and DAX measures
├── viz/                                            # Visual exports and screenshots
├── .gitignore                                      # Git ignore rules
├── README.md                                       # Project documentation
└── SaaS_Subscribers_Retention_Cohort.pbip          # Power BI Project entry file


## Data Transformation (Power Query M)

The dataset was transformed using Power Query to prepare it for cohort-based retention analysis.

### 1. Handle Null Cancellation Dates

Replace null values in `canceled_date` with the maximum date available in the dataset:

```powerquery
#date(2023, 9, 8)
```

This ensures that active subscriptions are treated as continuing through the dataset's latest available date.

### 2. Calculate Start of Month (SOM)

Create a new column named `Created Date (SOM)` to identify the first day of each subscription's creation month.

```powerquery
Date.StartOfMonth([created_date])
```

This allows subscriptions created within the same calendar month to belong to the same cohort.

### 3. Calculate Active Month Span

Create a custom column named `Month Span` to calculate the number of active subscription months.

```powerquery
12 * (Date.Year([canceled_date]) - Date.Year([created_date]))
+ (Date.Month([canceled_date]) - Date.Month([created_date]))
+ (if Date.Day([canceled_date]) < Date.Day([created_date]) then -1 else 0)
+ 1
```

The calculation accounts for differences in years, months, and days to determine the subscription's active monthly duration.

### 4. Generate and Expand Tenure Months

Generate a list of sequential tenure months for each subscription using `List.Numbers`.

```powerquery
= Table.AddColumn(
    #"Added Custom",
    "Month List",
    each List.Numbers(1, [Month Span]),
    type list
)
```

Expand the `Month List` column using `Table.ExpandListColumn` to create individual rows for each active subscription month.

This transformation produces the structure required for cohort retention analysis, where each customer is represented across their active tenure months.

---

## DAX Measures

Two primary DAX measures were created to calculate customer retention volume and retention percentage.

### 1. Customer Retention Volume

Calculates the distinct number of customers retained in a specific cohort and tenure month.

```dax
Customer Retention Volume =
VAR CurrentMonth =
    SELECTEDVALUE (
        'Subscription Cohort Analysis Data'[Month List]
    )

VAR FirstOrderMonth =
    SELECTEDVALUE (
        'Subscription Cohort Analysis Data'[Created Date (SOM)]
    )

VAR CustomerRetentionVolume =
    CALCULATE (
        DISTINCTCOUNT (
            'Subscription Cohort Analysis Data'[customer_id]
        ),
        FILTER (
            ALL ( 'Subscription Cohort Analysis Data' ),
            'Subscription Cohort Analysis Data'[Created Date (SOM)] = FirstOrderMonth
                && 'Subscription Cohort Analysis Data'[Month List] = CurrentMonth
        )
    )

RETURN
    CustomerRetentionVolume
```

### 2. Customer Retention %

Calculates the percentage of customers retained in each tenure month relative to the original subscription volume of their cohort.

```dax
Customer Retention % =
VAR FirstMonthSubscriptionVolume =
    CALCULATE (
        DISTINCTCOUNT (
            'Subscription Cohort Analysis Data'[customer_id]
        ),
        ALLEXCEPT (
            'Subscription Cohort Analysis Data',
            'Subscription Cohort Analysis Data'[Created Date (SOM)]
        )
    )

VAR Ratio =
    DIVIDE (
        [Customer Retention Volume],
        FirstMonthSubscriptionVolume
    )

RETURN
    Ratio
```

**Calculation logic:**

- **Numerator:** Number of distinct customers retained in the selected cohort and tenure month.
- **Denominator:** Original number of distinct customers in the cohort.
- **Result:** Retention percentage for each tenure month.

---

## How to Run Locally

Follow these steps to explore the project:

1. Clone or download this repository.
2. Place the cohort matrix screenshot inside the `viz/` folder.

   Example:
   ```text
   viz/cohort_retention_matrix.png
   ```

3. Open `SaaS_Subscribers_Retention_Cohort.pbip` using **Power BI Desktop**.
4. Refresh the data model if required.
5. Navigate to the cohort retention matrix to explore customer retention patterns.

---

## Tools & Technologies

- **Power BI Desktop:** Data visualization and cohort analysis.
- **Power Query (M):** Data cleaning, transformation, and subscription tenure expansion.
- **DAX:** Customer retention calculations and analytical measures.

## Key Concepts

- SaaS Subscription Analytics
- Customer Retention Analysis
- Cohort Analysis
- Subscription Tenure
- Customer Retention Rate
- Power Query Data Transformation
- DAX Measures

---

*This project demonstrates a practical approach to SaaS customer retention analysis, transforming subscription-level data into a monthly cohort structure and calculating retention metrics using Power BI.*
