# YouTube trending videos
## Questions and Answers
#### by arnabdey1996420@gmail.com


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

#### Find the Month on Month growth in profit for each year.
#### MoM_growth is calculated as follows.
#### (https://github.com/KopiteArnab/temp/blob/c37bf00dc68e115d1e10d8a9b7a7d7791344ddf6/pics/sql5revampredoa011a3.png)
#### For example,
#### MoM (Feb 2003) = ((Annual Revenue of Feb 2003/Annual Revenue of Jan 2003) - 1) * 100
#### Note: Please make sure that MoM is calculated for each year. Meaning, that ideally MoM growth of the first month of every year should be NULL as shown in the sample output below.
