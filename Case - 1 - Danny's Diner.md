1.
   What is the total amount each customer spent at the restaurant?
   
   **for each customer, find the purchases and sum them, later group by customer_id**

>     select customer_id, sum(price)
>     from dannys_diner.sales s 
>     join dannys_diner.menu m
>     on s.product_id = m.product_id
>     group by customer_id

2.
-- How many days has each customer visited the restaurant?

**For each customer, count the number of orders and use distinct to filter unique stuff**

    select customer_id,  
    count(distinct order_date)  
    from dannys_diner.sales  
    group by customer_id

3.

-- What was the first item from the menu purchased by each customer?

**use rank feature to give count for each order 
 pickup the rows with least rank**

    with cte as 
    (
    select s.product_id, s.customer_id,
    s.order_date, m.product_name,
    dense_rank() over (Partition by s.customer_id order by s.order_date) as order_rank
    from dannys_diner.sales s 
    join dannys_diner.menu m 
    on s.product_id = m.product_id 
    )
    
    select distinct customer_id, product_name
     from cte
    where order_rank = 1;

4.
-- What is the most purchased item on the menu and how many times was it purchased by all customers?

**join sales and menu tables, count the product ids for each product, sort by desc, pick top one.**

    select 
     count(m.product_id) as most_used, m.product_name
    from dannys_diner.sales s
    join dannys_diner.menu m
    on  s.product_id = m.product_id
    group by s.product_id, m.product_name
    order by most_used desc 
    limit 1

5.
-- Which item was the most popular for each customer?
 
 **For each customer, find the count of orders made for every product, select top 1 for every customer**


    with cte as(
    select customer_id,
    m.product_name,  count(m.product_id) as order_count,
    rank () over( partition by s.customer_id order by count(s.product_id) desc)  as rankk
    from dannys_diner.menu m 
    join dannys_diner.sales s
    on m.product_id = s.product_id
    group by customer_id, product_name)
    
    select customer_id, product_name, order_count
    from cte
    where rankk = 1;

6.
Which item was purchased first by the customer after they became a member?

    with cte as 
    (
    select 
    s.customer_id, mm.product_name, s.order_date,
    dense_rank () over (partition by s.customer_id 
    order by s.order_date) as rankk
    
    from dannys_diner.members m
    inner join sales s 
    on m.customer_id = s.customer_id
    join menu mm 
    on mm.product_id = s.product_id
    where order_date >= join_date
    order by customer_id )
    
    select 
    distinct customer_id, product_name,  order_date
    from cte where 
    rankk =1;

7.
Which item was purchased just before the customer became a member?

    with cte as 
    (
    select 
    s.customer_id, m.product_name, s.order_date,
    dense_rank () over (partition by s.customer_id 
    order by s.order_date desc ) 
    as rankk
    from dannys_diner.menu m 
    join dannys_diner.sales s 
    on s.product_id = m.product_id
    join dannys_diner.members mm 
    on s.customer_id = mm.customer_id
    where s.order_date < mm.join_date)
    
    select  customer_id, product_name,  order_date
    from cte where 
    rankk =1;

8.
What is the total items and amount spent for each member before they became a member?

    select 
    s.customer_id, 
    count(distinct s.product_id) as count_sold,
    sum(m.price) as dish_price
    from dannys_diner.sales s
    join dannys_diner.menu m
    on s.product_id = m.product_id
    join dannys_diner.members mm 
    on s.customer_id = mm.customer_id
    where s.order_date < mm.join_date
    group by s.customer_id

9.
   If each $1 spent equates to 10 points and sushi has a 2x points multiplier -
   how many points would each customer have?

    > select s.customer_id,  count(s.product_id) as items_sold, sum(
    > 			    case when product_name = 'sushi' then (m.price *20)
    > 		    	else (m.price*10) end ) 
    > 		    	as total_points from sales s join menu m on s.product_id = m.product_id group by s.customer_id

10. 

-- In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


     with cte_dates as 
     (
     select  *, 
      DATE_ADD(join_date, interval 6 day ) as valid_date, 
      last_day('2021-01-31') as last_date
     from members as m
     )
     
     select  d.customer_id, s.order_date, d.join_date, 
     d.valid_date, d.last_date, m.product_name, m.price,
     
     sum(case
    			  when m.product_name = 'sushi' 
    			  then 2 * 10 * m.price
    			  when s.order_date between d.join_date and d.valid_date 
    			  then 2 * 10 * m.price
    					  else 10 * m.price
      end) as points
      
    from cte_dates as d
    join sales as s
     on d.customer_id = s.customer_id
    join menu as m
     on s.product_id = m.product_id
    where s.order_date < d.last_date
    
    group by d.customer_id, 
    s.order_date, 
    d.join_date, 
    d.valid_date, 
    d.last_date, 
    m.product_name, 
    m.price
     
     

 




