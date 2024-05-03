# Compliance-Dashboard
The delivery compliance dashboard provides insights into delivery performance and adherence to standards. It tracks KPIs, identifies issues, and supports decision-making for optimization. Its goal is to enhance customer satisfaction, ensure compliance, and drive efficiency in delivery operations.


Summary:

Source: Snowflake Data (Query: Load Number)
Standards for Different Formats: Ensure compliance across Large Format, Small Format, Food Service (non-tellsell and tellsell), Full Service Labor, Bulk Delivery, Pull Up Days, and Waste formats.
Operational Excellence Incentives: Focus on quantity of deliveries, cost-effectiveness, and optimizing order scheduling logic.
Large Format (PBNA + G2DSD): Determine delivery frequency based on weekly sales and PBV frequency ranges.
UXT Metrics: Monitor Account Type, Net Quantity, Delivery Type, and Customer Number for compliance.
Route and Customer Level Analysis: Analyze delivery performance at granular levels, including route and customer criteria.
OPEX and AOP: Compare operating costs against capital expenditures aligned with the Annual Operating Plan.
PBV Frequency Guiding Principles: Establish principles for determining delivery frequency based on Bag in Box volume and weekly volume.
Compliance KPIs: Define and track Compliance Key Performance Indicators to enhance effectiveness and meet regulatory demands.

Dataset : Pulled from snowflake cloud [Pepsi Bottling Ventures]

SQL Queries used to Pull the Datacolumns from snowflake.

SQL Queries:
-------------------------------------------------------------------------------------------------------------------------------
SELECT 
    TO_CHAR(TO_DATE(fsa.INVOICEDT_KEY::VARCHAR, 'YYYYMMDD'), 'YYYY-MM') AS INVOICE_MONTH,
    dl.LOCATIONCD AS Plant_Code,
    dl.LOCATIONNM AS Plant_Name,
    DC.SALESREGIONCD,
    DC.SALESREGIONDESC,
    dc.SALESREGIONDESC AS Region_Customer, -- Added field
    dc.DIVISIONDESC AS Division_Description, -- Added field
    fsa.VIPINVOICENBR AS INVOICE_NUMBER,
    fsa.INVOICEDT_KEY AS INVOICE_DATE,
    fsa.VIPSOLDTONBR AS ACCT,
    dc.SOLDTONM1 AS DBA,
    fsa.VIPLOADNBR AS LOAD,
    dp.PRODUCTNBR AS ITEM,
    dp.PRODUCTNM AS DESCRIPTION,
    dc.SOLDTONBR AS SAP,
    dp.UPC,
    ROUND(DIV0(sum(fsa.revenue), sum(fsa.volumecgt)),2) as newNetPrice,
    SUM(fsa.VolumeCGT) AS VOLUME,
    SUM(fsa.VolumeCBT) AS VOLUMECBT,
    SUM(fsa.REVENUE) AS AMOUNT,
    dr.ROUTENBR,
    dr.ROUTEDESC,
    dr.ROUTETYPE,
    cal.fiscalperiod AS Fiscal_Period, -- Added field
    cal.fiscalyr AS Fiscal_Year -- Added field
FROM IDENTIFIER('"PRD_SALES_DB"."SALES_DM"."FACT_SALES_ACTUALS"') fsa
INNER JOIN IDENTIFIER('"PRD_SAP_DB"."SAP_DM"."DIM_CUSTOMER"') dc ON fsa.CUSTOMER_KEY = dc.CUSTOMER_KEY
INNER JOIN IDENTIFIER('"PRD_SAP_DB"."SAP_DM"."DIM_PRODUCT"') dp ON fsa.PRODUCT_KEY = dp.PRODUCT_KEY
INNER JOIN IDENTIFIER('"PRD_SAP_DB"."SAP_DM"."DIM_LOCATION"') dl ON fsa.LOCATION_KEY = dl.LOCATION_KEY
INNER JOIN IDENTIFIER('"PRD_SAP_DB"."SAP_DM"."DIM_ROUTEHDR"') dr ON fsa.ROUTEHDR_KEY = dr.ROUTEHDR_KEY
INNER JOIN IDENTIFIER('"DEV_CONFORMED_DB"."CONF_DM"."DIM_CALENDAR"') cal ON cal.DATE_KEY = fsa.INVOICEDT_KEY -- Added join
WHERE fsa.VIPLOADNBR = '4233051'    
AND fsa.VIPSOLDTONBR = 'C2DXK'      
AND fsa.INVOICEDT_KEY BETWEEN 20240404 AND 20240404
GROUP BY 
    fsa.VIPLOADNBR, 
    fsa.INVOICEDT_KEY,
    TO_CHAR(TO_DATE(fsa.INVOICEDT_KEY::VARCHAR, 'YYYYMMDD'), 'YYYY-MM'), 
    fsa.VIPSOLDTONBR,
    dc.SOLDTONBR,
    dc.SOLDTONM1,
    dl.LOCATIONCD,
    dl.LOCATIONNM,
    DC.SALESREGIONCD,
    DC.SALESREGIONDESC,
    fsa.VIPINVOICENBR,
    dp.PRODUCTNBR,
    dp.PRODUCTNM,
    dp.UPC,
    dr.ROUTENBR,
    dr.ROUTEDESC,
    dr.ROUTETYPE,
    dc.SALESREGIONDESC, -- Added to group by clause
    dc.DIVISIONDESC, -- Added to group by clause
    cal.fiscalperiod, -- Added to group by clause
    cal.fiscalyr -- Added to group by clause
