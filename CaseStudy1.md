SQL-casestudy1
================
ASHA
2022-10-12

##### Casestudy from: <https://8weeksqlchallenge.com/case-study-1/>

##### Queries practiced: SQLite

### Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he
decides to embark upon a risky venture and opens up a cute little
restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of assistance to help the restaurant stay
afloat - the restaurant has captured some very basic data from their few
months of operation but have no idea how to use their data to help them
run the business.

#### Problem Statement

Danny wants to use the data to answer a few simple questions about his
customers, especially about \* Their visiting patterns

-   How much money they’ve spent

-   Which menu items are their favourite

Having this deeper connection with his customers will help him deliver a
better and more personalised experience for his loyal customers.He plans
on using these insights to help him decide whether he should expand the
existing customer loyalty program - additionally he needs help to
generate some basic datasets so his team can easily inspect the data
without needing to use SQL.Danny has shared 3 key datasets for this case
study:

• sales

• menu

• members

**SCHEMA**

    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;

    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );

    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
    VALUES
      ('A', '2021-01-01', '1'),
      ('A', '2021-01-01', '2'),
      ('A', '2021-01-07', '2'),
      ('A', '2021-01-10', '3'),
      ('A', '2021-01-11', '3'),
      ('A', '2021-01-11', '3'),
      ('B', '2021-01-01', '2'),
      ('B', '2021-01-02', '2'),
      ('B', '2021-01-04', '1'),
      ('B', '2021-01-11', '1'),
      ('B', '2021-01-16', '3'),
      ('B', '2021-02-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-07', '3');
     

    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );

    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      

    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );

    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');

------------------------------------------------------------------------

#### Case Study Questions

**Query \#1**

What is the total amount each customer spent at the restaurant?

    SELECT customer_id,
           sum(price) AS Total_money_spent
      FROM sales
           JOIN
           menu ON sales.product_id = menu.product_id
     GROUP BY customer_id
     ORDER BY customer_id;

*Output-Summary:-*

-   A spent $76;  
-   B spent $74;  
-   C spent $36

**Query \#2**

How many days has each customer visited the restaurant?

    SELECT customer_id,
           count(order_date) AS No_of_days_visit
      FROM sales
     GROUP BY customer_id;

*Output-Summary:-*

-   A visited 6 days;  
-   B visited 6 days;  
-   C visited 3 days

**Query \#3**

What was the first item from the menu purchased by each customer?

    SELECT customer_id,
           min(order_date) AS First_Orderdate,
           product_name AS First_Item
      FROM sales
           JOIN
           menu ON sales.product_id = menu.product_id
     GROUP BY customer_id;

*Output-Summary:-*

-   First item purchased by A was ‘sushi’;  
-   First item purchased by B was ‘curry’;  
-   First item purchased by C was ‘ramen’

**Query \#4**

What is the most purchased item on the menu and how many times was it
purchased by all customers?

    SELECT product_id,
           product_name,
           count(product_name) AS No_of_Purchases
      FROM sales
           JOIN
           menu ON sales.product_id = menu.product_id
     GROUP BY product_name
     ORDER BY No_of_Purchases DESC
     LIMIT 1;
     

*Output-Summary:-*

-   The most purchased item is ‘ramen’ and it was purchased 8 times by
    all customers

<!-- -->

    WITH famous_item AS (
        SELECT sales.product_id,
               product_name as Popular_Item,
               count(product_name) AS No_of_Purchases
          FROM sales
               JOIN
               menu ON sales.product_id = menu.product_id
         GROUP BY product_name
         ORDER BY No_of_Purchases DESC
         LIMIT 1
    )
    SELECT customer_id,Popular_Item,
           count(sales.product_id) AS No_of_Orders
      FROM sales
           JOIN
           famous_item ON sales.product_id = famous_item.product_id
     GROUP BY customer_id;
     

*Output-Summary:-*

-   A has ordered the popular item (ramen) 3 times;
-   B has ordered the popular item (ramen) 2 times;
-   C has ordered the popular item (ramen) 3 times.

**Query \#5**

Which item was the most popular for each customer?

    CREATE VIEW cus_fav_item AS
        SELECT customer_id,
               sales.product_id,
               count(sales.product_id) AS fav_item,
               product_name
          FROM sales
               JOIN
               menu ON sales.product_id = menu.product_id
         GROUP BY customer_id,
                  sales.product_id;

    SELECT customer_id,
           product_name as fav_item,
           max(fav_item) as no_of_orders
      FROM cus_fav_item
     GROUP BY customer_id;

*Output-Summary:-*

-   Popular item of customer A was ‘ramen’;
-   Popular item of customer B was ‘sushi’;
-   Popular item of customer C was ‘ramen’

**Query \#6**

Which item was purchased first by the customer after they became a
member?

    SELECT sales.customer_id,
           min(sales.order_date) AS order_date,
           members.join_date,
           menu.product_name AS FirstOrder_AfterMembership
      FROM sales
           JOIN
           members ON sales.customer_id = members.customer_id
           JOIN
           menu ON sales.product_id = menu.product_id
     WHERE sales.order_date >= members.join_date
     GROUP BY sales.customer_id;

*Output-Summary:-*

-   Customer A purchased ‘curry’ after A became a member;
-   Customer B purchased ‘sushi’ after B became a member

**Query \#7**

