
# PLSQL Assignment One - Sunrise Supermarket

* **Student Name:** [Your Name]
* **Student ID:** [Your Student ID]
* **DBMS Used:** PostgreSQL 16 (SQL Shell / `psql`)


* **Repository Name:** `assignment_1_your_name-your_id`


---

## 1. Business Scenario Summary

Sunrise Supermarket sells a variety of products to retail customers across different Rwandan cities. The relational database tracks customer accounts, orders, products categorized by type, and line items per transaction. Management requires structured data analysis to identify top-performing customer segments, calculate purchasing frequency, monitor running sales trajectory, and find inactive user profiles.

---

## 2. Queries, Explanations & Execution Results

### Question 1: INNER JOIN (Orders + Customers)

* **Description:** Lists every order along with the customer's name, city, and order date.


* **Explanation:** Performs an `INNER JOIN` linking `orders` to `customers` on `customer_id`.



```sql
SELECT 
    o.order_id,
    c.customer_name,
    c.city,
    o.order_date
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
ORDER BY o.order_id;

```

#### Output Screenshot:

[cite: 3]

---

### Question 2: JOIN (Order Items + Products)

* **Description:** Lists every item ordered, including product details, category, unit price, and quantity.


* **Explanation:** Joins `order_items` with `products` on `product_id` to compute individual item entries across all orders[cite: 1, 2].

```sql
SELECT 
    oi.order_item_id,
    oi.order_id,
    p.product_name,
    p.category,
    p.price,
    oi.quantity
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.product_id
ORDER BY oi.order_item_id;

```

#### Output Screenshot:

[cite: 2]

---

### Question 3: LEFT JOIN (Customers + Orders)

* **Description:** Lists all customers and their associated orders, including customers who have never placed an order.


* **Explanation:** Employs a `LEFT JOIN` starting from `customers` to keep non-ordering accounts in the result set[cite: 1, 4].

```sql
SELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_date;

```

#### Output Screenshot:

[cite: 4]

---

### Question 4: CTE for High Spenders

* **Description:** Calculates total customer expenditure and returns only those spending above the customer population average.


* **Explanation:** Uses a CTE (`CustomerSpend`) to aggregate individual spending (`quantity * price`) and filters against `AVG(total_spent)` in the outer query[cite: 1, 5].

```sql
WITH CustomerSpend AS (
    SELECT 
        c.customer_id,
        c.customer_name,
        SUM(oi.quantity * p.price) AS total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT 
    customer_id,
    customer_name,
    total_spent
FROM CustomerSpend
WHERE total_spent > (SELECT AVG(total_spent) FROM CustomerSpend)
ORDER BY total_spent DESC;

```

#### Output Screenshot:

[cite: 5]

---

### Question 5: Rank Customers by Spend

* **Description:** Ranks all active customers by their cumulative spending, highest first.


* **Explanation:** Uses the `RANK()` window function ordered by total revenue in descending order[cite: 1, 6].

```sql
WITH CustomerSpend AS (
    SELECT 
        c.customer_id,
        c.customer_name,
        SUM(oi.quantity * p.price) AS total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT 
    customer_id,
    customer_name,
    total_spent,
    RANK() OVER (ORDER BY total_spent DESC) AS spend_rank
FROM CustomerSpend;

```

#### Output Screenshot:

[cite: 6]

---

### Question 6: Number Customer Orders Sequentially

* **Description:** Numbers each order per customer chronologically.


* **Explanation:** Uses `ROW_NUMBER() OVER (PARTITION BY o.customer_id ORDER BY o.order_date, o.order_id)` to create sequential order indexes[cite: 1, 7].

```sql
SELECT 
    o.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date,
    ROW_NUMBER() OVER (
        PARTITION BY o.customer_id 
        ORDER BY o.order_date, o.order_id
    ) AS order_sequence_number
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
ORDER BY o.customer_id, o.order_date;

```

#### Output Screenshot:

[cite: 7]

---

### Question 7: Running Total of Revenue

* **Description:** Computes cumulative sales revenue over time ordered by date.


* **Explanation:** Calculates order level sums inside a CTE, then applies `SUM(order_revenue) OVER (ORDER BY order_date, order_id)` to generate a cumulative sum[cite: 1, 8].

```sql
WITH DailyOrderRevenue AS (
    SELECT 
        o.order_id,
        o.order_date,
        SUM(oi.quantity * p.price) AS order_revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY o.order_id, o.order_date
)
SELECT 
    order_id,
    order_date,
    order_revenue,
    SUM(order_revenue) OVER (
        ORDER BY order_date, order_id
    ) AS running_total_revenue
FROM DailyOrderRevenue
ORDER BY order_date, order_id;

```

#### Output Screenshot:

[cite: 8]

---

### Question 8: Days Passed Between Orders

* **Description:** Shows elapsed days between consecutive orders for customers with repeat visits.


* **Explanation:** Applies `LAG(o.order_date)` partitioned by customer to fetch the preceding date, then subtracts dates directly `(order_date - previous_order_date)`[cite: 1, 9].

```sql
WITH CustomerOrders AS (
    SELECT 
        o.customer_id,
        c.customer_name,
        o.order_id,
        o.order_date,
        LAG(o.order_date) OVER (
            PARTITION BY o.customer_id 
            ORDER BY o.order_date
        ) AS previous_order_date
    FROM orders o
    JOIN customers c ON o.customer_id = c.customer_id
)
SELECT 
    customer_id,
    customer_name,
    order_id,
    order_date,
    previous_order_date,
    (order_date - previous_order_date) AS days_since_last_order
FROM CustomerOrders
WHERE previous_order_date IS NOT NULL
ORDER BY customer_id, order_date;

```

#### Output Screenshot:

[cite: 9]

---

## 3. Business Interpretation

1. **High-Value Accounts:** Out of 5 purchasing customers, Alice Smith ($77.50) and Bob Jones ($50.00) spend significantly above the population average ($46.40)[cite: 5, 6]. Management should target these high spenders with VIP loyalty rewards.


2. **Purchasing Frequency:** Repeat customers average between 7 to 24 days between purchases[cite: 9]. Management can set automated re-engagement notifications around day 15 to incentivize faster repeat visits.


3. **Inactive User Acquisition:** The `LEFT JOIN` highlights Fiona Gallagher (Customer ID 6) as a registered account with 0 orders[cite: 4]. Targeted welcome discounts can help convert registered non-buyers into active customers.


4. **Revenue Growth:** Running revenue grew steadily from $8.00 on 2026-01-05 to $232.00 on 2026-03-01[cite: 8]. This demonstrates continuous financial momentum across the first quarter.



---

## 4. Challenges Encountered & Resolutions

* **Challenge 1:** Calculating running revenue required joining line items first before aggregating across order dates[cite: 1, 8].
* *Resolution:* Built a CTE (`DailyOrderRevenue`) to resolve order level totals prior to executing the window function[cite: 8].


* **Challenge 2:** Displaying date differences in PostgreSQL without platform-specific functions[cite: 1, 9].
* *Resolution:* Used native PostgreSQL date subtraction `(order_date - previous_order_date)`, which returns integer day values directly[cite: 9].



---

## 5. How to Run

1. Open **SQL Shell (psql)**.


2. Connect to the database:
```sql
\c sunrise_supermarket

```


3. Run table definitions and seed data:
```text
\i 'path/to/setup.sql'

```


4. Run analytical queries:
```text
\i 'path/to/queries.sql'

```
