# Consumer-Behavior-Analysis

## 📚 Background About the Project and Data

The goal for this project was to only use SQL to write an analysis. I found this "Customer Personality" dataset from Kaggle, which was last updated 2 years ago. The data is somewhat limited, so I tried to do as much as possible.

Dataset (Kaggle): [Link](https://www.kaggle.com/datasets/imakash3011/customer-personality-analysis)

## 💡 Highlights
- From most to least purchases are Wine, Meat, Gold, Fish, Sweet, and Fruit.
- Gen X and Boomers make the most purchases at this store. However, a caveat to this is that the base sizes for those two groups are the largest.
- Gen X and Boomers make the most deal purchases, but that is also because they are the largest customer bases for this store. However, on average, all generations make about the same deal purchases at an average of 3
- On average, customers accept marketing campaigns at 7.45%. A major finding is that campaign #2 is the weakest and campaign #6 is the strongest. Therefore, this store should create more campaigns like #6.
- A large majority of customers have made purchases in the past 3 months. With the limited depth of this data, we unfortunately do not know why customers have not made purchases in over 3 months.
- Boomers have made the most complaints so far, but it's not too far off of the other generations. A nice way to revisit this is to calculate the rate at which each generation makes complaints.

## 📄 SQL Code and Analysis

### #0a View for the entire database
```sql
SELECT [ID]
      ,[Year_Birth]
      ,[Generation]
      ,[Education]
      ,[Marital_Status]
      ,[Income]
      ,[Kidhome]
      ,[Teenhome]
      ,[Dt_Customer]
      ,[Recency]
      ,[MntWines]
      ,[MntFruits]
      ,[MntMeatProducts]
      ,[MntFishProducts]
      ,[MntSweetProducts]
      ,[MntGoldProds]
      ,[NumDealsPurchases]
      ,[NumWebPurchases]
      ,[NumCatalogPurchases]
      ,[NumStorePurchases]
      ,[NumWebVisitsMonth]
      ,[AcceptedCmp1]
      ,[AcceptedCmp2]
      ,[AcceptedCmp3]
      ,[AcceptedCmp4]
      ,[AcceptedCmp5]
      ,[Complain]
      ,[Z_CostContact]
      ,[Z_Revenue]
      ,[Response]
FROM [marketing_campaign]
```

### #0b Figure out a hypothetical date of when the data was last recorded.
- As I worked on this project, I found out I needed a specific date and because the dataset is a little limited on context, I had to find a way to get a hypothetical current date.
- I've decided to add the days since their last purchase to their first registration date.
- 2014-10-04 is the most current date and I can technically add a buffer, but I chose this date for simplicity.
```sql
SELECT [ID]
      ,[Year_Birth]
      ,[Generation]
      ,[Dt_Customer]
      ,[Recency]
      ,DATEADD(day, Recency, Dt_Customer) AS 'Current Date'
FROM [marketing_campaign]
ORDER BY [Current Date] DESC
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/395dd498-b315-4e2a-8819-b2c1e0c90f8f)

### #1 Insert a new column to classify customers into age/generation brackets. 
- Used this query to insert a blank column that I will use to fill out. 
ALTER TABLE dbo.marketing_campaign
ADD Generation varchar(255)

- Used this query to define the generation's brackets and inserted them into the new column. There are 3 ID's with ages over 100, and because the base size is so small, I just left them as NULL.
```sql
SELECT ID
       ,[Year_Birth]
       ,[Generation] = CASE WHEN [Year_Birth] >= 1997 AND [Year_Birth] <= 2012 THEN 'Gen Z'
                            WHEN [Year_Birth] >= 1981 AND [Year_Birth] <= 1996 THEN 'Millennials'
                            WHEN [Year_Birth] >= 1965 AND [Year_Birth] <= 1980 THEN 'Gen X'
                            WHEN [Year_Birth] >= 1946 AND [Year_Birth] <= 1964 THEN 'Boomers'
                            WHEN [Year_Birth] >= 1928 AND [Year_Birth] <= 1945 THEN 'Silent'
                            ELSE NULL
                            END
       ,[Age] = 2023 - Year_Birth
- update marketing_campaign SET Generation = CASE WHEN Year_Birth >= 1997 AND Year_Birth <= 2012 THEN 'Gen Z'
                            WHEN [Year_Birth] >= 1981 AND [Year_Birth] <= 1996 THEN 'Millennials'
                            WHEN [Year_Birth] >= 1965 AND [Year_Birth] <= 1980 THEN 'Gen X'
                            WHEN [Year_Birth] >= 1946 AND [Year_Birth] <= 1964 THEN 'Boomers'
                            WHEN [Year_Birth] >= 1928 AND [Year_Birth] <= 1945 THEN 'Silent'
                            ELSE NULL
                            END
FROM [marketing_campaign]
ORDER BY [Age]
```

### #2 Where do customers like to make their purchases from?
- Customers like to purchase in store the most. Followed up by web purchases and lastly catalog purchases.
```sql
SELECT [Index] = CASE WHEN [Generation] = 'Gen Z' THEN 1
                      WHEN [Generation] = 'Millennials' THEN 2
                      WHEN [Generation] = 'Gen X' THEN 3
                      WHEN [Generation] = 'Boomers' THEN 4
                      WHEN [Generation] = 'Silent' THEN 5
                      ELSE NULL
                      END
       ,[Generation]
       ,COUNT(*) AS 'Count'
       ,AVG(2023 - Year_Birth) AS 'AVG Age'
       ,SUM(NumCatalogPurchases) AS 'SUM Catalog Purchases'
       ,AVG(NumCatalogPurchases) AS 'AVG Catalog Purchases'
       ,SUM(NumStorePurchases) AS 'SUM Store Purchases'
       ,AVG(NumStorePurchases) AS 'AVG Store Purchases'
       ,SUM(NumWebPurchases) AS 'SUM Web Purchases'
       ,AVG(NumWebPurchases) AS 'AVG Web Purchases'
       ,SUM(NumWebVisitsMonth) AS 'SUM WebVisitsMonth'
       ,AVG(NumWebVisitsMonth) AS 'AVG WebVisitsMonth'
FROM [marketing_campaign]
WHERE [Generation] IS NOT NULL
GROUP BY Generation
UNION ALL 
SELECT [Index] = 6
       ,[Generation] = '*Total*'
       ,COUNT(*) AS 'Count'
       ,AVG(2023 - Year_Birth) AS 'AVG Age'
       ,SUM(NumCatalogPurchases) AS 'SUM Catalog Purchases'
       ,AVG(NumCatalogPurchases) AS 'AVG Catalog Purchases'
       ,SUM(NumStorePurchases) AS 'SUM Store Purchases'
       ,AVG(NumStorePurchases) AS 'AVG Store Purchases'
       ,SUM(NumWebPurchases) AS 'SUM Web Purchases'
       ,AVG(NumWebPurchases) AS 'AVG Web Purchases'
       ,SUM(NumWebVisitsMonth) AS 'SUM WebVisitsMonth'
       ,AVG(NumWebVisitsMonth) AS 'AVG WebVisitsMonth'
FROM [marketing_campaign]
WHERE [Generation] IS NOT NULL
ORDER BY [Index] 
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/82d60aa0-a529-44b7-be21-a0ab197d86e2)


