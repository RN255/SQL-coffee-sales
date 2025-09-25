# Coffee shop sales and stock tracker

## Task
- Track sales of coffee
- Record drinks recipe with ingredients
- Ingredients and their stock
- Work out how much stock we have left after today

## Tables

We first need to create some tables to store the data.
They will be as follows:

1. Orders
	order_details_id, order_id, order_date, drink_id 

2. Drinks
	drink_id, drink_name, drink_price

3. Drink_recipe
	recipe_details_id, drink_id, ingredient, quantity, measure 

6. Stock
	ing_name, stock_item_quantity, measure

## SQL work process

-- Create the orders table

CREATE TABLE orders (\
	order_details_id INT PRIMARY KEY, \
	order_id VARCHAR, \
	order_date TIMESTAMP, \
	drink_id VARCHAR\
	)

SELECT * FROM orders;

-- Create the drinks table

CREATE TABLE drinks (\
	drink_id VARCHAR PRIMARY KEY,\
	drink_name VARCHAR,\
	drink_price DECIMAL (5,2),\
	recipe_id VARCHAR\
);

SELECT * FROM drinks;

-- Remove the recipe_id column, drink_id already serves as a unique key

ALTER TABLE drinks\
DROP COLUMN recipe_id

-- How many drinks were sold? Which drinks brough in the most revenue?

SELECT \
o.drink_id, \
d.drink_name, \
COUNT (order_details_id) AS sales_volume, \
SUM(d.drink_price) AS total_revenue\
FROM orders o\
LEFT JOIN drinks d \
ON o.drink_id = d.drink_id\
GROUP BY \
o.drink_id, \
d.drink_name\
ORDER BY \
sales_volume DESC

-- What was the total revenue?

SELECT SUM(d.drink_price) AS total_revenue\
FROM orders o\
JOIN drinks d ON d.drink_id = o.drink_id;

-- Create the recipes table

CREATE TABLE drink_recipe (\
	recipe_details_id VARCHAR PRIMARY KEY,\
	drink_id VARCHAR,\
	ingredient VARCHAR,\
	quantity INT,\
	measure VARCHAR\
)

SELECT * FROM drink_recipe

-- We now need to know how much of each ingredient we have used that day and how much for each drink or order...

-- Join the relevant data of orders, drinks and ingredients

SELECT * FROM orders o\
LEFT JOIN drinks d \
ON o.drink_id = d.drink_id\
LEFT JOIN drink_recipe dr \
ON d.drink_id = dr.drink_id\
ORDER BY order_details_id\

-- SUM the data to find the total amount of each ingredient used in the orders

SELECT dr.ingredient, SUM(dr.quantity) AS total_amount_used FROM orders o\
LEFT JOIN drinks d \
ON o.drink_id = d.drink_id\
LEFT JOIN drink_recipe dr \
ON d.drink_id = dr.drink_id\
GROUP BY dr.ingredient\

-- Create an ingredient stock level table

CREATE TABLE stock (\
	ing_name VARCHAR PRIMARY KEY,\
	stock_item_quantity INT,\
	measure VARCHAR\
)

SELECT * FROM stock

-- Deduct the ingredients used in the orders

-- We subtract the ingredients used in the orders to get the remaining stock

-- We can see if the stock has fallen below 75% or 50%

SELECT dr.ingredient,\
SUM(dr.quantity) AS total_amount_used,\
s.stock_item_quantity,\
s.stock_item_quantity - SUM(dr.quantity) AS qty_remaining,\
(s.stock_item_quantity - SUM(dr.quantity) < (s.stock_item_quantity*0.75)) AS stock_below_75_pc,\
(s.stock_item_quantity - SUM(dr.quantity) < (s.stock_item_quantity/2)) AS stock_below_50_pc\
FROM orders o\
LEFT JOIN drinks d \
ON o.drink_id = d.drink_id\
LEFT JOIN drink_recipe dr \
ON d.drink_id = dr.drink_id\
LEFT JOIN stock s\
ON dr.ingredient = s.ing_name\
GROUP BY dr.ingredient, s.stock_item_quantity

## Potential improvements 

- The drinks recipe might need to be split again, with a separate table for the ingredients.
- The stock table might need a better identifier for the ingredients. 
