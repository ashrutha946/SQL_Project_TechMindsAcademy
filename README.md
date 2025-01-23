# SQL_Project_TechMindsAcademy

Database Setup
Database Creation: The project starts by creating a database named project_pizza_sales.
Table Creation:  A table named order_details is created to store the order data. The Table structure includes columns for Order_Details_ID, Order_ID, Pizza_ID, Quantity.
                 A table named orders is created to store the order details. The Table structure includes columns for order_id, Date, Time.
                 A table named Pizzas is created to store the Pizza Details. The Table structure includes columns for Pizza_ID, Pizza_Type_Id, Size, Price.
                 A table named pizza_types is created to store the Pizza Types details. The Table structure includes columns for Pizza_Type_Id, Name, Category.

Pizza Sales
Pizza Hut is a multinational chain of pizza restaurants and delivery outlets known for its extensive menu of pizzas. Here we are analyzing the given data set on different parameters to find valuable information.


This data set contains 4 tables.
Orders:
Order_id: It contains the total orders placed at pizza hut. (Bill no)
Date: It contains the date on which pizza is ordered.
Time: It contains timestamp when pizza order is placed.

Order details:
Order_details_id: It contains the id for each pizza that is ordered.
Order_id: It contains the total orders placed at pizza hut. (Bill no)
Pizza_id: It contains pizza id with size that is orderd.
Quantity: It contains total count for that particular pizza that is ordered.

Pizzas:
Pizza_id: It contains pizza id with size for all the available pizzas.
Pizza_type_id: It contains pizza id for all the available pizzas at pizza hut.
Size: It contains size of all the pizzas available at pizza hut.
Price: It contains price for each pizza depending upon its size.

Pizza_types:
Pizza_type_id: It contains pizza id for all the available pizzas at pizza hut.
Name: It contains detail name of the pizzas available at pizza hut
Category: It contains category for each pizza.

Basic Level Query

Retrieve the total number of orders placed.

Select Count(order_id) as Total_Orders From orders;

--Calculate the total revenue generated from pizza sales

SELECT SUM(order_details.quantity * pizzas.price) AS Total_Sales
FROM order_details INNER JOIN pizzas
ON pizzas.pizza_id=order_details.pizza_id;

--Identify the highest-priced pizza

Select pizza_types.name, pizzas.price
FROM pizza_types INNER JOIN pizzas 
ON pizzas.pizza_type_id=pizza_types.pizza_type_id
ORDER BY pizzas.price DESC LIMIT 2;

---Identify the most common pizza size ordered.

Select pizzas.size, count(order_details.order_details_id) AS Total_quantity
From pizzas Inner join order_details
ON order_details.pizza_id= pizzas.pizza_id
Group by pizzas.size
ORDER BY Total_quantity DESC;

--- List the top 5 most ordered pizza types along with their quantities

Select pizza_types.name, Count(order_details.quantity) as total_quantity
FROM pizza_types INNER JOIN pizzas
ON pizzas.pizza_type_id=pizza_types.pizza_type_id
INNER JOIN order_details
ON order_details.pizza_id=pizzas.pizza_id
Group by pizza_types.name
ORDER BY total_quantity desc LIMIT 5;

Intermediate Level Query

---Join the necessary tables to find the total quantity of each pizza category ordered.

Select pizza_types.category, Count(order_details.quantity) as Total_Quantity
FROM pizza_types INNER JOIN pizzas
ON pizzas.pizza_type_id=pizza_types.pizza_type_id
Inner Join order_details
ON order_details.pizza_id=pizzas.pizza_id
Group by pizza_types.category;

---Determine the distribution of orders by hour of the day.

SELECT HOUR(time) AS Hour, COUNT(order_id) AS Total_orders
FROM order GROUP BY HOUR(time) ORDER BY Hour ASC;

---Join relevant tables to find the category-wise distribution of pizzas
Select category, count(name) as Total_pizzas from pizza_types
group by category;

---Group the orders by date and calculate the average number of pizzas ordered per day.

select avg(quantity) AS Average_number from
(select orders.date,sum(order_details.quantity) as quantity
from orders inner join order_details 
on order_details.order_id=orders.order_id 
group by orders.date) as  order_quantity;


---Determine the top 3 most ordered pizza types based on revenue.

Select pizza_types.name, SUM(order_details.quantity *pizzas.price) as revenue
from pizza_types INNER JOIN pizzas
ON pizzas.pizza_type_id=pizza_types.pizza_type_id INNER JOIN order_details
ON order_details.pizza_id=pizzas.pizza_id
Group by pizza_types.name ORDER BY revenue DESC LIMIT 3;

Advanced Level Query

---Calculate the percentage contribution of each pizza type to total revenue.

Select pizza_types.category, (SUM(order_details.quantity  * pizzas.price) / (Select 
SUM(order_details.quantity * pizzas.price) AS Total_Sales
FROM order_details INNER JOIN pizzas
ON pizzas.pizza_id=order_details.pizza_id) ) * 100  AS revenue_percentage
from pizza_types INNER JOIN pizzas
ON pizzas.pizza_type_id=pizza_types.pizza_type_id INNER JOIN order_details
ON order_details.pizza_id=pizzas.pizza_id
Group by pizza_types.category order by revenue_percentage desc;


---Analyze the cumulative revenue generated over time.
SELECT Date,
       SUM(revenue) OVER (ORDER BY date) AS cum_revenue
FROM (
    SELECT orders.date,
           SUM(order_details.quantity * pizzas.price) AS revenue
    FROM order_details
   INNER JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
    INNER JOIN orders ON orders.order_id = order_details.order_id
    GROUP BY orders.date
) AS sales;


---Determine the top 3 most ordered pizza types based on revenue for each pizza category.

With A AS
(Select pizza_types.category, pizza_types.name,
SUM(order_details.quantity * pizzas.price) AS revenue 
from pizza_types inner join pizzas
on pizza_types.pizza_type_id=pizzas.pizza_type_id 
inner join order_details 
on order_details.pizza_id=pizzas.pizza_id
group by pizza_types.category, pizza_types.name)

Select Name, revenue from
(select category, name, revenue, rank() over(partition by category order by revenue desc) as rn 
from A) AS B 
where rn<=3 Order By rn Desc LIMIT 3;


-----------
Select Name, revenue from
(select category, name, revenue, rank() over(partition by category order by revenue desc) as rn 
from 
(Select pizza_types.category, pizza_types.name,
SUM(order_details.quantity*pizzas.price) AS revenue 
from pizza_types inner join pizzas
on pizza_types.pizza_type_id=pizzas.pizza_type_id 
inner join order_details 
on order_details.pizza_id=pizzas.pizza_id
group by pizza_types.category, pizza_types.name) as A)as b
where rn<=3 LIMIT 3;