### #3 What kind of products do customers like to make?
- Used this query to get the average and sum of product purchases grouped by age generations and by the total.
- We can see that in order from most to least is Wine, Meat, Gold, Fish, Sweet, and Fruit.
- Not sure what kind of store this is, but the distribution is interesting as you can purchase groceries, but also gold.
- My guess, is it could be similar to a wholesale store like Costco or Sam's Club.
```sql
SELECT [Index] = CASE WHEN [Generation] = 'Gen Z' THEN 1
                      WHEN [Generation] = 'Millennials' THEN 2
                      WHEN [Generation] = 'Gen X' THEN 3
                      WHEN [Generation] = 'Boomers' THEN 4
                      WHEN [Generation] = 'Silent' THEN 5
                      ELSE NULL
                      END
       ,[Generation]
       ,COUNT(*) AS 'Count'
       ,AVG(2023 - Year_Birth) AS 'AVG Age'
       ,SUM([MntWines]) AS 'SUM Wine Purchases'
       ,AVG([MntWines]) AS 'AVG Wine Purchases'
       ,SUM([MntFruits]) AS 'SUM Fruit Purchases'
       ,AVG([MntFruits]) AS 'AVG Fruit Purchases'
       ,SUM([MntMeatProducts]) AS 'SUM Meat Purchases'
       ,AVG([MntMeatProducts]) AS 'AVG Meat Purchases'
       ,SUM([MntFishProducts]) AS 'SUM Fish Purchases'
       ,AVG([MntFishProducts]) AS 'AVG Fish Purchases'
       ,SUM([MntSweetProducts]) AS 'SUM Sweet Purchases'
       ,AVG([MntSweetProducts]) AS 'AVG Sweet Purchases'
       ,SUM([MntGoldProds]) AS 'SUM Gold Purchases'
       ,AVG([MntGoldProds]) AS 'AVG Gold Purchases'
FROM [marketing_campaign]
WHERE Generation IS NOT NULL
GROUP BY [Generation]
UNION ALL
SELECT [Index] = 6
       ,[Generation] = '*Total*'
       ,COUNT(*) AS 'Count'
       ,AVG(2023 - Year_Birth) AS 'AVG Age'
       ,SUM([MntWines]) AS 'SUM Wine Purchases'
       ,AVG([MntWines]) AS 'AVG Wine Purchases'
       ,SUM([MntFruits]) AS 'SUM Fruit Purchases'
       ,AVG([MntFruits]) AS 'AVG Fruit Purchases'
       ,SUM([MntMeatProducts]) AS 'SUM Meat Purchases'
       ,AVG([MntMeatProducts]) AS 'AVG Meat Purchases'
       ,SUM([MntFishProducts]) AS 'SUM Fish Purchases'
       ,AVG([MntFishProducts]) AS 'AVG Fish Purchases'
       ,SUM([MntSweetProducts]) AS 'SUM Sweet Purchases'
       ,AVG([MntSweetProducts]) AS 'AVG Sweet Purchases'
       ,SUM([MntGoldProds]) AS 'SUM Gold Purchases'
       ,AVG([MntGoldProds]) AS 'AVG Gold Purchases'
FROM [marketing_campaign]
WHERE Generation IS NOT NULL
ORDER BY [Index]
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/ecd1dd29-5f07-40f5-a1e0-76597b2fb542)

### #4 Which customer group makes the most purchases?
- Used this query to see that Gen X and Boomers make the most purchases at this store. Millennials are much smaller, but can definitely climb up as time goes on.
- However, the Silent generation is the smallest segment.
- A minor flaw in this finding is that the base sizes for each generation are different. It currently lines up with the idea that the more people there are, the more sales there would be.
- Reflecting on this, we can calculate an average of sales per customer for each generation, which would give us a more accurate conclusion.
```sql
SELECT [Index] = CASE WHEN [Generation] = 'Gen Z' THEN 1
                      WHEN [Generation] = 'Millennials' THEN 2
                      WHEN [Generation] = 'Gen X' THEN 3
                      WHEN [Generation] = 'Boomers' THEN 4
                      WHEN [Generation] = 'Silent' THEN 5
                      ELSE NULL
                      END
       ,[Generation]
       ,COUNT(*) AS 'Count'
       ,AVG(2023 - [Year_Birth]) AS 'AVG Age'
       ,[Total] = SUM([MntWines]) 
                + SUM([MntFruits])
                + SUM([MntMeatProducts]) 
                + SUM([MntFishProducts])
                + SUM([MntSweetProducts])
                + SUM([MntGoldProds])                      
       ,SUM([MntWines]) AS 'SUM Wine Purchases'
       ,AVG([MntWines]) AS 'AVG Wine Purchases'
       ,SUM([MntFruits]) AS 'SUM Fruit Purchases'
       ,AVG([MntFruits]) AS 'AVG Fruit Purchases'
       ,SUM([MntMeatProducts]) AS 'SUM Meat Purchases'
       ,AVG([MntMeatProducts]) AS 'AVG Meat Purchases'
       ,SUM([MntFishProducts]) AS 'SUM Fish Purchases'
       ,AVG([MntFishProducts]) AS 'AVG Fish Purchases'
       ,SUM([MntSweetProducts]) AS 'SUM Sweet Purchases'
       ,AVG([MntSweetProducts]) AS 'AVG Sweet Purchases'
       ,SUM([MntGoldProds]) AS 'SUM Gold Purchases'
       ,AVG([MntGoldProds]) AS 'AVG Gold Purchases'
