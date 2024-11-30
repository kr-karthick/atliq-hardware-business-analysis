## MySQL Course Completion - Project Overview 🚀


## 📌 Project Overview:

This repository showcases projects and exercises completed during the MySQL course by Codebasics. It highlights practical applications of MySQL for data analysis, database design, and query optimization. 

## 📜 Project Overview: AtliQ Hardware

AtliQ Hardware is a global computer hardware firm selling products across multiple channels like distributors, direct sales, and retailers (Brick & Mortar, E-commerce).

🔍 Key Highlights:

✨Analyzed the differences between Relational and Non-Relational Databases to determine optimal use cases.
✨Modeled a Profit & Loss Analysis (P&L) framework for financial insights.
✨Built database structures with Fact and Dimension Tables using Star and Snowflake Schemas.
✨Implemented User-Defined Functions to calculate fiscal year and fiscal quarter (get_fiscal_year, get_fiscal_quarter).
✨Applied Kanban Methodology via Jira for Agile project management.


💡 Key Skills and Concepts Acquired

✨ SQL Proficiency:
Mastered fundamental and advanced SQL concepts, including SELECT, WHERE, GROUP BY, ORDER BY, and Joins (INNER, LEFT, RIGHT, FULL, CROSS).

🔄 Advanced SQL Techniques:
Developed skills in Subqueries, Common Table Expressions (CTEs), Views, Window Functions (ROW_NUMBER, RANK, DENSE_RANK), and Temporary Tables.

📊 Database Design:
Designed normalized relational database schemas (1NF, 2NF, 3NF) with primary and foreign keys. Implemented constraints like NOT NULL, UNIQUE, and intelligent indexing for enhanced performance.

⚡ Optimization Strategies:
Optimized query performance using indexing, partitioning, execution plans, and query restructuring.

🔧 ETL Fundamentals:
Understood OLTP vs. OLAP concepts and explored Data Warehouse architecture.

🛠️ Development Techniques:
Implemented User-Defined Functions, Stored Procedures, Indexes, Triggers, and Events for dynamic database management.


🛠 Tools and Technologies:

💾 Database: Powered by MySQL 8.0.

🖥  IDE: MySQL Workbench

💻 Languages: SQL

## 📌 Problem Statements:


