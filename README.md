# CustomerInsights
/* Q: Write a query to segregate all products into expensive, mid-expensive, and least-expensive categories based on their unit price. Additionally, display the customer's full name, gender, product name, product category, and the unit price of the product. Ensure the segregation is done per product category. */

-- Create a temporary table to gather customer and product information along with their unit prices
WITH product_table AS (
    SELECT
        c.FirstName + ' ' + c.LastName AS full_name, 
        c.Gender,
        p.EnglishProductName AS product_name,
        pc.EnglishProductCategoryName AS product_category,
        s.UnitPrice
    FROM DimCustomer AS c
    LEFT JOIN FactInternetSales AS s ON c.CustomerKey = s.CustomerKey
    LEFT JOIN DimProduct AS p ON p.ProductKey = s.ProductKey
    LEFT JOIN DimProductSubcategory AS psc ON p.ProductSubcategoryKey = psc.ProductSubcategoryKey
    LEFT JOIN DimProductCategory AS pc ON pc.ProductCategoryKey = psc.ProductCategoryKey
),

-- Create a windowed table to rank products within each category by their unit prices
product_windowed AS (
    SELECT *,
        NTILE(3) OVER (PARTITION BY product_category ORDER BY UnitPrice DESC) AS bucket
    FROM product_table
)

-- Final selection of products with their segment classification
SELECT 
    full_name, 
    Gender,
    product_name,
    product_category, 
    UnitPrice,
    CASE 
        WHEN bucket = 1 THEN 'expensive'
        WHEN bucket = 2 THEN 'mid_expensive'
        ELSE 'least_expensive'
    END AS product_segment  
FROM product_windowed
WHERE product_category = 'Clothing'