FROM [marketing_campaign]
WHERE [Generation] IS NOT NULL
GROUP BY [Generation]
ORDER BY [Total] DESC
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/0b769107-04b5-40ed-ae92-4dba1bcbbad8)

### #5 Which customer segment is more susceptible to making purchases that are on sale?
- Since we know that Gen X and Boomers make the most purchases, it does make sense why they would have the most deal purchases.
- However, on average, they are about the same. Therefore, I believe the deals are working effectively for all customers.
- A deeper level to analyze this further would be to use weights to make each generation the same size. 
```sql
SELECT [Index] = CASE WHEN [Generation] = 'Gen Z' THEN 1
                      WHEN [Generation] = 'Millennials' THEN 2
                      WHEN [Generation] = 'Gen X' THEN 3
                      WHEN [Generation] = 'Boomers' THEN 4
                      WHEN [Generation] = 'Silent' THEN 5
                      ELSE NULL
                      END
      ,[Generation]
      ,SUM([NumDealsPurchases]) AS 'SUM Deal Purchases'
      ,AVG([NumDealsPurchases]) AS 'AVG Deal Purchases'
FROM [marketing_campaign]
WHERE [NumDealsPurchases] > 1
GROUP BY [Generation]
ORDER BY [Index]
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/53adce58-fdf8-448f-a20c-fb814fd90e96)

### #6 Which customer segment is more susceptible to marketing campaigns?
- The Silent Generation has the highest rate of accepting campaigns, but that is not reliable as there are only 24 of them.
- Moving from highest to lowest, we have Millennials, Boomers, and Gen X. However, overall it is at 7.45%.
- Diving into the campaigns, Campaign #2 was the worst one by far. 1, 3, 4, and 5 are pretty good at 6-7%. However, Campaign 6 really stood out with almost 15%.
- Therefore, this store should continue to create campaigns like #6 as they would have the best results. 
```sql
SELECT [Generation]
       ,COUNT([Generation]) AS [Count]
       ,SUM([AcceptedCmp1]) AS [Accepted Cmp1]
       ,SUM([AcceptedCmp2]) AS [Accepted Cmp2]
       ,SUM([AcceptedCmp3]) AS [Accepted Cmp3]
       ,SUM([AcceptedCmp4]) AS [Accepted Cmp4]
       ,SUM([AcceptedCmp5]) AS [Accepted Cmp5]
       ,SUM([Response]    ) AS [Accepted Cmp6]    
       ,SUM([AcceptedCmp1] + [AcceptedCmp2] + [AcceptedCmp3] + [AcceptedCmp4] + [AcceptedCmp5] + [Response]) AS 'SUM of Accepted Cmps'
       ,CONCAT(ROUND(AVG((([AcceptedCmp1] + [AcceptedCmp2] + [AcceptedCmp3] + [AcceptedCmp4] + [AcceptedCmp5] + CAST([Response]AS FLOAT)) / 6)*100), 2), '%') AS 'AVG Rate of Cmp Acceptance'
