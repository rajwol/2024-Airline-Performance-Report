# 2024 Airline Performance Analysis Report

## Project Overview  
This report provides a **data-driven analysis of airline performance** based on **Power BI dashboards and SQL queries**. I was provided with a datset containing **detailed flight performance metrics** from various airlines. The stakeholders of the FAA requested an **in-depth assessment** to evaluate **flight delays, cancellations, and on-time performance** across different commercial carriers.

### **Key Goals of the Analysis**
- Identify **which airlines experience the most delays and cancellations**.
- Determine the **leading causes of flight cancellations**.
- Compare **on-time performance** across airlines.
- Provide **data-driven recommendations** to improve airline performance.

### **Data Sources**
The analysis was conducted using:  
- **SQL Queries** for extracting insights from raw datasets. 
- A **[Power BI Dashboard](https://app.powerbi.com/view?r=eyJrIjoiNTA0NjBhYjktODE5OC00MjUzLWI2YTgtOTE5OTAxNjg5MTE2IiwidCI6IjVhNzRkMDlhLWU5YzktNDAzZi1iMGZkLTk5ZGVjNDE4OTdlZCIsImMiOjN9)** for data visualization.
  
  ![GIF](https://github.com/user-attachments/assets/ad6ac12d-2dcc-404d-9210-a2337fda6456)

---

## Flight Performance Summary  
My first priority was to create a SQL query that analyzes airline performance based on key flight metrics (total flights, delayed flights, and canceled flights for each airline). To ensure a fair assessment, I incorporated conditional aggregations to calculate percentages, allowing performance evaluation without bias towards airlines with a higher total flight volume. The results are grouped by airlined and sorted by on-time performance in descending order to identify the airlines with the best and worst punctuality rates.
### SQL Query:
```sql
SELECT 
    a.airline AS [Airline],
    COUNT(*) AS [Total Flights],

    -- Calculate On-Time Percentage
    CONCAT(
        ROUND(
            (CAST(SUM(CASE WHEN departure_delay <= 0 AND f.cancelled = 0 THEN 1 END) AS FLOAT) 
            / COUNT(*)) * 100.0, 2
        ), '%'
    ) AS [On-Time %],

    -- Count of Delayed Flights
    COUNT(CASE WHEN departure_delay > 0 THEN 1 END) AS [Delayed Flights],

    -- Calculate Delay Percentage
    CONCAT(
        ROUND(
            (CAST(SUM(CASE WHEN departure_delay > 0 THEN 1 ELSE 0 END) AS FLOAT) 
            / COUNT(*)) * 100.0, 2
        ), '%'
    ) AS [Delay %],

    -- Count of Canceled Flights
    SUM(CASE WHEN f.cancelled = 1 THEN 1 ELSE 0 END) AS [Canceled Flights],

    -- Calculate Cancellation Percentage
    CONCAT(
        ROUND(
            (CAST(SUM(CASE WHEN f.cancelled = 1 THEN 1 ELSE 0 END) AS FLOAT) 
            / COUNT(*)) * 100.0, 2
        ), '%'
    ) AS [Cancellation %]

FROM flights f 
LEFT JOIN airlines a ON f.airline = a.iata_code

GROUP BY a.airline
ORDER BY 3 DESC;

```
### Results:
<img width="781" alt="Image" src="https://github.com/user-attachments/assets/15bc5467-6a67-472f-8ed7-ecf5a9b9b0b9" />

### Key Observations

- 🏆 **On-Time Performance**:  
  - **Best On-Time Airlines**: **Alaska Airlines (74.4%)**, **Hawaiian Airlines (73.4%)**, and **Delta Air Lines (67.3%)**.  
  - **Worst On-Time Airlines**: **United Airlines (49.0%)**, **Spirit Airlines (49.83%)**, and **Southwest Airlines (53.8%)**.  

- ⏳ **Flight Delays**:  
  - **Most Delayed Airlines**: **United Airlines (49.75%)**, **Spirit Airlines (44.3%)**, and **Southwest Airlines (44.9%)**.  
  - **Least Delayed Airlines**: **Hawaiian Airlines (26.4%)** and **Alaska Airlines (25.2%)**.  

- ❌ **Flight Cancellations**:  
  - **Highest Cancellation Rate**: **American Eagle Airlines (5.1%)** and **Atlantic Southeast Airlines (2.66%)**.  
  - **Lowest Cancellation Rate**: **Hawaiian Airlines (0.22%)** and **Delta Air Lines (0.44%)**.  

 
---

## Most Common Reason for Flight Cancellations  
Next, I wanted to determine the primary reason for flight cancellations for each airline. I grouped cancellations by airline and cancellation type, then used DENSE_RANK() to rank cancellation reasons within each airline. By incorporating this logic into a Common Table Expression (CTE), I was able to efficiently retrieve the top cancellation reason for each carrier.
### SQL Query:
```sql
WITH cancelledflightsranked AS (
    -- Rank cancellation reasons by occurrence for each airline
    SELECT 
        a.airline,
        cc.cancellation_description,
        COUNT(*) AS cancelled_amount,
        DENSE_RANK() OVER (
            PARTITION BY a.airline 
            ORDER BY COUNT(*) DESC
        ) AS top_cancellation_reason
    FROM flights f
    LEFT JOIN airlines a ON f.airline = a.IATA_CODE
    LEFT JOIN cancellation_codes cc ON f.cancellation_reason = cc.cancellation_reason
    WHERE f.CANCELLED = 1
    GROUP BY a.airline, cc.cancellation_description
)

-- Select the most common cancellation reason for each airline
SELECT 
    airline, 
    cancellation_description AS [Most Common Cancellation Reason]
FROM cancelledflightsranked
WHERE top_cancellation_reason = 1;
```
### Results:
<img width="480" alt="Image" src="https://github.com/user-attachments/assets/24003c89-16c7-4f3c-8df4-2c97b93202ff" />

### Key Observations**

- 🌧️ **Weather** is the dominant cause of flight cancellations across most airlines.
- 🏢 **Alaska Airlines, Frontier Airlines, and Hawaiian Airlines** experienced the most cancellations due to **Airline/Carrier issues**.
- 🛫 **Atlantic Southeast Airlines and Virgin America** had a high cancellation rate due to **National Air System disruptions**.

---

## Challenges & Limitations

- **🌦️ Weather Dependency**: External factors like storms and airport congestion impact flight schedules.  
- **🏢 Airline Management**: Differences in airline policies for handling delays can skew comparisons.  
- **📊 Data Completeness**: Data accuracy depends on **consistent reporting** from airlines.  

---

## Recommendations & Next Steps

### 🔹 Improving On-Time Performance  
- Airlines with **low on-time performance** (**Spirit - 49.83%, United - 49.0%, Southwest - 53.8%**) should implement **better scheduling and operational efficiency strategies**.  
- **Goal:** Improve on-time performance by at least **10%**, bringing Spirit and United above **60%** and Southwest to at least **65%**.

### 🔹 Reducing Delays  
- Airlines with **high delay rates** (**United - 49.75%, Spirit - 44.3%, Southwest - 44.9%, JetBlue - 38.2%**) should focus on **improving turnaround times and strengthening weather contingency planning**.  
- **Goal:** Reduce delay percentages by at least **5-7%**, ensuring no airline exceeds a **40% delay rate**.

### 🔹 Minimizing Cancellations  
- **American Eagle Airlines (5.1%)** and **Atlantic Southeast Airlines (2.66%)** need **enhanced risk management strategies**, including **better crew and equipment planning, improved passenger rebooking processes, and investment in predictive analytics** to anticipate and mitigate cancellations.  
- **Goal:** Reduce cancellation rates by **30%** within the next operational cycle (e.g., from **5.1% to ~3.5%** for American Eagle and from **2.66% to ~1.8%** for Atlantic Southeast Airlines).  

### 📌 **Additional Strategic Recommendations**  
✅ **Implement predictive analytics** to forecast delays and cancellations.  
✅ **Enhance ground crew efficiency** to minimize turnaround delays.  
✅ **Improve weather adaptation measures** by leveraging historical data trends. 



