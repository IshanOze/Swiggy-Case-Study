# QUERIES AND SOLUTIONS

#### 1. List Customers and their number of Orders
~~~~sql
select name, count(order_id) as total_orders
from orders
join users on users.user_id = orders.user_id
group by name
order by total_orders;
~~~~

Answer
| name     | orders |
|----------|--------|
| Nitish   | 5      |
| Khushboo | 5      |
| Vartika  | 5      |
| Ankit    | 5      |
| Neha     | 5      |

- All of the customers have equal number of orders except for Anupama and Rishabh who haven't ordered at all

---


#### 2. Calculate the spending of each customer
~~~~sql
select name, sum(amount) as total_spent
from orders o
join users u on u.user_id = o.user_id
group by name
order by total_spent desc;
~~~~

Answer
| name     | spent |
|----------|-------|
| Neha     | 3035  |
| Khushboo | 2670  |
| Ankit    | 1800  |
| Nitish   | 1665  |
| Vartika  | 1320  |

- Neha has spent the most

---

#### 3. Delivery Rating by each customer and the respective Delivery Time
~~~~sql
select name, round(avg(delivery_rating), 2) as avg_del_rating, round(avg(delivery_time), 2) as avg_del_time
from orders o
join users u on u.user_id = o.user_id
group by name
order by avg_del_rating;
~~~~

Answer 
| name     | avg_del_rating | avg_del_time |
|----------|----------------|--------------|
| Khushboo | 2.40           | 43.40        |
| Vartika  | 3.20           | 35.80        |
| Ankit    | 3.20           | 43.40        |
| Nitish   | 3.60           | 34.60        |
| Neha     | 3.60           | 39.20        |

- Although Khushboo has the same delivery time as Ankit, the average Delivery Rating is drastically lower than Ankit
- Neha has given the same Delivery Rating as Nitish although her delivery is late by around 5 minutes 

---

#### 4. Name of Restaurant in which most orders were placed by each customer
~~~~sql
with cte as (select distinct user_id, r_id,
count(r_id) over(partition by user_id, r_id) as r
from orders
order by r desc)

select name, r_name,
first_value(r) over (partition by name) as r
from cte c
join users u on u.user_id = c.user_id
join restaurants r on r.r_id = c.r_id
order by name;
~~~~

Answer
| name     | restaurant | times_ordered |
|----------|------------|---------------|
| Ankit    | Dosa Plaza | 3             |
| Ankit    | China Town | 3             |
| Khushboo | dominos    | 1             |
| Khushboo | kfc        | 1             |
| Khushboo | box8       | 1             |
| Khushboo | Dosa Plaza | 1             |
| Khushboo | China Town | 1             |
| Neha     | kfc        | 3             |
| Neha     | dominos    | 3             |
| Nitish   | box8       | 3             |
| Nitish   | dominos    | 3             |
| Nitish   | kfc        | 3             |
| Vartika  | kfc        | 3             |
| Vartika  | dominos    | 3             |
| Vartika  | Dosa Plaza | 3             |

- Khushboo placed only one order in each of the restaurants
- All the customers have placed a maximum of 3 orders in various restaurants

---

#### 5. Restaurant Rating by customer
~~~~sql
select name, r_name, round(avg(restaurant_rating), 2) as avg_rating
from orders o
join users u on u.user_id = o.user_id
join restaurants r on r.r_id = o.r_id
where restaurant_rating <> 0
group by name, r_name
order by name;
~~~~

Answer 
| name     | restaurant | avg_rating |
|----------|------------|------------|
| Ankit    | China Town | 3.5        |
| Ankit    | Dosa Plaza | 5          |
| Khushboo | box8       | 5          |
| Khushboo | China Town | 4          |
| Khushboo | kfc        | 5          |
| Neha     | dominos    | 1          |
| Neha     | kfc        | 1          |
| Nitish   | box8       | 4.5        |
| Nitish   | dominos    | 3          |
| Nitish   | kfc        | 2          |
| Vartika  | dominos    | 1          |
| Vartika  | Dosa Plaza | 1          |
| Vartika  | kfc        | 2          |

- Neha and Vartika have embarrasingly low ratings for each restaurant

---

#### 6. Top dish ordered by each customer
~~~~sql
with cte as(select user_id, d.f_id, count(d.f_id) as count,
rank() over (partition by user_id order by count(d.f_id) desc) as r
from orders o 
join order_details d on d.order_id = o.order_id
join food f on f.f_id = d.f_id
group by user_id, d.f_id)

select name, f_name
from cte c
join food f on f.f_id = c.f_id
join users u on u.user_id = c.user_id
where r = 1
order by name;
~~~~

Answer
| name     | restaurant       |
|----------|------------------|
| Ankit    | Veg Manchurian   |
| Ankit    | Schezwan Noodles |
| Khushboo | Choco Lava cake  |
| Neha     | Choco Lava cake  |
| Nitish   | Choco Lava cake  |
| Vartika  | Chicken Wings    |