FROM [marketing_campaign]
WHERE [Generation] IS NOT NULL
GROUP BY [Generation]
UNION ALL 
SELECT [Generation] = '*Total/%*'
       ,COUNT([Generation]) AS [Count]
       ,ROUND(((SUM(CAST([AcceptedCmp1] AS FLOAT))/COUNT(*))*100),2) AS [Accepted Cmp1]
       ,ROUND(((SUM(CAST([AcceptedCmp2] AS FLOAT))/COUNT(*))*100),2) AS [Accepted Cmp2]
       ,ROUND(((SUM(CAST([AcceptedCmp3] AS FLOAT))/COUNT(*))*100),2) AS [Accepted Cmp3]
       ,ROUND(((SUM(CAST([AcceptedCmp4] AS FLOAT))/COUNT(*))*100),2) AS [Accepted Cmp4]
       ,ROUND(((SUM(CAST([AcceptedCmp5] AS FLOAT))/COUNT(*))*100),2) AS [Accepted Cmp5]
       ,ROUND(((SUM(CAST([Response]     AS FLOAT))/COUNT(*))*100),2) AS [Accepted Cmp6]
       ,ROUND(((SUM([AcceptedCmp1] + [AcceptedCmp2] + [AcceptedCmp3] + [AcceptedCmp4] + [AcceptedCmp5] + CAST([Response] AS FLOAT))/COUNT(*))*100),2) AS 'SUM of Accepted Cmps'
       ,CONCAT(ROUND(AVG((([AcceptedCmp1] + [AcceptedCmp2] + [AcceptedCmp3] + [AcceptedCmp4] + [AcceptedCmp5] + CAST([Response]AS FLOAT)) / 6)*100), 2), '%') AS 'AVG Rate of Cmp Acceptance'
