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

![Defects](https://github.com/user-attachments/assets/8336f67e-cf5c-4d57-a966-172c88eb16e7)


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


![Vendors](https://github.com/user-attachments/assets/cb281681-9641-4313-8651-b09714982d6f)


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


![Materials](https://github.com/user-attachments/assets/2fdbf9a7-eb4b-4270-b7c9-af9d4fd8ef16)

![Plants](https://github.com/user-attachments/assets/9338481b-611c-469e-961c-fd10514d5435)


![Down-Time Impact](https://github.com/user-attachments/assets/32876e49-4390-499a-b791-5d1500913e97)

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

 # Key Insights
High Total Defects:
The overall total defects are extremely high at 2.59 billion. The month of October and specific segments have significantly higher defect rates than others.

Medium Risk Vendors by Downtime:
Medium-risk vendors have accumulated 41,738 hours of downtime. There are spikes in Q4 2019 and other segments, indicating potential issues with certain vendors during this period.

Recent Moving Average Trends:
The moving average of downtime costs started declining sharply on November 23, 2019, dropping by 28.88% over approximately 1.27 months.
Conversely, there was a steep increase in downtime costs between January 1, 2018, and January 19, 2018, where costs surged from 14,592 to 141,168.
The longest growth trend in downtime costs was observed between January 21, 2018, and May 7, 2018, increasing by 70,512.

# Recommendations
Investigate and Address High Defect Rates:

Conduct a thorough root cause analysis for the months and segments with the highest defect rates, particularly focusing on October. Identify whether specific suppliers, materials, or production processes are contributing to these spikes.
Implement a Quality Assurance (QA) program with your suppliers to reduce defects. This could include tighter quality controls, regular audits, and setting stricter KPIs.
Optimize Medium Risk Vendors by Downtime:

For medium-risk vendors contributing to 41,738 hours of downtime, focus on performance reviews and improvement plans. Prioritize suppliers that caused spikes in Q4 2019 and analyze the root causes of the downtime.
Consider renegotiating contracts or implementing Service Level Agreements (SLAs) to hold vendors accountable for excessive downtime.
Leverage the Moving Average Trend Insights:

The decline in downtime costs after November 23, 2019, is a positive sign, indicating improvements. However, investigate what specific actions or changes led to this reduction so they can be replicated across other areas.
Analyze the steep spike in costs during January 2018 to understand the factors driving such sudden increases. Implement preventive measures to avoid similar future occurrences.
For the prolonged period of growth between January and May 2018, assess whether it was related to new supplier onboarding, changes in production, or external factors. Use these insights to plan better resource allocation during high-risk periods.

# Conclusion
The supplier assessment reveals significant areas for improvement, especially regarding defect management and downtime reduction. By taking a proactive approach to quality control, optimizing supplier performance, and learning from historical trends, the company can reduce operational risks, enhance supply chain efficiency, and lower overall costs.
