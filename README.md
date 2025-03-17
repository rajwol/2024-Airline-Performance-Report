# 2024 Airline Performance Analysis Report

![GIF](https://github.com/user-attachments/assets/ad6ac12d-2dcc-404d-9210-a2337fda6456)

## 1. Project Overview  
This report provides a **data-driven analysis of airline performance** based on **Power BI dashboards and SQL queries**.  
The primary goal is to assess flight performance by evaluating **total flights, delays, and cancellations** across different airlines.  

### **Key Data Sources:**
- **Power BI Dashboards** for visual analysis  
- **SQL Queries** for extracting performance metrics  

---

## 2. Flight Performance Summary  

### SQL Query Used:
```sql
select 
a.airline as [Airline],
count(*) as [Total Flights],
concat(round((cast(sum(case when departure_delay <= 0 and f.cancelled = 0 then 1 end) as float) /
	count(*)) * 100.0, 2),'%') as [On-Time %],
count(case when departure_delay > 0 then 1 end) as [Delayed Flights],
concat(round((cast(sum(case when departure_delay > 0 then 1 else 0 end) as float) /
	count(*)) * 100.0, 2), '%') as [Delay %],
sum(case when f.cancelled = 1 then 1 else 0 end) as [Canceled Flights],
concat(round((cast(sum(case when f.cancelled = 1 then 1 else 0 end) as float) /
	count(*)) * 100.0, 2),'%') as [Cancellation %]
from flights f 
left join airlines a on f.airline = a.iata_code
group by a.airline
order by 3 desc
```

<img width="781" alt="Image" src="https://github.com/user-attachments/assets/15bc5467-6a67-472f-8ed7-ecf5a9b9b0b9" />

### **3. Airline Performance Analysis by Category**

#### **3.1 Best and Worst On-Time Performance**

- **Best on-time performance:**  
  - 🏆 **Alaska Airlines (74.4%)**  
  - **Hawaiian Airlines (73.4%)**  
  - **Delta Air Lines (67.3%)**

- **Worst on-time performance:**  
  - ❌ **United Airlines (49.0%)**  
  - **Spirit Airlines (49.83%)**  
  - **Southwest Airlines (53.8%)**

---

#### **3.2 Airlines with the Most Flight Delays**

- **Most delayed:**  
  - 🚨 **United Airlines (49.75%)**  
  - **Spirit Airlines (44.3%)**  
  - **Southwest Airlines (44.9%)**

- **Least delayed:**  
  - ✅ **Hawaiian Airlines (26.4%)**  
  - **Alaska Airlines (25.2%)**

---

#### **3.3 Airlines with the Most Cancellations**

- **Highest cancellation rate:**  
  - ⚠️ **American Eagle Airlines (5.1%)**  
  - **Atlantic Southeast Airlines (2.66%)**

- **Lowest cancellation rate:**  
  - 🏅 **Hawaiian Airlines (0.22%)**  
  - **Delta Air Lines (0.44%)**
 
---

## 2. Most Common Reason for Flight Cancellations  

### SQL Query Used:
```sql
with cancelledflightsranked as (select 
a.airline,
cc.cancellation_description,
count(*) as cancelled_amount,
DENSE_RANK () over(partition by a.airline order by count(*) desc) as top_cancellation_reason
from flights f
left join airlines a on f.airline = a.IATA_CODE
left join cancellation_codes cc on f.cancellation_reason = cc.cancellation_reason
where f.CANCELLED = 1
group by a.airline, cc.cancellation_description)
select airline, cancellation_description as [Most Common Cancellation Reason]
from cancelledflightsranked
where top_cancellation_reason = 1
```

<img width="480" alt="Image" src="https://github.com/user-attachments/assets/24003c89-16c7-4f3c-8df4-2c97b93202ff" />

### **3. Key Observations**

- 🌧️ **Weather** is the dominant cause of flight cancellations across most airlines.
- 🏢 **Alaska Airlines, Frontier Airlines, and Hawaiian Airlines** experienced the most cancellations due to **Airline/Carrier issues**.
- 🛫 **Atlantic Southeast Airlines and Virgin America** had a high cancellation rate due to **National Air System disruptions**.