FROM [marketing_campaign]
WHERE [Generation] IS NOT NULL
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/636a56ae-aef9-4f0c-9d7a-7adeb48fa15c)

### #7a How many customers have made a purchase within 1 month, 3 months, 3-6 months, and 7-12 months?
- Basing this off of 2014-10-04 as the current date.
- With this query, we can see that a large majority of customers have made a purchase in the past 3 months. This is great to see as that means we have a good base of loyal customers.
- Let's dive further into those who have not made a purchase in over 3 months. 
```sql
SELECT [Purchased]
       ,COUNT(*) AS 'Number of Purchases'
FROM (
     SELECT [ID]
           ,[Generation]
           ,[Dt_Customer]
           ,[Recency]
           ,CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE) AS [Last Purchase Date]
           ,ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) AS [Seniority]
           ,[Purchased] = CASE WHEN Recency <= 30 THEN '1 Month'
                           WHEN Recency > 30 AND Recency <= 90 THEN '3 Months'
                           WHEN Recency > 90 AND Recency <= 180 THEN '3-6 Months'
                           WHEN Recency > 180 AND Recency < 365 THEN '7-12 Months'
                           WHEN Recency >= 365 THEN 'Over a year ago'
                           ELSE NULL
                           END
     FROM [marketing_campaign] 
     ) [marketing_campaign mod] 
GROUP BY [Purchased]
ORDER BY [Purchased]
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/39bf13c9-fae3-4041-8f26-7d7cfc394428)

### #7b Analyze the customers who have not made a purchase in over 3 months and provide some findings.
- First, this is the entire list of customers who have not made purchases in 3-6 Months.
- This list can be used to send them emails with promotions to get them back. 
```sql
SELECT *
FROM (
     SELECT [ID]
           ,[Generation]
           ,[Dt_Customer]
           ,[Recency]
           ,CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE) AS [Last Purchase Date]
           ,ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) AS [Seniority]
           ,[Purchased] = CASE WHEN Recency <= 30 THEN '1 Month'
                           WHEN Recency > 30 AND Recency <= 90 THEN '3 Months'
                           WHEN Recency > 90 AND Recency <= 180 THEN '3-6 Months'
                           WHEN Recency > 180 AND Recency < 365 THEN '7-12 Months'
                           WHEN Recency >= 365 THEN 'Over a year ago'
                           ELSE NULL
                           END
           ,[Complain]
     FROM [marketing_campaign] 
     ) [marketing_campaign mod] 
