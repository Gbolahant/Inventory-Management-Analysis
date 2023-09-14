# Inventory-Management-Analysis
Inventory analysis report

[View here](https://app.powerbi.com/view?r=eyJrIjoiM2Q4ZTVkMmMtMmYwZi00YzQ2LWFjNDMtNDYwY2YyMzkwNDY2IiwidCI6ImQ2YjZlODcwLTkwNTgtNDgzOS1hOTJiLTNiNzRhMGI1ODliMSJ9)


<img src="Warehouse.jpg" alt="Warehouse" style="width:100%; height:auto;">

## Introduction
The Inventory Management Optimization project aims to enhance FitCapacity's inventory management practices by leveraging data analysis and predictive modelling. This documentation outlines the process and steps involved in solving this business problem using data-driven insights.


## Problem Statement
FitCapacity faces challenges in managing its inventory effectively, resulting in issues such as stockouts and overstocking. The primary problems to address include:

- Identifying products with high and low turnover rates.
- Calculating the average time products are out of stock.
- Predicting demand for major products and forecasting stockout occurrences.
- Classifying products into high-selling, low-selling, and average-selling categories.


## Skills Demonstrated 
This project demonstrates the following skills and competencies:

- Data retrieval and manipulation using SQL.
- Descriptive statistics for data analysis.
- Data visualization with Power BI.
- Business problem-solving through data-driven insights.
- Deneb Visuals
- Tooltips
- Bookmarks
- Page Navigation etc

## Data Sourcing
Data for this analysis is sourced from the Adventure Works database, which contains comprehensive sales and inventory information. The following SQL queries were employed to extract relevant data:


 **Inventory Table**
 ```
SELECT
	ProductID
	,loc.Name Location
	,Quantity
FROM Production.ProductInventory inv
JOIN Production.Location loc
ON inv.LocationID = loc.LocationID

```

**Product Table**
 ```
SELECT 
	pro.ProductID,
	pro.Name,
	CASE WHEN pro.MakeFlag = 1 THEN 'Manufactured' ELSE 'Purchased' END AS ProductSource,
	CASE WHEN pro.FinishedGoodsFlag = 1 THEN 'Salable' ELSE 'Not Salable' END AS ProductTag,
	pro.SafetyStockLevel,
	pro.ReorderPoint,
	pro.StandardCost,
	pro.ListPrice,
	pro.SellStartDate,
	pro.SellEndDate,
	pro.DiscontinuedDate,
	ISNULL(psc.Name,'Others') Subcategory,
	ISNULL(pc.Name,'Others') Category,
	ISNULL(Avg(pur.AverageLeadTime) ,Avg(pro.DaysToManufacture)) AverageLeadTime,
	ISNULL(Max(pur.AverageLeadTime) ,Max(pro.DaysToManufacture)) MaxLeadTime
FROM Production.Product pro
LEFT JOIN Production.ProductSubcategory psc ON pro.ProductSubcategoryID = psc.ProductSubcategoryID
LEFT JOIN Production.ProductCategory pc ON psc.ProductCategoryID = pc.ProductCategoryID
LEFT JOIN Purchasing.ProductVendor pur ON pro.ProductID = pur.ProductID
GROUP BY pro.ProductID,
	pro.Name, CASE WHEN pro.MakeFlag = 1 THEN 'Manufactured' ELSE 'Purchased' END , 
	CASE WHEN pro.FinishedGoodsFlag = 1 THEN 'Salable' ELSE 'Not Salable' END ,
	pro.SafetyStockLevel,	pro.ReorderPoint,	pro.StandardCost,	pro.ListPrice,	pro.SellStartDate,
	pro.SellEndDate,	pro.DiscontinuedDate,	ISNULL(psc.Name,'Others') ,	ISNULL(pc.Name,'Others'),	DaysToManufacture
```

 **Sales Table**
 ```
Select 
	salesOD.ProductID,
	salesOD.OrderQty,
	salesOD.UnitPrice,
	salesOD.LineTotal,
	salesOH.OrderDate
FROM sales.SalesOrderDetail salesOD
JOIN Sales.SalesOrderHeader salesOH
ON salesOD.SalesOrderID = salesOH.SalesOrderID
WHERE salesOH.Status = 5
```
 **Product Photo Table**
 ```
SELECT ThumbNailPhoto,LargePhoto, ProductID
FROM Production.ProductPhoto pp
JOIN Production.ProductProductPhoto ppp
ON pp.ProductPhotoID = ppp.ProductPhotoID
```

 **Purchases Table**
 ```
Select 
	PurchaseOD.ProductID,
	PurchaseOD.StockedQty OrderQty,
	PurchaseOD.UnitPrice,
	PurchaseOD.StockedQty *	PurchaseOD.UnitPrice LineTotal,
	PurchaseOH.OrderDate
FROM Purchasing.PurchaseOrderDetail PurchaseOD
JOIN Purchasing.PurchaseOrderHeader PurchaseOH
ON PurchaseOD.PurchaseOrderID = PurchaseOH.PurchaseOrderID
WHERE PurchaseOH.Status = 4
```

## Data Transformation
Data transformation is a critical phase in preparing raw data for analysis. This phase encompasses several key processes and involves the creation of calculated tables and field parameters to facilitate the analysis:

### Calculated Tables

1. **Daily Table (Hidden)**:
A calculated table, 'Daily Table,' was generated to span the last 365 days of sales data using the following DAX expression:
`Daily Table = GENERATESERIES((MAX(Sales[OrderDate]) - 365), MAX(Sales[OrderDate]), 1)`
This table serves as a hidden reference for daily time intervals.

2. **Daily Demand Sheet**:
'Daily Demand Sheet' was created by combining 'Daily Table' and 'Product Table' with this DAX expression:
`Daily Demand Sheet = GENERATE('Daily Table', 'Product Table')`

3. **Week Table (Hidden)**:
Another calculated table, 'Week Table,' was generated to cover the last 365 days in weekly intervals. It is hidden from the model and created using the following DAX expression:
`Week Table = GENERATESERIES((MAX(Sales[OrderDate]) - 365), MAX(Sales[OrderDate]), 7)`

4. **Weekly Demand Sheet**:
'Weekly Demand Sheet' was created by combining 'Week Table' and 'Product Table':
`Weekly Demand Sheet = GENERATE('Week Table', 'Product Table')`

5. **DateTable**:
A DateTable was constructed to encompass the entire date range within the sales data, created with this DAX expression:
`DateTable = CALENDAR(MIN(Sales[OrderDate]), MAX(Sales[OrderDate]))`


### Field Parameters
Additionally, two field parameters were introduced to enhance flexibility in the analysis:

1. **Demand Factor**:
A parameter named 'Demand Factor' was defined, offering values ranging from -0.2 to 0.5 in steps of 0.1.

2. **Forecast Days**:
Another parameter named 'Forecast Days' was introduced, providing values ranging from 10 to 100 in increments of 10.

*These calculated tables and field parameters collectively play a pivotal role in the analysis, enabling dynamic exploration of demand factors and flexible forecasting over varying timeframes. They enhance the adaptability and robustness of the inventory management optimization process.*

**Calculating Turnover Rates**: Determining the turnover rates for each product. Turnover rates provide insights into how quickly products are sold and replenished. The formula for turnover rate is:
`Inventory Turnover Ratio = (SUM('Product'[Annual Revenue])/SUM('Product'[Stock value]))`

**Identifying Stockout Occurrences**: Detecting instances when products go out of stock. This involves tracking the transitions between product availability and unavailability. The average time a product remains out of stock is calculated.

**Data Preparation for Predictive Modeling**: Structuring the data in a format suitable for predictive modeling. This includes selecting relevant features and target variables, handling missing values, and encoding categorical variables if needed.

Data transformation is a crucial step to ensure that the data is clean, consistent, and ready for analysis and modeling. It sets the foundation for meaningful insights and accurate predictions in subsequent stages of the project.



![](Transformations.png)

## Data modelling
No model required as its just a single table.

## Analysis and Visualization
The report comprises of 3 pages interact with it [here](https://app.powerbi.com/view?r=eyJrIjoiOWIwMWZiOGItYjBlNy00YmNkLWI4NjItMDA4ZTQ3Y2IzZTFlIiwidCI6ImQ2YjZlODcwLTkwNTgtNDgzOS1hOTJiLTNiNzRhMGI1ODliMSJ9&pageName=ReportSection)
- Homepage 
- Summary
- Content

### Summary 
---
![](SummaryPage.png)

After performing the analysis of the #30DAYSOFLEARNING Twitter hashtag, several key insights have been derived, providing valuable information about the campaign and user engagement. The following insights have been discovered:

**Tweet and User Statistics**: Within the first two months of showcasing the #30days of learning program on Twitter, there have been a total of 674 tweets from 204 different people. This indicates an active participation in the campaign on the platform.

**Increasing Trend**: There has been an exponential increase in the number of tweets over time, particularly in the month of June compared to May. This suggests that the campaign gained momentum and generated greater interest as time progressed.

**Most Active User**: "The Oyinbooke" stands out as the most active user of the hashtag, contributing significantly to the conversation and engagement surrounding the #30DAYSOFLEARNING campaign.

**Device Usage**: A majority (61%) of the users engaging with the hashtag do so using Android devices, indicating a higher prevalence of Android users within the campaign. Conversely, only 9% of the users tweet with iPhones.

**Engagement by Day of the Week**: The analysis reveals that user engagement with the hashtag varies by day of the week. Wednesdays show the highest engagement, while weekends experience the lowest levels of engagement. This suggests that weekdays are more conducive to active participation and interaction with the campaign.

These insights provide stakeholders with valuable information regarding user behavior, platform preferences, and temporal patterns of engagement within the #30DAYSOFLEARNING campaign on Twitter. This understanding can help inform decision-making, content strategy, and campaign optimization efforts to further enhance the impact and success of the campaign.

### Content
---
![](Content.png)
This page provids a deeper understanding of the topics and discussions surrounding the campaign:

**Tool Mentions**: The analysis reveals that GitHub is the most frequently mentioned tool in tweets related to the #30DAYSOFLEARNING campaign, indicating its popularity and relevance within the learning community. On the other hand, Python is the least mentioned tool during the first two months of the program. This insight suggests a stronger association and emphasis on GitHub as a preferred documentation tool for the learning activities.

**Top Liked Tweet**: The user @its_royalgold's tweet garnered the highest number of likes among all the tweets associated with the hashtag. This tweet likely contained compelling content or resonated strongly with the audience, leading to its popularity and high engagement.

**Dominance of @Theoyinbooke**: The facilitator of the program, @TheOyinbooke, appears to have a significant presence within the Top 15 most liked tweets and also was the Most mentioned as shown on the word cloud. This suggests that @TheOyinbooke's contributions, insights, or interactions with the community have been well-received and appreciated by the audience. 

![](MostLiked.png)

## Conclusion:

The analysis of the #30DAYSOFLEARNING Twitter hashtag using Power BI has provided valuable insights into the engagement, reach, and user behavior related to the campaign. The findings indicate active participation in the campaign, with an increasing trend in tweet volume over time. "The Oyinbooke" emerged as the most active user, and Android devices were the most commonly used for engagement. Weekdays, particularly Wednesdays, showed higher levels of engagement compared to weekends. GitHub was the most mentioned tool, and the hashtags #NG30DAYSOFLEARNING and #30DAYSOFLEARNING were the most recurring. Furthermore, specific tweets, including one by @its_royal, received significant likes and engagement.

## Recommendations:

Based on the analysis and insights gained from the #30DAYSOFLEARNING Twitter hashtag, the following recommendations can be made:

**Content Strategy**: Recognize the popularity of GitHub and tailor the campaign content to align with the preferences of the learning community. Emphasize the usage of GitHub as a documentation tool and provide resources, tutorials, and discussions around its effective utilization.

**Mobile Optimization**: Given the majority of users engaging through Android devices, ensure that the campaign's content, visuals, and resources are optimized for mobile viewing and interaction. This includes responsive design, mobile-friendly layouts, and seamless user experiences on Android devices.

**Weekday Focus**: Capitalize on the higher engagement levels observed on weekdays, particularly Wednesdays. Plan and schedule key campaign activities, announcements, and interactive sessions to align with these peak engagement periods to maximize participation and impact.

**Hashtag Utilization**: Maintain the prominence of the hashtags #NG30DAYSOFLEARNING and #30DAYSOFLEARNING in campaign communications, promotions, and discussions. Encourage participants to include these hashtags in their tweets and amplify their usage through engaging and shareable content.

By implementing these recommendations, the #30DAYSOFLEARNING campaign can further enhance its effectiveness, reach, and impact. Continuously monitor and analyze the campaign's performance using tools like Power BI to gather insights and adjust strategies accordingly. This iterative approach will ensure the campaign remains dynamic, engaging, and aligned with the evolving needs and preferences of the learning community.
