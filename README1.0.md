📦 Supply Chain Efficiency & Risk Analytics (Google BigQuery & SQL)

Project Overview
This project focuses on analyzing the operational efficiency of logistics and supply chain processes using a real-world dataset from **DataCo Global**. The core objective is to identify operational bottlenecks, evaluate financial risks associated with delivery delays, and deliver actionable business insights to optimize key performance indicators (KPIs).

* **Technology Stack:** Google Cloud Platform (GCP), Google BigQuery, SQL (Window Functions, Advanced Aggregations, CTEs, Conditional Logic via CASE WHEN).
* **Dataset Volume:** 180,519 transactions.

---

📊 Key Business Insights & SQL Queries

 1. OTIF (On-Time In-Full) Performance & Revenue at Risk Analysis
**Objective:** Identify which shipping modes frequently breach delivery SLAs and quantify the volume of revenue at risk due to delays.

SQL
SELECT 
    `Shipping Mode` AS Delivery_Type,
    COUNT(*) AS Total_Orders,
    ROUND(SUM(CASE WHEN `Delivery Status` = 'Late delivery' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS Late_Delivery_Rate_Pcnt,
    ROUND(SUM(CASE WHEN `Delivery Status` = 'Late delivery' THEN `Order Item Total` ELSE 0 END), 2) AS Revenue_At_Risk
FROM `supply_chain.orders`
GROUP BY `Shipping Mode`
ORDER BY Late_Delivery_Rate_Pcnt DESC;
💡 Business Insight:
•	A critical operational vulnerability was identified: 95.32% of premium-tier (First Class) orders suffer from delivery delays.
•	The total revenue compromised by these delays within the First Class segment alone reaches $4.86M, exposing the company to severe refund claims and a decline in customer lifetime value (LTV).




2. Geographic Logistics Risk Mapping
Objective: Locate the Top 3 highest-risk sub-regions per global market based on delay frequencies, leveraging SQL window functions.
SQL
WITH regional_risks AS (
    SELECT 
        `Market` AS Global_Market,
        `Order Region` AS Sub_Region,
        COUNT(*) AS Total_Shipments,
        AVG(`Late delivery risk`) AS Risk_Score,
        DENSE_RANK() OVER (PARTITION BY `Market` ORDER BY AVG(`Late delivery risk`) DESC) AS Risk_Rank
    FROM `supply_chain.orders`
    GROUP BY `Market`, `Order Region`
)
SELECT 
    Global_Market, 
    Sub_Region, 
    Total_Shipments, 
    ROUND(Risk_Score * 100, 2) AS Delay_Probability_Pcnt
FROM regional_risks
WHERE Risk_Rank <= 3;

💡 Business Insight:
•	Delay probabilities remain anomalously consistent worldwide, fluctuating tightly between 53% and 57%.
•	This suggests that fulfillment bottlenecks are not isolated to local regional distribution centers but point to a systemic issue embedded within the company's central scheduling and planning algorithms.



3. Product Category Profitability & Timeline Discrepancy (ABC-Analysis)
Objective: Highlight the most profitable product segments and analyze the discrepancy between scheduled and actual shipping durations.
SQL
SELECT 
    `Category Name` AS Product_Category,
    ROUND(SUM(`Benefit per order`), 2) AS Total_Profit,
    ROUND(AVG(`Days for shipping _real_`), 1) AS Avg_Shipping_Days_Real,
    ROUND(AVG(`Days for shipment _scheduled_`), 1) AS Avg_Shipping_Days_Scheduled
FROM `supply_chain.orders`
WHERE `Delivery Status` != 'Shipping canceled'
GROUP BY `Category Name`
ORDER BY Total_Profit DESC
LIMIT 15;


💡 Business Insight:
•	High-margin categories like Fishing ($731K in profits) and Cleats ($474K in profits) drive a significant portion of overall company revenue.
•	Across all high-performing categories, the actual average shipping duration (3.5 days) consistently exceeds the scheduled baseline (2.9 days). The system systematically miscalculates and miscommunicates delivery windows by ~14 hours, inherently creating the high failure rate found in the OTIF analysis.


🚀 Strategic Recommendations:
1.	Re-engineer Premium SLA Fulfillment: Review warehouse picking, packing, and dispatch priorities for all First Class shipments to align operational execution with customer expectations.
2.	Calibrate Scheduling Baselines: Adjust the internal system's core delivery baseline from 2.9 days to 3.5 days. This instant software recalibration will significantly lower customer dissatisfaction and false delay triggers without requiring immediate physical supply chain restructuring.