WHERE [Purchased]  = '3-6 Months'
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/17ea808f-f463-4065-9c5c-0e942bc2199c)

- Used this query to find out that only 4 people made complaints, so I would say complaints are not the main factor for those who stopped making purchases. 
```sql
SELECT [Complain]
      ,COUNT(*) AS 'Number of Customers'
FROM (
     SELECT [ID]
           ,[Generation]
           ,[Dt_Customer]
           ,[Recency]
           ,CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE) AS [Last Purchase Date]
           ,ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) AS [Seniority]
           ,[Purchased] = CASE WHEN Recency <= 30 THEN '1 Month'
                           WHEN Recency > 30 AND Recency <= 90 THEN '3 Months'
                           WHEN Recency > 90 AND Recency <= 180 THEN '3-6 Months'
                           WHEN Recency > 180 AND Recency < 365 THEN '7-12 Months'
                           WHEN Recency >= 365 THEN 'Over a year ago'
                           ELSE NULL
                           END
           ,[Complain]
     FROM [marketing_campaign] 
     ) [marketing_campaign mod] 
WHERE [Purchased]  = '3-6 Months'
GROUP BY [Complain]
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/fb1b7634-1220-47e5-8849-d79908a2af5a)

- Used this query to see the level of the customers' membership seniority, which is calculated by subtracting their last purchase date from their first registration date.
- About half of the customers have been members for over a year, so that seems okay. 
- The other half includes newer customers and there is a possibility the store was not right for them to purchase items monthly.
```sql
SELECT *
FROM (
    SELECT [ID]
              ,[Generation]
              ,[Dt_Customer]
              ,[Recency]
              ,CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE) AS [Last Purchase Date]
              ,ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) AS [Seniority]
              ,[Purchased] = CASE WHEN Recency <= 30 THEN '1 Month'
                              WHEN Recency > 30 AND Recency <= 90 THEN '3 Months'
                              WHEN Recency > 90 AND Recency <= 180 THEN '3-6 Months'
                              WHEN Recency > 180 AND Recency < 365 THEN '7-12 Months'
                              WHEN Recency >= 365 THEN 'Over a year ago'
                              ELSE NULL
                              END
    FROM [marketing_campaign]) [M]
WHERE [Purchased]  = '3-6 Months'
ORDER BY [Seniority] DESC
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/115ff4cc-dda2-43c3-919c-f0f8cac86e24)

