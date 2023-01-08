# Car Retailer
## Questions and Answers
#### by arnabdey1996420@gmail.com

#### Question 1
#### Find out the top 3 Products (“productLine”) for each month that generated the highest profit.
#### Profit = ( Price per unit - Buying Price per Unit ) x No. of Units Ordered.

````sql
SELECT *
FROM    (SELECT Month_name,
                Productline,
                Profit,
                RANK()
                  OVER  (
                    partition BY month_name
                    ORDER BY profit DESC) AS Ranks
        FROM       (SELECT Productline,
                          Monthname(o.orderdate) AS Month_Name,
                          Sum((od.priceeach - p.buyprice ) *od.quantityordered) AS Profit
                   FROM   cr_orderdetails od
                          JOIN cr_orders o
                            ON od.ordernumber = o.ordernumber
                          JOIN cr_products p
                            ON p.productcode = od.productcode
                 GROUP  BY month_name,
                             productline) a) b
 WHERE ranks <=3
````
**Results:**
-[Output](https://github.com/KopiteArnab/Car-Retailer/blob/a2e8c1c3b0c621fe5cc38978d939c7a6f91432e7/Output/Question_1.csv)

#### Question 2
#### Find the Month on Month growth in profit for each year.
#### MoM_growth is calculated as follows.
#### ![alt text](https://github.com/KopiteArnab/temp/blob/c37bf00dc68e115d1e10d8a9b7a7d7791344ddf6/pics/sql5revampredoa011a3.png)
#### For example,
#### MoM (Feb 2003) = ((Annual Revenue of Feb 2003/Annual Revenue of Jan 2003) - 1) * 100

````sql
SELECT Year_Name,
       Month_Name,
       profit,
       ( ( profit / Lag(profit, 1, NULL)
                      OVER (
                        partition BY year_name) ) -1 ) * 100 As MoM_Growth
FROM   (SELECT Year(o.orderdate) AS Year_Name,
               Monthname(o.orderdate) AS Month_Name,
               Sum((od.priceeach-p.buyprice) * od.quantityordered) AS profit
        FROM cr_orderdetails od
            JOIN cr_orders o
             ON od.ordernumber = o.ordernumber
            JOIN cr_products p
             ON p.productcode = od.productcode
        GROUP BY Month_Name, Year_Name) a
````
**Results:**
-[Output](https://github.com/KopiteArnab/Car-Retailer/blob/a453d91794f747b8ff5c9afd621c561903d9b5ab/Output/Question_2.csv)

#### Question 3
#### For each customer i.e (customerNumber) count the number of orders above and below the average order value.
#### “Average Order Value(AOV)” is calculated as below,
#### ![alt text](https://github.com/KopiteArnab/temp/blob/c2dddb37654084c64913b633f8e2f61fbe580f90/pics/sql5revampredoa011a6.png)
#### Total Revenue is calculated as the sum of the revenue of each order. Each order revenue is Price per unit * No. of Units.

````sql
SELECT customernumber,
       contactfirstname,
       contactlastname,
       Sum(order_value >= avg_order_value) AS order_above_avg,
       Sum(order_value <  avg_order_value) AS order_below_avg
FROM   (SELECT c.customernumber,
               c.contactfirstname,
               c.contactlastname,
               o.ordernumber,
               od.quantityordered,
               od.priceeach,
               priceeach * quantityordered AS order_value,
               Round(Avg(priceeach * quantityordered)
                       OVER(),2) AS avg_order_value
         FROM cr_orderdetails od
              JOIN cr_orders o
               ON od.ordernumber = o.ordernumber
              JOIN cr_customers c
               ON c.customernumber = o.customernumber) a
GROUP BY customernumber,contactfirstname,contactlastname
````

**Results:**
-[Output](https://github.com/KopiteArnab/Car-Retailer/blob/a453d91794f747b8ff5c9afd621c561903d9b5ab/Output/Question_3.csv)

#### Question 4
#### In the business, customer order the product in bulk and it is the utmost priority of the business to fulfill the order requirement as to when required. Hence a business has to keep checking their stock regularly to give uninterrupted delivery.
#### To make that happen, create a report to get the updated stock and status to check if we are running out of stock (or are we going to fulfill the next customer order?)

````sql
SELECT orderdate,
       productcode,
       ordernumber,
       quantityinstock AS initial_qnty_in_stock,
       quantityordered,
       product_sold_so_far,
       quantityinstock - product_sold_so_far AS updatedStock,
       IF(quantityinstock - product_sold_so_far - Lead(quantityordered,1,0)
                                                  OVER(partition By productcode Order By orderdate) > 0, "yes","no") AS are_we_going_to_fulfill_next_order
FROM ( SELECT o.orderdate,
              o.ordernumber,
              p.productcode,
              od.quantityordered,
              od.priceeach,
              p.quantityinstock,
              SUM(od.quantityordered)
               OVER ( partition By p.productcode order By o.orderdate) AS product_sold_so_far
       FROM cr_orderdetails od
            JOIN cr_products p
               ON od.productcode = p.productcode
            JOIN cr_orders o
               ON od.ordernumber = o.ordernumber) a
````

**Results:**
-[Output](https://github.com/KopiteArnab/Car-Retailer/blob/a453d91794f747b8ff5c9afd621c561903d9b5ab/Output/Question_4.csv)