- Choco Lava Cake is the favourite dish of majority of customers

---

#### 7. Type of orders by each customer
~~~~sql
select name, sum(
case
	when type = 'Veg' then 1
    else 0
end) as veg_count,
sum(
case
	when type='Non-veg' then 1
    else 0
end) as non_veg_count
from orders o
join order_details d on d.order_id = o.order_id
join food f on f.f_id = d.f_id
join users u on u.user_id = o.user_id
group by name;
~~~~  

Answer 
| name     | veg_count | non_veg_count |
|----------|-----------|---------------|
| Ankit    | 10        | 0             |
| Khushboo | 10        | 2             |
| Neha     | 5         | 8             |
| Nitish   | 8         | 2             |
| Vartika  | 1         | 4             |

---

#### 8. Top cuisine by each customer
~~~~sql
with cte as (select user_id, cuisine,
rank() over (partition by user_id order by count(cuisine) desc) as r
from orders o 
join restaurants r on r.r_id = o.r_id
group by user_id, cuisine)

select name, cuisine
from cte c
join users u on u.user_id = c.user_id
where r = 1
order by name;
~~~~

Answer 
| name     | cuisine      |
|----------|--------------|
| Ankit    | South Indian |
| Khushboo | Italian      |
| Khushboo | American     |
| Khushboo | North Indian |
| Khushboo | South Indian |
| Khushboo | Chinese      |
| Neha     | American     |
| Nitish   | North Indian |
| Vartika  | American     |

- Khushboo has each cuisine has she has placed exactly one order from each restaurant

---

#### 9. Orders by each month
~~~~sql
select name, 
sum(
case 
	when month(order_date) = 5 then 1 
    else 0
end) as may_orders,
sum(
case 
	when month(order_date) = 6 then 1 
    else 0
end) as june_orders,
sum(
case 
	when month(order_date) = 7 then 1 
    else 0
end) as july_orders
from orders o
join users u on u.user_id = o.user_id
group by name
order by name;
~~~~

Answer
| name     | may_orders | june_orders | july_orders |
|----------|------------|-------------|-------------|
| Ankit    | 2          | 2           | 1           |
| Khushboo | 0          | 2           | 3           |
| Neha     | 0          | 0           | 5           |
| Nitish   | 2          | 2           | 1           |
| Vartika  | 3          | 2           | 0           |

---

#### 10. Average Spend by each customer
~~~~sql
select name, round(avg(amount), 2) as avg_spend_per_order
from orders o
join users u on u.user_id = o.user_id
group by name;
~~~~

Answer
| name     | average_spend |
|----------|---------------|
| Ankit    | 360.00        |
| Khushboo | 534.00        |
| Neha     | 607.00        |
| Nitish   | 333.00        |
| Vartika  | 264.00        |

# IN-DEPTH ANALYSIS

## Ankit 
- is **completely Veg**
- is **pretty satisfied** with the delivery experience
- loves **Veg Manchurain and Shezwan Noodles** but prefers **South Indian cuisine**
- he has ordered from **Dosa Plaza and China Town** and has given **good ratings** to both of the restaurants
- spends **moderately**

## Khushboo
- **prefers Veg** but also buys **Non Veg often**
- is **easily displeased** by late delivery and has **voted the lowest** for delivery rating
- **spends considerably more** than other customers
- **Choco Lava Cake** is her favourite dish while **she likes to try different cuisine**
- she has ordered from **various restaurants** indicating that she **likes exploring different dishes** but has given **excellent ratings** to **Box8 and KFC** 
- her **orders are increasing by the month**, indicating that she's **relatively new on the app and loves exploring different dishes and restaurants**

## Neha
- prefers **Non Veg more often** than **Veg**
- shows **generous delivery rating regardless** of the delivery time
- **spends the most** 
- **Choco Lava Cake** is her favourite dish and **American** is her favourite cuisine.
- although she is the **biggest spender**, she has given the **lowest** ratings to **KFC and Dominos**
- there's a prospect of her ordering **more in the coming months**, but she is a **picky eater and big spender**

## Nitish
- prefers **Veg more frequently** than **Non-Veg**
- **gives moderate** ratings to the delivery personnel
- **spends adequately**
- **Choco Lava Cake** is his favourite dish and **enjoys North Indian cuisine**
-  loves to eat dishes from **Box8** but also likes ordering from **Dominos and KFC**
-  has placed **consistent orders** in the previous months

## Vartika
- **prefers Non-Veg** than **Veg**
- **gives moderate** ratings to the delivery personnel
- **spends the least**
- **Chicken Wings** is her favourite dish and **likes American cuisine**
- has given **very low ratings** to **all the restaurants** that she has ordered from
- her momentum of ordering is **declining over the months** 