- Used this query to get a breakdown of the customers' membership seniority whose last purchase was 3-6 months ago.
- Overall, there is not sufficient data to determine why customers have not made purchases in a long time.
- Ideally, we would have their entire purchase history, so we can develop patterns to base off of.
```sql
SELECT [Seniority Bracket]
      ,COUNT(*) AS [Count]
FROM 
   (SELECT [ID]
          ,[Generation]
          ,[Dt_Customer]
          ,[Recency]
          ,CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE) AS [Last Purchase Date]
          ,ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) AS [Seniority]
          ,[Purchased] = CASE WHEN Recency <= 30 THEN '1 Month'
                          WHEN Recency > 30 AND Recency <= 90 THEN '3 Months'
                          WHEN Recency > 90 AND Recency <= 180 THEN '3-6 Months'
                          WHEN Recency > 180 AND Recency < 365 THEN '7-12 Months'
                          WHEN Recency >= 365 THEN 'Over a year ago'
                          ELSE NULL
                          END
         ,[Seniority Bracket] = CASE WHEN ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) <= 30 THEN '1 Month'
                                     WHEN ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) > 30 AND ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) <= 90 THEN '3 Months'
                                     WHEN ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) > 90 AND ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) <= 180 THEN '3-6 Months'
                                     WHEN ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) > 180 AND ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) < 365 THEN '7-12 Months'
                                     WHEN ABS(DATEDIFF(day, CAST(DATEADD(day, -Recency, '2014-10-04') AS DATE), Dt_customer)) >= 365 THEN 'Over a year'
                                     ELSE NULL
                                     END
    FROM [marketing_campaign]) [M]
WHERE [Purchased]  = '3-6 Months'
GROUP BY [Seniority Bracket]
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/83868996-6d66-45fc-8132-f3b161724070)

### #8 Analyze why the customers have made complaints. To preface this, we previously learned that complaints were not the main cause for customers to stop making purchases in 3 months. 
- First, I found that only 21 customers made complaints before. In general, it seems like customers are having a good time.
```sql
SELECT Complain 
       ,COUNT(*) AS 'Number of Customers'
FROM [marketing_campaign]
GROUP BY Complain
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/3ce0551a-e06f-489f-8f0a-991c90ee4219)

- Used this query to see that on average, those who made complaints bought fewer items, but that is also because our base sizes are low.
- We can also see that on average, they have the same location purchasing preferences. Therefore, where they make their purchase is less likely to affect their experience.
```sql
SELECT Complain
       ,COUNT(*) AS 'Number of Customers'
       ,AVG([MntWines]) AS 'AVG Wine Purch'
       ,AVG([MntFruits]) AS 'AVG Fruits Purch'
       ,AVG([MntMeatProducts]) AS 'AVG Meats Purch'
       ,AVG([MntFishProducts]) AS 'AVG Fishs Purch'
       ,AVG([MntSweetProducts]) AS 'AVG Sweets Purch'
       ,AVG([MntGoldProds]) AS 'AVG Gold Purch'
FROM [marketing_campaign]
GROUP BY Complain
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/8dc98cfd-5be8-4b14-a633-7adc44e8fa3d)

```sql
SELECT Complain
       ,COUNT(*) AS 'Number of Customers'
       ,SUM(NumCatalogPurchases) AS 'SUM Catalog Purchases'
       ,AVG(NumCatalogPurchases) AS 'AVG Catalog Purchases'
       ,SUM(NumStorePurchases) AS 'SUM Store Purchases'
       ,AVG(NumStorePurchases) AS 'AVG Store Purchases'
       ,SUM(NumWebPurchases) AS 'SUM Web Purchases'
       ,AVG(NumWebPurchases) AS 'AVG Web Purchases'
       ,SUM(NumWebVisitsMonth) AS 'SUM WebVisitsMonth'
       ,AVG(NumWebVisitsMonth) AS 'AVG WebVisitsMonth'
FROM [marketing_campaign]
GROUP BY Complain
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/91e33606-6610-400f-8639-50028d5e4dba)

- Used this query to see that Boomers have made the most complaints so far.
- And unfortunately, with the limited data, it would be tough to figure out the reasons why for these complaints.
```sql
SELECT [Index] = CASE WHEN [Generation] = 'Gen Z' THEN 1
                      WHEN [Generation] = 'Millennials' THEN 2
                      WHEN [Generation] = 'Gen X' THEN 3
                      WHEN [Generation] = 'Boomers' THEN 4
                      WHEN [Generation] = 'Silent' THEN 5
                      ELSE NULL
                      END
      ,[Generation]
      ,COUNT([Complain]) AS 'Count'
FROM [marketing_campaign]
WHERE [Complain] = 1
      AND [Generation] IS NOT NULL
GROUP BY [Generation]
ORDER BY [Index]
```
![image](https://github.com/davidwong001/Consumer-Behavior-Analysis/assets/146798360/3838f875-3633-47c8-b130-540dd5e8d546)
