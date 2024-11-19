# Customers-Analysis
## Objectives
  The objective of this project is to analyze product cancellation trends in an e-commerce business using SQL, with a focus on identifying key factors contributing to cancellations. By uncovering patterns related to products, regions, promotions, and fulfillment methods, this analysis aims to provide actionable insights that can help the business:
  1. Reduce the cancellation rate by addressing underlying issues in product offerings, delivery, or customer expectations.
  2. Enhance customer satisfaction and loyalty by refining operational strategies based on data-driven insights.
  3. Optimize resource allocation in fulfillment, marketing, and inventory to improve overall efficiency and profitability.
     
 This project is essential for improving customer retention, streamlining operations, and supporting long-term business growth in a competitive e-commerce landscape.
## Role
This project is a self-initiated endeavor, undertaken to demonstrate my ability to analyze e-commerce data and address critical business challenges using SQL. Over a span of two weeks, I have:

  1. Gained a comprehensive understanding of the dataset, including its structure, fields, and potential insights.
  2. Designed business-relevant questions focused on understanding product cancellation trends and their underlying causes.
  3. Showcased proficiency in SQL by writing efficient queries to extract and analyze data directly from the source.

This project reflects my ability to independently manage end-to-end data analysis, from understanding the problem and framing questions to delivering actionable insights for business growth.
## Data
This project utilizes a dataset sourced from [Kaggle’s Unlock Profits with E-commerce Sales Data](https://www.kaggle.com/datasets/thedevastator/unlock-profits-with-e-commerce-sales-data/data), which provides detailed information on e-commerce sales and products.

The analysis is based on two main tables:

sales_report Table
This table contains detailed transactional data, including:
|Column Name|	Description|
|-----------|------------|
|Category| The type of product sold|
|Size| The size of the product|
|Date| The date of the transaction|
|Status| The current status of the sale (e.g., completed, canceled)|
|Fulfilment| The method used for order fulfillment (e.g., by Amazon or the merchant)|
|Style| The design or style of the product|
|SKU| The Stock Keeping Unit, a unique identifier for each product|
|ASIN| Amazon Standard Identification Number for the product|
|Courier Status| Status updates on the courier's delivery process|
|Qty| The quantity of the product sold in each transaction|
|Amount| The monetary value of the transaction|
|B2B| Boolean indicator of whether the sale was Business-to-Business|
|Currency| The currency used for the transaction|

products_data Table
This table provides additional product details, including:
|Column Name|	Description|
|-----------|------------|
|SKU| Matches the identifier in the sales_report table|
|Category| The product category|
|Size| The product size|
|ASIN| Amazon's unique identifier for the product|

These tables allow for a comprehensive analysis of sales trends, cancellations, and product-level insights, enabling the development of actionable strategies for improving e-commerce operations and customer satisfaction.


## Model
This project leverages PostgreSQL as the database management system to efficiently handle and query the dataset, ensuring accurate and scalable analysis of e-commerce data. SQL queries were designed and executed to analyze product cancellation trends, uncover patterns, and generate actionable insights.

For development and coding, Visual Studio Code was utilized as the primary integrated development environment (IDE), offering a streamlined workflow for writing and debugging SQL code.

By focusing on tools and techniques aligned with industry practices, this project demonstrates proficiency in handling large datasets and conducting analysis relevant to data-driven business decision-making.


## Code
- Data Cleaning
  1. CHECKING for NULL value
     ```sql
     SELECT *
      FROM sales_report
      WHERE order_id IS NULL
       OR date IS NULL
       OR status IS NULL
       OR fulfilment IS NULL
       OR sales_channel IS NULL
       OR ship_service_level IS NULL
       OR style IS NULL
       OR sku IS NULL
       OR courier_status IS NULL
       OR qty IS NULL
       OR currency IS NULL
       OR amount IS NULL
       OR ship_city IS NULL
       OR ship_state IS NULL
       OR ship_postal_code IS NULL
       OR ship_country IS NULL
       OR promotion_ids IS NULL
       OR b2b IS NULL
     ```
  2. Fixing Null value
     ```sql
       UPDATE sales_report
      SET promotion_ids = 'not applied'
      WHERE promotion_ids IS NULL;
     ```
  4. Standadis Data
     ```sql
      UPDATE sales_report
      SET status = 'Pending'
      WHERE status = 'Pending - Waiting for Pick Up';

      UPDATE sales_report
      SET status = 'Shipping'
      WHERE status = 'Shipped - Out for Delivery';

      UPDATE sales_report
      SET status = 'Shipped'
      WHERE status = 'Shipped - Delivered to Buyer';
  
      UPDATE sales_report
      SET status = 'Shipped'
      WHERE status = 'Shipped - Picked Up'

      UPDATE sales_report
      SET status = 'Returned'
      WHERE status = 'Shipped - Rejected by Buyer'
        or status = 'Shipped - Returned to Seller'
        or status = 'Shipped - Returning to Seller'

      UPDATE sales_report
      SET status = 'Shipped'
      WHERE status = 'Shipping'

      UPDATE sales_report
      SET status = 'Issue'
      WHERE status = 'Shipped - Damaged'
        or status = 'Shipped - Lost in Transit'

      UPDATE sales_report
      set ship_state = UPPER(ship_state);
     ```
- Data exporatory
  
  1. Finding total order, total order shipped, total order cancelled
  ```sql
  --COUNT ALL THE ORDER 
  SELECT 
    COUNT (order_id) AS total_order,
    ship_state
  FROM 
    sales_report
  GROUP BY
    ship_state;

  -- COUNT ORDER THAT SHIPPED
  SELECT 
    COUNT (order_id) AS total_shipped
  FROM 
    sales_report
  WHERE status = 'Shipped';

  -- COUNT ORDER THAT GOT CANCELLED
  SELECT 
    COUNT (order_id) AS total_cancelled
  FROM 
    sales_report
  WHERE status = 'Cancelled';
  ```
  2. Finding Top sales products
  ```sql
  SELECT 
    sr.sku,  
    pd.category,
    COUNT (sr.sku) as sale_count,
    pd.asin
  FROM sales_report AS sr
  JOIN products_data AS pd 
    ON sr.sku = pd.sku
  GROUP BY sr.sku, pd.category,pd.asin
  ORDER BY sale_count DESC
  LIMIT 10;
  ```
  3. Finding trends in order cancellation by-products
  ```sql
  SELECT 
    sr.sku, 
    pd.category, 
    COUNT(sr.status) AS cancellations
  FROM sales_report AS sr
  JOIN products_data AS pd ON sr.sku = pd.sku
  WHERE Status = 'Cancelled'
  GROUP BY sr.sku, pd.category
  ORDER BY cancellations DESC
  LIMIT 10;
  ```
  4. Compare shipped vs cancel order each month
  ```sql
  SELECT 
    EXTRACT(month FROM date) AS Month, 
    COUNT (date) as numberofsale,
    COUNT (CASE WHEN status = 'Cancelled' THEN 1 END) AS TotalCancellations,
    COUNT (CASE WHEN status = 'Shipped' THEN 1 END) AS TotalShipped
  FROM 
    sales_report
  GROUP BY 
    EXTRACT(month FROM date)
  ORDER BY 
    TotalCancellations DESC;
  ```
  5.Finding products sale each month
  ```sql
  SELECT 
    EXTRACT(month FROM date) AS Month, 
    pd.category,
    ROUND (SUM(amount)) AS TotalSales
  FROM 
    sales_report AS sr
  JOIN products_data AS pd ON pd.sku = sr.sku
  GROUP BY 
    EXTRACT(month FROM date), 
    pd.category
  ORDER BY 
    Month, 
    TotalSales DESC;
  ```
  After obtaining the results from SQL queries, the data was exported into CSV files for further processing and sharing. This step allowed seamless integration with other tools, such as Tableau, for advanced data visualization and presentation of insights.
## Results
The analysis revealed key insights into product cancellation trends, which were effectively visualized using Tableau allowed for a clear understanding of:

![Dashboard 1](https://github.com/user-attachments/assets/3a811ef4-ceab-4c26-9905-b200627ffb6b)

## Recommendation
This section leverages SQL-driven insights from the dataset to identify cancellation trends by product, category, and month. Based on these findings, actionable recommendations are proposed to help reduce cancellations, improve customer satisfaction, and optimize overall business performance.

By addressing these issues, the business can strengthen customer loyalty, enhance operational efficiency, and maximize profitability. So here are the recommendations:

- Analyze High-Cancellation Products
  The products with the highest cancellations, such as the JNE3797-KR-L (Western Dress) with 127 cancellations, represent an opportunity for improvement.
  Action:
  1. Investigate customer feedback and reviews for these products to identify common complaints (e.g., size issues, quality concerns).
  2. Reassess product descriptions and images to ensure accuracy.
  3. Improve quality control processes for top-cancellation SKUs.
- Address Category-Specific Trends
  Categories like Western Dress and Sets dominate both sales and cancellations.
  Action:
  1. Focus on reducing returns by offering detailed sizing guides and fit reviews for high-return categories.
  2. Explore offering free size exchanges instead of full returns to minimize cancellations.
- Seasonal Demand Management
  Sales data reveals significant monthly variation in product sales. For example, Western Dresses saw sales peak in April (2,927,781) but decline in June (3,899,334).
  Action:
  1. Optimize inventory levels based on seasonal sales trends to prevent overstocking and cancellations due to unfulfilled demand.
  2. Plan targeted promotions for declining categories to maintain sales momentum.
- Monitor Fulfillment Issues by Month
  Monthly data shows a high disparity between orders and shipments, especially in April, where 7,137 orders were canceled.
  Action:
  1. Evaluate the efficiency of logistics operations during peak months.
  2. Strengthen partnerships with courier services to improve reliability and reduce fulfillment delays.
  3. Communicate expected delivery timelines clearly to customers to manage their expectations.
  
This project highlights the power of data-driven decision-making in addressing key challenges within e-commerce operations. By focusing on order cancellation trends, this analysis has uncovered actionable insights that can guide businesses in optimizing inventory, improving fulfillment processes, and enhancing customer satisfaction.

Through the application of SQL for data exploration and Tableau for visualization, the findings are both detailed and accessible, enabling informed strategic decisions. The recommendations provided not only aim to reduce cancellations but also to foster a more seamless and satisfying customer experience.

This project demonstrates the potential of leveraging data analytics to drive meaningful business outcomes, showcasing skills in problem-solving, technical analysis, and strategic thinking—all essential components for success in today's competitive e-commerce landscape.
