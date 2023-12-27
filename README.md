# Consumer-Behavior-Analysis

USE [Project 2: Customer Personality]

--#0a View for the entire database
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

--#0b Try and figure out the current date of the data.
--I've decided to add the days since their last purchase to their first registration date. 
--2014-10-04 is the most current date and hypothetically, I can add a buffer, but I will choose this date to make it simple. 
```sql
SELECT [ID]
      ,[Year_Birth]
      ,[Generation]
      ,[Dt_Customer]
      ,[Recency]
      ,DATEADD(day, Recency, Dt_Customer) AS 'Current Date'
FROM [marketing_campaign]
ORDER BY 'Current Date' DESC
```

--#1 Insert a new column to classify customers into age/generation brackets. 

--Used this query to insert a blank column that I will use to fill out. 
ALTER TABLE dbo.marketing_campaign
ADD Generation varchar(255)

--Used this query to define the generation's brackets and insert them into the new column. There are 3 ID's with ages over 100, so I am going to omit them by leaving them as NULL.
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
       ,[Ag]e = 2023 - Year_Birth
--update marketing_campaign SET Generation = CASE WHEN Year_Birth >= 1997 AND Year_Birth <= 2012 THEN 'Gen Z'
                            WHEN [Year_Birth] >= 1981 AND [Year_Birth] <= 1996 THEN 'Millennials'
                            WHEN [Year_Birth] >= 1965 AND [Year_Birth] <= 1980 THEN 'Gen X'
                            WHEN [Year_Birth] >= 1946 AND [Year_Birth] <= 1964 THEN 'Boomers'
                            WHEN [Year_Birth] >= 1928 AND [Year_Birth] <= 1945 THEN 'Silent'
                            ELSE NULL
                            END
FROM [marketing_campaign]
ORDER BY [Age]
```

--#2 Where do our customers like to make their purchases from?
--Our customers like to purchase in store the most. Followed up by web purchases and lastly catalog purchases.
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

--#3 What kind of products do our customers like to make?
--Used this query to get the average and sum of product purchases grouped by age generations and by the total.
--We can see that in order from most ot least is Wine, Meat, Gold, Fish, Sweet, and Fruit.
--Not sure what kind of store this is, but the distribution is interesting as you can purchase groceries, but also gold.
--My guess, it could be like a wholesale store like Costco or Sam's Club.
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

--#4 Which customers make the most purchases?
--Used this query to see that Gen X and Boomers make the most purchases at this store. Millennials are much smaller, but can definitely climb up as time goes on.
--The Silent generation is the smallest segment however. 
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

--#5 Which of our customer segments are more susceptible to making purchases that are on sale?
--Since we know that Gen X and Boomers make the most purchases, it does make sense why they would have the most deal purchases.
--However, on average, they are about the same. Therefore, I believe the deals are working effectively for all customers.
--A deeper level to analyze this further would be to use weights to make each generation the same size. 
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

--#6 Which of our customer segments are customers more susceptible to our marketing camapigns?
--The Silent generation has the highest rate of accepting campaigns, but that is not reliable as there are only 24 of them.
--Moving from highest to lowest, we have Millennials, Boomers, and Gen X. However, overall it is at 7.45%.
--Diving into the campaigns, Campaign #2 was the worst one by far. 1, 3, 4, and 5 are pretty good at 6-7%. However, Campaign 6 really stood out with almost 15%.
--Therefore, this store should continue to create campaigns like #6 as they would have the best results. 
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

--#7a How many customers have made a purchase wwithin 1 month, 3 months, and 3-12 months?
--2014-10-04 is the current date

--With this query, we can see that a large majority of customers have made a purchase in the past 3 months. This is great to see.
--Let's dive further into those who have not made a purchase in over 3 months. 
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

--#7b Analyze the customers who have not made a purchase in over 3 months and provide some findings.

--First, this is the entire list of customers who have not made purchases in 3-6 Months.
--This list can be used to send them emails with promotions to get them back. 
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

--Used this query to find out that only 4 people made complaints, so I would say complaints is not the main factor for those who stopped making purchases. 
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

--Using this query, we can see the level of the customers' membership seniority, which is calculated by subtracting their last purchase date to their first registration date.
--About half of the customers have been members for over a year, so that seems ok. 
--The other half includes newer customers and there is a possibiltiy the store was not right for them to purchase items monthly.
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

--Using this query, we get a breakdown of the customers' membership seniority who's last purchase was 3-6 months ago.
--Overall, there is not sufficient data to determine why customers have not made purchases in a long time.
--Ideally, we would have their entire purchase history, so we can develop patterns to base off of.
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

--#8 Analyze why the customers have made complaints. To preface this, we previously learned that complaints were not the main cause for customers to stop making purchases in 3 months. 

--First, I found that only 21 customers made complaints before. In general, it seems like customers are having a good time.
```sql
SELECT Complain 
       ,COUNT(*) AS 'Number of Customers'
FROM [marketing_campaign]
GROUP BY Complain
```

--Using this query we can see that on average, those who made complaints bought fewer items, but that is also because our base sizes are low.
--We can also see that on average, they have the same location purchasing preferences. Therefore, where they make their purchase is less likely to affect their experience.
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


--Using this query, we can see that Boomers have made the most complaints so far.
--And unfortunately, with the limited data, it would be tough to figure out the reasons why for these complaints.
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




--#X Create customer segments. Find the percentages of them
    --Loyal customers
    --Occasional customers
    --One off customers

--Customer Demographics
