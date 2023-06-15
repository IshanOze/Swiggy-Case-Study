# Contents
- [Queries and Solutions](/DELIVERYMEN.md#queries-and-solutions)
- [Findings](/DELIVERYMEN.md#findings)

---

# QUERIES AND SOLUTIONS

#### 1. Average delivery time by each delivery man
~~~~sql
select partner_name, round(avg(delivery_time), 2) as avg_del_time
from orders o
join delivery_partner d on d.partner_id = o.partner_id
group by partner_name
order by partner_name;
~~~~
Answer
| partner_name | avg_del_time |
|--------------|--------------|
| Amit         | 39.67        |
| Gyandeep     | 29.25        |
| Kartik       | 41.50        |
| Lokesh       | 34.50        |
| Suresh       | 46.14        |

---

#### 2. Average delivery rating for each deliveryman
~~~~sql
select partner_name, round(avg(delivery_rating), 2) as avg_rating
from orders o
join delivery_partner d on d.partner_id = o.partner_id
group by partner_name
order by partner_name;
~~~~
Answer
| partner_name | avg_rating |
|--------------|------------|
| Amit         | 3.00       |
| Gyandeep     | 3.50       |
| Kartik       | 3.00       |
| Lokesh       | 4.00       |
| Suresh       | 2.86       |

---

#### 3. Most deliveries by restaurant
~~~~sql
with cte as(select partner_id, r_id, count(r_id),
rank() over(partition by partner_id order by count(r_id) desc) as r
from orders o
group by partner_id, r_id)

select partner_name, r_name
from cte c
join delivery_partner d on d.partner_id = c.partner_id
join restaurants r on r.r_id = c.r_id
where r = 1
order by partner_name;
~~~~
Answer
| partner_name | r_name     |
|--------------|------------|
| Amit         | kfc        |
| Gyandeep     | box8       |
| Kartik       | kfc        |
| Lokesh       | kfc        |
| Suresh       | dominos    |
| Suresh       | China Town |

---

#### 4. Delivery time and day
~~~~sql
with cte as(select partner_id, dayname(order_date) as order_day, delivery_time,
rank() over(partition by partner_id order by delivery_time desc) as r
from orders
group by partner_id, order_day, delivery_time)

select partner_name, order_day
from cte c
join delivery_partner d on d.partner_id = c.partner_id
where r = 1;
~~~~
Answer
| partner_name | day      |
|--------------|----------|
| Suresh       | Thursday |
| Amit         | Thursday |
| Lokesh       | Thursday |
| Kartik       | Monday   |
| Gyandeep     | Thursday |
| Gyandeep     | Friday   |

# FINDINGS
- Gyandeep takes the **least amount of delivery time** and has a **modest rating**
- Suresh takes the **most amount of delivery time** and has the **least rating**
- Majority of the delivery men have taken **more time to deliver on Thursdays**
- Lokesh has the **best rating** with **subpar delivery time**