Which item was purchased just before the customer became a member?

    SELECT sales.customer_id,
           max(sales.order_date) AS order_date,
           members.join_date,
           menu.product_name AS LastOrder_BeforeMembership
      FROM sales
           JOIN
           members ON sales.customer_id = members.customer_id
           JOIN
           menu ON sales.product_id = menu.product_id
     WHERE sales.order_date < members.join_date
     GROUP BY sales.customer_id;

*Output-Summary:-*

-   Last purchase of Customer A was ‘sushi’ before A became a member;
-   Last purchase of Customer B was ‘sushi’ before B became a member;

**Query \#8**

What is the total items and amount spent for each member before they
became a member?

    SELECT sales.customer_id,
           sales.order_date AS order_date,
           members.join_date AS membership_joindate,
           count(menu.product_name) AS no_items,
           sum(menu.price) AS tot_amt_spent
      FROM sales
           JOIN
           members ON sales.customer_id = members.customer_id
           JOIN
           menu ON sales.product_id = menu.product_id
     WHERE sales.order_date < members.join_date
     GROUP BY sales.customer_id;

*Output-Summary:-*

-   Total amount spent by customer A before A became a member was $25;
-   Total amount spent by customer B before A became a member was $40

**Query \#9**

If each $1 spent equates to 10 points and sushi has a 2x points
multiplier - how many points would each customer have?

    CREATE VIEW points_view AS
        SELECT sales.customer_id,
               menu.product_name,
               menu.price,
               CASE WHEN menu.product_name = "sushi" THEN menu.price * 20 ELSE menu.price * 10 END AS points
          FROM sales
               JOIN
               menu ON sales.product_id = menu.product_id;

    SELECT customer_id,
           sum(points) total_points
      FROM points_view
     GROUP BY customer_id;
     

*Output-Summary:-*

-   Total points of customer A is 860
-   Total points of customer B is 940
-   Total points of customer C is 360

**Query \#10**

In the first week after a customer joins the program (including their
join date) they earn 2x points on all items, not just sushi - how many
points do customer A and B have at the end of January?

*joined tables sales,menu and members; selected the columns customer_id
and order_date from sales ,product_name and price from menu,join_date
from members; made 2 new columns(1)membership first week end date-added
7 days to the join_date,(2)month-extracted month no from order_date.*

    SELECT sales.customer_id,
           menu.product_name,
           sales.order_date,
           members.join_date,
           menu.price,
           date(members.join_date, '+7 days') AS FirtstWeek_date,
           substr(order_date, 6, 2) AS month,
           CASE WHEN sales.order_date >= members.join_date AND 
                     sales.order_date <= date(members.join_date, '+7 days') THEN 
                     menu.price * 20 ELSE menu.price * 10 END AS points
      FROM sales
           JOIN
           menu ON sales.product_id = menu.product_id
           JOIN
           members ON sales.customer_id = members.customer_id;
           

*using the above table found how many points do customer A and B have at
the end of January by filtering only the January data(where month =
‘01’).*

    WITH Points_FirstWeek_ofer AS (
        SELECT sales.customer_id,
               substr(order_date, 6, 2) AS month,
               CASE WHEN sales.order_date >= members.join_date AND 
                         sales.order_date <= date(members.join_date, '+7 days') 
                         THEN menu.price * 20 ELSE menu.price * 10 END AS points
          FROM sales
               JOIN
               menu ON sales.product_id = menu.product_id
               JOIN
               members ON sales.customer_id = members.customer_id
    )
    SELECT customer_id,
           sum(points) AS Total_points
      FROM Points_FirstWeek_ofer
     WHERE month = '01'
     GROUP BY customer_id;
     

*Output-Summary:-*

-   Total points after memberships; customer-A :- 1270, customer-B :-
    840

**Bonus Question**

Recreate a table with the available sales data

    CREATE TABLE combined_table AS SELECT sales.customer_id,
                                          sales.order_date,
                                          menu.product_name,
                                          menu.price,
                                          CASE WHEN sales.order_date < members.join_date THEN "N" WHEN members.join_date IS NULL THEN "N" ELSE "Y" END AS member
                                     FROM sales
                                          LEFT JOIN
                                          menu ON sales.product_id = menu.product_id
                                          LEFT JOIN
                                          members ON sales.customer_id = members.customer_id;

    SELECT *
      FROM combined_table;
      

**Create a table with Ranking**

    ALTER TABLE combined_table ADD COLUMN ranking VARCHR (4); /* adding a new column called ranking */

    UPDATE combined_table
       SET ranking = 
       CASE WHEN member = "N" THEN "null" 
       WHEN customer_id = "A" AND product_name = "curry" THEN 1 
       WHEN customer_id = "A" AND product_name = "ramen" AND order_date = "2021-01-10" THEN 2 
       WHEN customer_id = "A" AND product_name = "ramen" AND order_date = "2021-01-11" THEN 3 
       WHEN customer_id = "A" AND product_name = "ramen" AND order_date = "2021-01-11" THEN 3 
       WHEN customer_id = "B" AND product_name = "ramen" AND order_date = "2021-01-16" THEN 2 
       WHEN customer_id = "B" AND product_name = "ramen" AND order_date = "2021-02-01" THEN 3 
       WHEN customer_id = "B" AND product_name = "sushi" AND order_date = "2021-01-11" THEN 1 END;

### Conclusion

-   Well designed casestudy to practice complex SQL queries with a small
    dataset.

-   Well organized casestudy Quenstions to understand how to do an
    analysis on a dataset.

-   Observations from the analysis:

    -   Nature of the customers; no. of visits, total money spent,
        favorite item

    -   Membership effects in purchases

    -   Total points earned before n after becoming member.
