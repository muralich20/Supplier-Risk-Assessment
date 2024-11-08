# Supplier-Risk-Assessment
Analyzing supplier performance data ensures quality control, cost savings, and operational stability. By tracking defect rates, downtime, and material quality, companies identify underperforming suppliers, cut costs, and reduce risks. This leads to a more reliable supply chain, stronger supplier relationships, and higher operational efficiency.

The above analysis is crucial for several reasons in managing supplier risk and optimizing supply chain operations:

1. **Quality Control**: Tracking defects and their impact helps identify underperforming suppliers, ensuring that product quality standards are maintained.

2. **Cost Management**: By analyzing downtime caused by defective materials, companies can assess the financial impact on operations and take corrective actions to reduce costs.

3. **Supplier Performance Evaluation**: It enables businesses to evaluate suppliers' consistency in timely delivering high-quality materials, leading to better decision-making on future contracts.

4. **Risk Mitigation**: Understanding defect trends and downtime helps proactively manage risks, allowing businesses to develop contingency plans with reliable suppliers.

5. **Operational Efficiency**: Reducing downtime and improving supplier reliability enhances overall production efficiency, leading to smoother operations and higher customer satisfaction.

This analysis allows businesses to make data-driven decisions to optimize supplier selection, reduce operational risks, and ensure a stable supply chain.

Here’s a breakdown of why analyzing supplier performance data is important, along with real-world examples:

1. **Ensuring Product Quality**:
   - **Example**: An automotive manufacturer tracks defects in parts received from suppliers. If a specific supplier consistently delivers faulty brake components, this could lead to recalls or safety issues. By identifying this trend, the company can switch suppliers or demand improvements to maintain safety standards.

2. **Cost Reduction**:
   - **Example**: A consumer electronics company notices that a supplier’s defective microchips lead to frequent production stoppages. By analyzing downtime data, they discover this supplier is causing significant financial losses. Switching to a more reliable supplier reduces these costs, enhancing profitability.

3. **Improving Supplier Relationships**:
   - **Example**: A pharmaceutical company uses defect data to work collaboratively with a supplier of raw ingredients, helping them resolve quality issues. This strengthens the partnership, ensuring a consistent supply of high-quality inputs, crucial for compliance and patient safety.

4. **Minimizing Operational Downtime**:
   - **Example**: A food processing plant experiences production delays due to contaminated packaging materials. By analyzing downtime metrics, they identify the issue’s root cause and switch to a more reliable supplier, ensuring smooth operations and preventing further revenue loss.

5. **Mitigating Supply Chain Risks**:
   - **Example**: A tech company monitors supplier data to anticipate potential disruptions. When geopolitical tensions arise in a region where a key supplier is based, they proactively source alternatives to prevent shortages in critical components for their upcoming product launch. 

By analyzing this data, businesses can make strategic, data-driven decisions, optimize their supplier base, and prevent costly disruptions in their supply chain.