Question :![Screenshot 2024-11-18 125556](https://github.com/user-attachments/assets/cbccf1b2-58f4-486e-adf6-61a7f6697c52)

Query


```Sql 
        Select fcm.date,fcm.product_code,dp.product,dp.variant,fcm.sold_quantity ,fgp.gross_price,
    round(fcm.sold_quantity*fgp.gross_price,2) as gross_price_total
    from fact_sales_monthly fcm
    join dim_product dp
    on fcm.product_code=dp.product_code
    join fact_gross_price fgp
    on 	fgp.product_code=fcm.product_code and 
        fgp.fiscal_year=get_fiscal_year(fcm.date)
    where customer_code = 90002002 and get_fiscal_year(date)=2021
    order by date desc;
```

Output :![image](https://github.com/user-attachments/assets/9626b2f8-f961-4bc4-b792-8965355aea63)



Question: ![image](https://github.com/user-attachments/assets/e0f60c51-bcc1-4baa-9159-dc51cf2d075f)

Query


```Sql 
            select Year(date) as Year,round(sum(sold_quantity *gross_price),2) as Total_gorss_price from fact_sales_monthly fsm
    join
    dim_product dp
    on dp.product_code=fsm.product_code
    join
    fact_gross_price fgp
    on fgp.product_code=fsm.product_code and 
    fgp.fiscal_year=get_fiscal_year(date)
    where customer_code=90002002
    group by Year
    order by Year;
```


Output: ![image](https://github.com/user-attachments/assets/8c46ea69-cdbb-4c1c-8190-1507a9c1529d)


    
Question: ![image](https://github.com/user-attachments/assets/03ad7bd8-e9ef-474d-b7b2-89cf2f3fcec8)

Query:

```Sql 
                CREATE DEFINER=`root`@`localhost` PROCEDURE `get_monthly_gross_sales_for_customer`(
    in_customer_codes text)
    BEGIN

    select MONTHNAME(date) as Month,round(sum(sold_quantity *gross_price),2) as Total_gorss_price from fact_sales_monthly fsm
    join
    fact_gross_price fgp
    on fgp.product_code=fsm.product_code and 
    fgp.fiscal_year=get_fiscal_year(date)
    where find_in_set(customer_code,in_customer_codes)
    group by Month
    order by Month;

END
```


Output:![image](https://github.com/user-attachments/assets/8d6559eb-c4a8-4dfb-9b7f-e357d3cde268)


Question:![Screenshot 2024-11-18 155303](https://github.com/user-attachments/assets/c08e0c15-1299-4c53-a67e-7ca16d298e15)


Query


```Sql 
                    CREATE DEFINER=`root`@`localhost` PROCEDURE `get_market_badge`(
    in in_market varchar(45),
    in in_fiscal_year year,
    out out_badge varchar(45)
    )
    BEGIN
    declare qty int default 0;
    #set default market to india
    if in_market="" then
        set in_market="india";
        end if;
    #Retreive total qty for a given market+fyear
    select market,sum(sold_quantity) from fact_sales_monthly s
    join
    dim_customer c
    on s.customer_code=c.customer_code
    where get_fiscal_year(date)=in_fiscal_year
    and market=in_market
    group by market;
    #determine market badge
    if qty > 5000000 then 
    set out_badge = "Gold";
    else 
    set out_badge = "Silver";
    end if;
    END

END
```

Output:![image](https://github.com/user-attachments/assets/94e58aa6-6497-41b4-8a15-fd294b27bb9f)


Question: The supply chain business manager wants to see which customers’ forecast accuracy has dropped from 2020 to 2021. Provide a complete report with these columns: customer_code, customer_name, market, forecast_accuracy_2020, forecast_accuracy_2021

Query:

```Sql 
    WITH forecast_2020 AS (
        SELECT 
            s.customer_code,
            SUM(s.sold_quantity) AS total_sold_qty,
            SUM(s.forecast_quantity) AS total_forecast_quantity,
            SUM(s.forecast_quantity - s.sold_quantity) AS net_err,
            (SUM(s.forecast_quantity - s.sold_quantity) * 100) / SUM(s.forecast_quantity) AS net_err_prcnt,
            SUM(ABS(s.forecast_quantity - s.sold_quantity)) AS abs_err,
            (SUM(ABS(s.forecast_quantity - s.sold_quantity)) * 100) / SUM(s.forecast_quantity) AS abs_err_prcnt
        FROM fact_act_est s
        WHERE s.fiscal_year = 2020
        GROUP BY s.customer_code
    ),
    forecast_2021 AS (
        SELECT 
            s.customer_code,
            SUM(s.sold_quantity) AS total_sold_qty,
            SUM(s.forecast_quantity) AS total_forecast_quantity,
            SUM(s.forecast_quantity - s.sold_quantity) AS net_err,
            (SUM(s.forecast_quantity - s.sold_quantity) * 100) / SUM(s.forecast_quantity) AS net_err_prcnt,
            SUM(ABS(s.forecast_quantity - s.sold_quantity)) AS abs_err,
            (SUM(ABS(s.forecast_quantity - s.sold_quantity)) * 100) / SUM(s.forecast_quantity) AS abs_err_prcnt
        FROM fact_act_est s
        WHERE s.fiscal_year = 2021
        GROUP BY s.customer_code
    )
    SELECT 
        y20.customer_code,
        c.customer,
        c.market,
        IF(y20.abs_err_prcnt > 100, 0, (100 - y20.abs_err_prcnt)) AS forecast_accuracy_20,
        IF(y21.abs_err_prcnt > 100, 0, (100 - y21.abs_err_prcnt)) AS forecast_accuracy_21
    FROM forecast_2020 y20
    JOIN forecast_2021 y21
    ON y20.customer_code = y21.customer_code
    JOIN dim_customer c
    ON c.customer_code=y20.customer_code
    WHERE 
        IF(y20.abs_err_prcnt > 100, 0, (100 - y20.abs_err_prcnt)) > 
        IF(y21.abs_err_prcnt > 100, 0, (100 - y21.abs_err_prcnt));
```

Output:![image](https://github.com/user-attachments/assets/98e27d55-d124-43f8-bf96-ac87d8a57f45)

Question: ![image](https://github.com/user-attachments/assets/42e60353-d6e8-42b2-a578-fe86195d4915)

First Net sales view has been created then stored procedure for top markets,products and customer created.

net_sales View

```Sql 
       CREATE
    ALGORITHM = UNDEFINED
    DEFINER = `root`@`localhost`
    SQL SECURITY DEFINER
VIEW `net_sales` AS
    SELECT
        `sales_postinv_discount`.`date` AS `date`,
        `sales_postinv_discount`.`fiscal_year` AS `fiscal_year`,
        `sales_postinv_discount`.`customer_code` AS `customer_code`,
        `sales_postinv_discount`.`market` AS `market`,
        `sales_postinv_discount`.`product_code` AS `product_code`,
        `sales_postinv_discount`.`product` AS `product`,
        `sales_postinv_discount`.`variant` AS `variant`,
        `sales_postinv_discount`.`sold_quantity` AS `sold_quantity`,
        `sales_postinv_discount`.`gross_price_total` AS `gross_price_total`,
        `sales_postinv_discount`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`,
        `sales_postinv_discount`.`net_invoice_sale` AS `net_invoice_sale`,
        `sales_postinv_discount`.`post_invoice_discount_pct` AS `post_invoice_discount_pct`,
        ((1 - `sales_postinv_discount`.`post_invoice_discount_pct`) * `sales_postinv_discount`.`net_invoice_sale`) AS `net_sales`
    FROM
        `sales_postinv_discount`
```



Top 5 markets

Query:

```Sql 
        SELECT market,
    round(sum(net_sales)/1000000,2) as net_sales_mln
    FROM gdb0041.net_sales
    where fiscal_year=2021
    group by market
    order by net_sales_mln desc
    limit 5;
```

Stored Procedure:

```Sql 
            CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_markets_by_net_sales`(
    in_fiscal_year int,
    in_top_n INT)
    BEGIN
    SELECT market,
    round(sum(net_sales)/1000000,2) as net_sales_mln
    FROM gdb0041.net_sales
    where fiscal_year=in_fiscal_year
    group by market
    order by net_sales_mln desc
    limit in_top_n;
    END
```

Output:![image](https://github.com/user-attachments/assets/8e786ae5-d581-49c8-8038-a6b6e6be3d19)



Top 5 customers

```Sql 
        SELECT customer,
    round(sum(net_sales)/1000000,2) as net_sales_mln
    FROM net_sales n
    join dim_customer c
    on n.customer_code=c.customer_code
    where fiscal_year=2021
    group by customer
    order by net_sales_mln desc
    limit 5;
```

Stored Procedure:

```Sql 
            CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_customers_by_net_sales`(
    in_market varchar(45),
        in_fiscal_year int,
        in_top_n int)
    BEGIN
    SELECT customer,
    round(sum(net_sales)/1000000,2) as net_sales_mln
    FROM net_sales s
    join dim_customer c
    on s.customer_code=c.customer_code
    where s.fiscal_year=in_fiscal_year and s.market=in_market
    group by customer
    order by net_sales_mln desc
    limit in_top_n;
```



Output:![image](https://github.com/user-attachments/assets/4e5f2692-f66e-4583-9e0f-20808412f8a0)


Top 5 products

Query:

```Sql 
        SELECT product,round(sum(net_sales)/1000000,2) as net_sales_mln
    FROM gdb0041.net_sales
    where fiscal_year=2021
    group by product
    limit 5;
    ```

    Stored Procedure:

    ```Sql 
        CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_products_by_netsales`(
    in_fiscal_year int,
            in_top_n int
    )
    BEGIN
    SELECT product,round(sum(net_sales)/1000000,2) as net_sales_mln
    FROM gdb0041.net_sales
    where fiscal_year=in_fiscal_year
    group by product
    limit in_top_n;

    END
```


Output:![image](https://github.com/user-attachments/assets/96f141ea-e45b-453f-8d2a-59f607f10373)
