# Finding the Ideal Customer Through Language Filters

## Goal

A German tech start-up was on their mission to revolutionize the part-time job market. First step: Find their ideal customer: whether it’s students aiming to pay off loans, or moonlighting professionals aiming for financial security. They all share eligibility for short-term employment of up to 70 working days or 3 months per calendar year without mandatory social security contributions, provided it is not the employee’s main occupation.

## Hypothesis

Colleagues from the customer service department were wondering why so many customers spoke English instead of German, as the company's B2C app was originally designed for German students. Therefore I decided to take a look into the spoken languages that customers set during the sign-up process and also their country of origin. My hypothesis: People from countries other than Germany in general completed the sign-up process more frequently (1), worked more (2), were more satisfied and thus more likely to book another part-time job (3). 

## Process

**Tools:** MariaDB, Tableau, Excel

Data from the B2C app was stored in an AWS server, accessible via MariaDB and Tableau.

### Hypothesis 1: Non-German Customers Book More Jobs and Work More Hours

**Step 1:** Calculate average worked hours per day for non-German-speaking customers, grouped by month.

```sql
SELECT 
    c.Language, 
    EXTRACT(YEAR FROM cwd.Date) AS Year,
    EXTRACT(MONTH FROM cwd.Date) AS Month,
    AVG(cwd.Hours) AS AvgHours_Per_Day
FROM Customers c
JOIN CustomerWorkDays cwd ON c.CustomerID = cwd.CustomerID
WHERE c.Language <> 'German'
GROUP BY c.Language, EXTRACT(YEAR FROM cwd.Date), EXTRACT(MONTH FROM cwd.Date)
ORDER BY c.Language, Year, Month;
```

**Step 2:** Calculate average worked hours per day for German-speaking customers, grouped by month.

```sql
SELECT 
    'German' AS Language,
    EXTRACT(YEAR FROM cwd.Date) AS Year,
    EXTRACT(MONTH FROM cwd.Date) AS Month,
    AVG(cwd.Hours) AS AvgHours_Per_Day
FROM Customers c
JOIN CustomerWorkDays cwd ON c.CustomerID = cwd.CustomerID
WHERE c.Language = 'German'
GROUP BY EXTRACT(YEAR FROM cwd.Date), EXTRACT(MONTH FROM cwd.Date)
ORDER BY Year, Month;
```

**Step 3:** Combine the results to compare average worked hours per day between German-speaking and non-German-speaking customers, grouped by month.

```sql
SELECT 
    Combined.Language,
    Combined.Year,
    Combined.Month,
    Combined.AvgHours_Per_Day
FROM (
    SELECT 
        c.Language, 
        EXTRACT(YEAR FROM cwd.Date) AS Year,
        EXTRACT(MONTH FROM cwd.Date) AS Month,
        AVG(cwd.Hours) AS AvgHours_Per_Day
    FROM Customers c
    JOIN CustomerWorkDays cwd ON c.CustomerID = cwd.CustomerID
    WHERE c.Language <> 'German'
    GROUP BY c.Language, EXTRACT(YEAR FROM cwd.Date), EXTRACT(MONTH FROM cwd.Date)
    
    UNION ALL
    
    SELECT 
        'German' AS Language,
        EXTRACT(YEAR FROM cwd.Date) AS Year,
        EXTRACT(MONTH FROM cwd.Date) AS Month,
        AVG(cwd.Hours) AS AvgHours_Per_Day
    FROM Customers c
    JOIN CustomerWorkDays cwd ON c.CustomerID = cwd.CustomerID
    WHERE c.Language = 'German'
    GROUP BY EXTRACT(YEAR FROM cwd.Date), EXTRACT(MONTH FROM cwd.Date)
) AS Combined
ORDER BY Combined.Language, Combined.Year, Combined.Month;
```

### Hypothesis 2: Non-Germans Are More Likely to Finish the Onboarding Process

**Step 1:** Calculate completion rates for each onboarding step for non-German-speaking customers.

```sql
SELECT 
    'Non-German' AS CustomerGroup,
    AVG(o.Completed_1) AS CompletionRate_Completed_1,
    AVG(o.Completed_2) AS CompletionRate_Completed_2,
    AVG(o.Completed_3) AS CompletionRate_Completed_3,
    AVG(o.Onboarded) AS CompletionRate_Onboarded
FROM Customers c
JOIN Onboarding o ON c.CustomerID = o.CustomerID
WHERE c.Language <> 'German';
```