![2](https://github.com/user-attachments/assets/b7c7356d-e439-4e0a-9c2b-24bd4709a53d)

Let us understand, How can fetch the Top Worst Category (Which here is Mechanicals) by Defect rate using SQL;

<pre> WITH TotalDefects AS (
    SELECT 
        Category,
        SUM([Total Defect Qty]) AS Defects
    FROM Supplier
    GROUP BY Category
),
Top1Defect AS (
    SELECT 
        Category,
        Defects,
        RANK() OVER (ORDER BY Defects DESC) AS Rank
    FROM TotalDefects
    WHERE Defects IS NOT NULL
),
MaxDefects AS (
    SELECT 
        MAX(Defects) AS MaxValue
    FROM Top1Defect
    WHERE Rank = 1
)
SELECT 
    Category
FROM TotalDefects
WHERE Defects = (SELECT MaxValue FROM MaxDefects);</pre>

Let us Understand the same output using Power BI Dax Code;

<pre>
#1 Worst Defects Name by Category = 
Var vTable =
ADDCOLUMNS(
    SUMMARIZE(Category , Category[Category]),
    "Num_of_Defects", [Total Defects])
   
Var _TopN =
TOPN(
    1,
    _vTable,
    [Num_of_Defects],DESC)
   
Var _MaxValue =
MAXX(_TopN , [Num_of_Defects])
   
Var _Filter =
FILTER(vTable,
    [Num_of_Defects] = _MaxValue)

   Var Result =
CALCULATE(
    VALUES(Category[Category]),
    _Filter)
RETURN
Result</pre>

Summary,

The DAX code ultimately finds the category with the highest number of defects by:

- Summarizing the data by category.
- Adding a calculated column to compute total defects.
- Using the TOPN function to find the highest defect count.
- Filtering the summarized data to get the category/categories with the highest count.
- Returning the category name(s) as the result.

Downtime value,
![Screenshot (603)](https://github.com/user-attachments/assets/2b60e1d9-f66d-49c7-9a24-a02c478d81e3)

Let us find the Downtime value using SQl;
<pre>WITH TotalDowntime AS (
    SELECT 
        Category,
        SUM([Total Downtime (hrs)]) AS Downtime
    FROM Supplier
    GROUP BY Category
),
Top2Downtime AS (
    SELECT 
        Category,
        Downtime,
        RANK() OVER (ORDER BY Downtime DESC) AS Rank
    FROM TotalDowntime
    WHERE Downtime IS NOT NULL
)
SELECT 
    MIN(Downtime) AS Worst2ndDowntime
FROM Top2Downtime
WHERE Rank <= 2;
</pre>

![4](https://github.com/user-attachments/assets/b91e020f-e42a-42c0-a621-35e3f22b2fe2)

Let us understand, how to calculate the DownTime Hrs of High-Risk Vendors:

Note: We call a vendor as High-Risk if The DownTime Hrs is more than 400 hrs

<pre>WITH TotalDowntime AS (
    SELECT 
        Vendor,
        SUM([Total Downtime (Hrs) related to Supplier]) AS Downtime
    FROM SupplierQuality
    GROUP BY Vendor
),
HighRiskVendors AS (
    SELECT 
        Vendor,
        Downtime
    FROM TotalDowntime
    WHERE Downtime > 400
)
SELECT 
    SUM(Downtime) AS TotalHighRiskDowntime
FROM HighRiskVendors;
</pre>

We Calculate the DownTime cost by considering only, Impact and Rejected Defect Type;

<pre>SELECT 
    SUM([Total Downtime Cost]) AS TotalDowntimeCost
FROM 
    DefectType
WHERE 
    [Defect Type] IN ('Impact', 'Rejected');
</pre> 

![5](https://github.com/user-attachments/assets/98d223fe-9f5f-41a3-a26d-8e3cf5785b5b)

![6](https://github.com/user-attachments/assets/5068f668-c0da-4582-bbab-f02bd505b499)

![7](https://github.com/user-attachments/assets/15bd812e-139e-48a6-b838-abbebb224504)

Calculation of Moving Average,
Important Note: Make sure that FilteredData CTE:

- Joins the Calendar table with the Downtime table to ensure we have a continuous sequence of dates.
- Uses COALESCE() to replace NULL values in [Total Downtime Cost] with 0.
- Filters date to only include those up to the current date using GETDATE().

<pre>WITH FilteredData AS (
    SELECT 
        c.Date,
        COALESCE(d.[Total Downtime Cost], 0) AS TotalDowntimeCost
    FROM Calendar c
    LEFT JOIN Downtime d 
        ON c.Date = d.Date
    WHERE c.Date <= GETDATE() -- Ensures only past dates are considered
),
MovingAverage AS (
    SELECT 
        Date,
        TotalDowntimeCost,
        ROUND(
            AVG(TotalDowntimeCost) OVER (
                ORDER BY Date 
                ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
            ), 2
        ) AS MovingAvgCost
    FROM FilteredData
)
SELECT 
    Date, 
    TotalDowntimeCost, 
    MovingAvgCost
FROM MovingAverage
ORDER BY Date;

</pre>