ORDER BY INVOICE_MONTH;

-----------------------------------------------------------------------------------------------------------------------------------------------

The clean Dataset Loaded into Power BI.

Compliance Standards and Ranges Defined:
The compliance standards and ranges for each format and region were clearly defined. These standards and ranges were documented to ensure consistency and clarity.
Standards and Ranges Ingested into Data Model:
A new table was created in the Power BI data model to store the compliance standards and ranges.
Columns for format, region, minimum range, and maximum range were included in the table.
This table was populated with the defined standards and ranges for each format and region.
Compliance Standards Table Joined with Existing Dataset:
The appropriate relationships between the existing dataset and the compliance standards table were determined.
Relationships were established between the format and region fields in the existing dataset and the corresponding fields in the compliance standards table.
Compliance Metrics Calculated:
Calculated columns or measures were created in the dataset to determine compliance with the defined standards and ranges.
For each delivery, calculations were made to determine whether the number of cases fell within the specified range for the corresponding format and region.
IF statements or DAX functions like SWITCH were used to evaluate compliance based on the defined ranges.
Compliance Metrics Visualized:
Visualizations were created in the dashboard to display compliance metrics and insights.
Tables, bar charts, or KPI cards were used to show compliance status for each format and region.
Instances where deliveries met or exceeded standards and where they fell short were highlighted.
Drill-Down and Filter Options Provided:
Users were allowed to drill down into compliance metrics by format, region, or other relevant dimensions.
Slicers or filters were provided to enable users to focus on specific formats or regions and compare compliance across different criteria.
Trend Analysis Included:
Trend analysis was incorporated to track compliance performance over time.
Line charts or trendlines were used to visualize changes in compliance metrics and identify areas of improvement or deterioration.
Thresholds and Alerts Set:
Thresholds for compliance metrics were defined to indicate acceptable and unacceptable levels of compliance.
Conditional formatting or alerts were implemented to notify users when compliance fell below or exceeded specified thresholds.

------------------------------------------------------------------------------------------------------------------------------------------------------


DAX Measures Calculation:

Number of Deliveries = DISTINCTCOUNT('YourTableName'[Invoice Number])

Total Order Quantity = SUM('YourTableName'[Order Quantity])

Average Order Quantity = AVERAGE('YourTableName'[Order Quantity])

Max Order Quantity = MAX('YourTableName'[Order Quantity])

Number of Deliveries = DISTINCTCOUNT('YourTableName'[Invoice Number])

Order Quantity = SUM('YourTableName'[Volume])

Average Order Quantity = AVERAGE('YourTableName'[Volume])

Total Net Price = SUM('YourTableName'[Net Price])

Delivery Cost per Order = DIVIDE([Total Net Price], [Number of Deliveries], 0)

Total Stops in Route = DISTINCTCOUNT('YourTableName'[Customer Number])



----------------------------------------------------------------------------------------------------------------------------------------------------

Real Time KPI's :  [ Key Performance Indicators]


On-Time Delivery Rate: This KPI measures the percentage of deliveries that are made on schedule without delays. It reflects the company's ability to meet customer expectations and maintain efficient logistics operations.
Order Accuracy: This KPI assesses the accuracy of deliveries by comparing the items ordered by customers with the items actually delivered. It helps ensure that customers receive the correct products, quantities, and variants.
Delivery Cost per Order: This KPI calculates the average cost incurred for each delivery, including transportation, labor, and other related expenses. It helps in optimizing delivery routes and resource allocation to minimize costs.
Delivery Time Variance: This KPI measures the variance between the scheduled delivery time and the actual delivery time. It helps identify inefficiencies in the delivery process and allows for adjustments to improve timeliness.
Delivery Completion Rate: This KPI evaluates the percentage of orders that are successfully delivered to customers without any issues or disruptions. It helps assess overall delivery performance and customer satisfaction.
Driver Performance Metrics: Metrics such as number of deliveries per day, average delivery time, and customer feedback ratings can be tracked to evaluate individual driver performance and identify areas for improvement.
Inventory Management Accuracy: This KPI assesses the accuracy of inventory levels at delivery points compared to the recorded inventory levels. It helps prevent stockouts or overstock situations and ensures smooth delivery operations.
Customer Satisfaction Score (CSAT): This KPI measures customer satisfaction with the delivery process, including factors such as delivery timeliness, order accuracy, and overall experience. It provides valuable insights into customer perceptions and helps in maintaining high levels of customer satisfaction.
Delivery Incident Rate: This KPI tracks the frequency of delivery-related incidents such as damaged goods, missing items, or delivery errors. It helps in identifying root causes and implementing corrective actions to minimize future incidents.
Route Optimization Efficiency: This KPI evaluates the efficiency of delivery routes by measuring factors such as distance traveled, fuel consumption, and delivery stops. It helps in optimizing routes to minimize transportation costs and reduce delivery times.


![image](https://github.com/Honey-Dandwani/Compliance-Dashboard/assets/89872022/b361a193-80db-42ac-98bd-0b231f18aaf6)


