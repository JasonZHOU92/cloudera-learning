# Hive row manipulation
select cast(concat(substr(order_date, 1, 4), substr(order_date, 6, 2) as int) from orders limit 10;

select date_format("2019-05-10 00:00:00.0", 'YYYYMM')

select cast(date_format(order_date, 'YYYYMM') as int) from orders limit 10;


# Hive Join

## Inner Join
select o.*, c.* from orders o, customers c where o.order_customer_id = c.customer_id limit 10;

select o.*, c.* from orders o join customers c on o.order_customer_id = c.customer_id;


## Outer Join
// orders that doesn't have any customer
select * from orders o left outer join customers c on o.order_customer_id = c.customer_id;

// customers that doesn't have any order
select * from customers c right outer join orders o on o.order_customer_id = c.customer_id where o.order_customer_id is null;

select * from customer where customer_id not in (select distinct order_customer_id from orders)

// orders that doesn't have any customer or customers that doesn't have any order
select o.*, c.* from orders o full join customers c on o.order_customer_id = c.customer_id where o.order_customer_id is null;

select o.*, c.* from orders o full join customers c on o.order_customer_id = c.customer_id where c.customer_id is null;


## Aggregation
select order_status, count(1) order_status_count from orders group by order_status order by order_status_count DESC;

// wrong
select o.order_id, sum(oi.order_item_subtotal) order_revenue
  from orders o join order_items oi on o.order_id=oi.order_id
  group by o.order_id
  where order_revenue >= 1000;

select o.order_id, sum(oi.order_item_subtotal) order_revenue
  from orders o join order_items oi on o.order_id=oi.order_id
  group by o.order_id
  having sum(oi.order_item_subtotal) >= 1000;

select o.order_date, sum(order_item_subtotal) daily_revenue
  from orders o join order_items oi
  on o.order_id = oi.order_item_order_in
  where o.order_status in ("COMPLETE", "CLOSED")
  group by o.order_date;

select o.order_date, round(sum(order_item_subtotal),2) daily_revenue
  from orders o join order_items oi
  on o.order_id = oi.order_item_order_in
  where o.order_status in ("COMPLETE", "CLOSED")
  group by o.order_date
  sort by o.order_date, order_revenue desc;

select o.order_date, round(sum(order_item_subtotal),2) daily_revenue
  from orders o join order_items oi
  on o.order_id = oi.order_item_order_in
  where o.order_status in ("COMPLETE", "CLOSED")
  group by o.order_date
  having sum(oi.order_item_subtotal) >= 1000
  distributed by o.order_date sort by o.order_date, order_revenue desc;


## Set Operations
select c.customer_id from orders o join customers c on o.order_customer_id = c.customer_id
where cast(date_format(o.order_date, "YYYYMM") as int) = 201905
union
select c.customer_id from orders o join customers c on o.order_customer_id = c.customer_id
where cast(date_format(o.order_date, "YYYYMM") as int) = 201906


## Aggregation over partition
select o.order_date, round(sum(order_item_subtotal) over partition by o.order_id, 2) daily_revenue
  from orders o join order_items oi
  on o.order_id = oi.order_item_order_in
  where o.order_status in ("COMPLETE", "CLOSED")
  group by o.order_date
  having sum(oi.order_item_subtotal) >= 1000
  distributed by o.order_date sort by o.order_date, order_revenue desc;
