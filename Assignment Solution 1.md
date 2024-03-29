1. Download vechile sales data -> https://github.com/shashank-mishra219/Hive-Class/blob/main/sales_order_data.csv

2. Store raw data into hdfs location

3. Create a internal hive table "sales_order_csv" which will store csv data sales_order_csv .. make sure to skip header row while creating table

4. Load data from hdfs path into "sales_order_csv" 

5. Create an internal hive table which will store data in ORC format "sales_order_orc"

6. Load data from "sales_order_csv" into "sales_order_orc"

#CREATING SALES TABLE IN CSV FORMAT
create table sales_order_csv
(ORDERNUMBER INT,QUANTITYORDERED INT,PRICEEACH INT,ORDERLINENUMBER INT,SALES INT,STATUS STRING,QTR_ID INT,MONTH_ID INT,YEAR_ID INT,PRODUCTLINE STRING,MSRP INT,PRODUCTCODE STRING,PHONE STRING,CITY STRING,STATE STRING,POSTALCODE STRING,COUNTRY STRING,TERRITORY STRING,CONTACTLASTNAME STRING,CONTACTFIRSTNAME STRING,DEALSIZE STRING)
 row format delimited fields terminated by ','
tblproperties("skip.header.line.count"="1");

#LOADING DATA FROM LOCAL HADOOP PATH
load data local inpath 'file:///config/workspace/salesdata.csv' into table sales_data_csv;

#CREATING ORC TABLE
create table sales_order_orc
(ORDERNUMBER INT,QUANTITYORDERED INT,PRICEEACH INT,ORDERLINENUMBER INT,SALES INT,STATUS STRING,QTR_ID INT,MONTH_ID INT,YEAR_ID INT,PRODUCTLINE STRING,MSRP INT,PRODUCTCODE STRING,PHONE STRING,CITY STRING,STATE STRING,POSTALCODE STRING,COUNTRY STRING,TERRITORY STRING,CONTACTLASTNAME STRING,CONTACTFIRSTNAME STRING,DEALSIZE STRING)
STORED AS ORC;

#OVERWRITING ORC TABLE FROM CSV TABLE
FROM sales_data_csv insert overwrite table sales_order_orc select *;

Perform below menioned queries on "sales_order_orc" table :

a. Calculatye total sales per year
SELECT YEAR_ID,SUM(SALES) AS TOTALSALES FROM sales_order_csv GROUP BY YEAR_ID;

SOLUTION
2003    3516514
2004    4723531
2005    1791264

b. Find a product for which maximum orders were placed
SELECT PRODUCTCODE,SUM(QUANTITYORDERED)AS TOTALQUANTITY FROM sales_order_csv GROUP BY PRODUCTCODE ORDER BY SUM(QUANTITYORDERED) DESC LIMIT 1;

SOLUTION
productcode     totalquantity
S18_3232        1774

c. Calculate the total sales for each quarter
SELECT QTR_ID,SUM(SALES) FROM sales_order_csv GROUP BY QTR_ID;

qtr_id  _c1
1       2350510
2       2047855
3       1758673
4       3874271

d. In which quarter sales was minimum
SELECT QTR_ID,SUM(SALES) AS MINSALES FROM sales_order_csv GROUP BY QTR_ID ORDER BY SUM(SALES) ASC LIMIT 1;

SOLUTION
qtr_id  minsales
3       1758673

e. In which country sales was maximum and in which country sales was minimum
(SELECT COUNTRY,SUM(SALES) AS MAXIMUMSALE from sales_order_csv GROUP BY COUNTRY ORDER BY SUM(SALES) DESC LIMIT 1)
UNION
(SELECT COUNTRY,SUM(SALES) AS MAXIMUMSALE from sales_order_csv GROUP BY COUNTRY ORDER BY SUM(SALES) ASC LIMIT 1);

f. Calculate quartelry sales for each city
SELECT CITY,QTR_ID,SUM(SALES) AS TOTALSALES from sales_order_csv GROUP BY (CITY,QTR_ID);

h. Find a month for each year in which maximum number of quantities were sold
SELECT MAX(T.MONTH_ID),T.YEAR_ID,T.TOTALQUANTITY FROM (SELECT MONTH_ID,YEAR_ID,SUM(QUANTITYORDERED) AS TOTALQUANTITY FROM sales_order_csv GROUP BY MONTH)T GROUP BY T.YEAR_ID;