**Step 2:** Calculate completion rates for each onboarding step for German-speaking customers.

```sql
SELECT 
    'German' AS CustomerGroup,
    AVG(o.Completed_1) AS CompletionRate_Completed_1,
    AVG(o.Completed_2) AS CompletionRate_Completed_2,
    AVG(o.Completed_3) AS CompletionRate_Completed_3,
    AVG(o.Onboarded) AS CompletionRate_Onboarded
FROM Customers c
JOIN Onboarding o ON c.CustomerID = o.CustomerID
WHERE c.Language = 'German';
```

**Step 3:** Combine the results to compare the completion rates between German-speaking and non-German-speaking customers.

```sql
SELECT * FROM (
    SELECT 
        'Non-German' AS CustomerGroup,
        AVG(o.Completed_1) AS CompletionRate_Completed_1,
        AVG(o.Completed_2) AS CompletionRate_Completed_2,
        AVG(o.Completed_3) AS CompletionRate_Completed_3,
        AVG(o.Onboarded) AS CompletionRate_Onboarded
    FROM Customers c
    JOIN Onboarding o ON c.CustomerID = o.CustomerID
    WHERE c.Language <> 'German'
    
    UNION ALL
    
    SELECT 
        'German' AS CustomerGroup,
        AVG(o.Completed_1) AS CompletionRate_Completed_1,
        AVG(o.Completed_2) AS CompletionRate_Completed_2,
        AVG(o.Completed_3) AS CompletionRate_Completed_3,
        AVG(o.Onboarded) AS CompletionRate_Onboarded
    FROM Customers c
    JOIN Onboarding o ON c.CustomerID = o.CustomerID
    WHERE c.Language = 'German'
) AS CompletionRates;
```

### Hypothesis 3: Non-German Customers Are More Satisfied and More Likely to Book Jobs Again

As “Customer Happiness Manager”, I gathered first-hand qualitative insights and NPS values to measure customer satisfaction on a monthly basis. Customer input was received by a third party iframe integration into the company app. I stored all incoming values in an automated Excel sheet. Through a VLOOKUP formula, I could join the CustomerID field with the analysis that I conducted earlier to compare NPS values between Germans and non-Germans. 


```
=VLOOKUP(Customers[CustomerID], 
         AnalysisData[CustomerID], 
         AnalysisData[NPS],2, 
         FALSE)
```

## Success

(Due to data protection reasons, I will only use percentage values instead of absolute numbers)

**Hypothesis 1:** Customers without a German language tag worked 35% more than German customers, which is more than significant, facing the fact that the app did not yet have an English version or sufficient English-speaking customer agents. Later, we found out that they also worked more frequently and more hours per day on average.
![Bildschirmfoto 2024-09-09 um 17 27 59](https://github.com/user-attachments/assets/e0e1fe4a-1c0f-46f6-83d5-d9e6efa011ac)

**Hypothesis 2:** During the four onboarding steps that were set in MariaDB, we can see the following difference:

![Bildschirmfoto 2024-09-09 um 17 28 43](https://github.com/user-attachments/assets/2d772233-8064-4827-b114-e64101dffac9)

**Hypothesis 3:** We can see that the NPS is significantly higher among non-German customers. The data analysis was performed in July, which could have led to a further increase after targeted marketing measurements. 

![Bildschirmfoto 2024-09-09 um 17 28 25](https://github.com/user-attachments/assets/a4f4868e-b39a-49eb-9e46-82c572479588)

## Actions and Recommendations 🚀

Data is nothing without the actual reasons and qualitative context where it emerges. As a studied anthropologist and working in the qualitative research department of the company, I conducted interviews together with my research team to explain the findings. In short: Students and side jobber from other countries often have a difficult time finding a temporary job in Germany, most often due to the language barrier and prejudice. Also their situation only allowed them to work limited time. An app with flexible job offers, without a complicated application process or speaking German was perfect for this group. 

I was surprised to see that many significant differences between our two customer segments, and so were my colleagues and the managing board. As I was the company's Customer Happiness Manager, not the data analyst, I gave my findings to the data team for successful validation and thus, worked closely with the marketing team to kickstart the production of marketing material in english, that also took into account their needs and struggles. I set up regular meetings with Product Managers to drive forward a translated app environment and especially onboarding process, so that english speaking customers could understand it better.  

As a first result, we saw an **increased NPS value** and second, a significant increase in revenue, as the company was putting more effort into their new target group. Excluding seasonal fluctuations and average annual growth, the **revenue increase was around 17% YoY.** 📈
