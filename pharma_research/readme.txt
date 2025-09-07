1.Топ-3 аптеки по объему продаж
SELECT pharmacy_name, SUM(price * count) AS total_sales
FROM pharma_orders
GROUP BY pharmacy_name
ORDER BY total_sales DESC
LIMIT 3;

2.Топ-3 лекарства по объему продаж
SELECT drug, SUM(price * count) AS total_sales
FROM pharma_orders
GROUP BY drug
ORDER BY total_sales DESC
LIMIT 3;

3.Аптеки с оборотом больше 1.8 млн
SELECT pharmacy_name, SUM(price * count) AS total_sales
FROM pharma_orders
GROUP BY pharmacy_name
HAVING SUM(price * count) > 1800000;

4.Накопленная сумма продаж по каждой аптеке (OVER)
SELECT
  pharmacy_name,
  SUM(price * count) AS total_sales,
  SUM(SUM(price * count)) OVER (ORDER BY pharmacy_name) AS cumulative_sales
FROM pharma_orders
GROUP BY pharmacy_name
ORDER BY pharmacy_name;

5.Количество клиентов в аптеках
SELECT
  po.pharmacy_name,
  COUNT(DISTINCT c.customer_id) AS client_count
FROM pharma_orders po
JOIN customers c ON po.customer_id = c.customer_id
GROUP BY po.pharmacy_name
ORDER BY client_count DESC;

6.Лучшие клиенты (топ-10 по сумме заказов)
WITH client_totals AS (
  SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    c.second_name,
    SUM(po.price * po.count) AS total_orders
  FROM pharma_orders po
  JOIN customers c ON po.customer_id = c.customer_id
  GROUP BY c.customer_id, c.first_name, c.last_name, c.second_name
)
SELECT *,
  ROW_NUMBER() OVER (ORDER BY total_orders DESC) AS rn
FROM client_totals
ORDER BY total_orders DESC
LIMIT 10;

7.Накопленная сумма по клиентам
SELECT
  c.customer_id,
  CONCAT_WS(' ', c.last_name, c.first_name, c.second_name) AS full_name,
  SUM(po.price * po.count) AS total_sales,
  SUM(SUM(po.price * po.count)) OVER (ORDER BY SUM(po.price * po.count) DESC) AS cumulative_sales
FROM pharma_orders po
JOIN customers c ON po.customer_id = c.customer_id
GROUP BY c.customer_id, full_name
ORDER BY total_sales DESC;

8.Самые частые клиенты аптек Горздрав и Здравсити
WITH gorzdrav_clients AS (
  SELECT
    c.customer_id,
    CONCAT_WS(' ', c.last_name, c.first_name, c.second_name) AS full_name,
    COUNT(*) AS order_count
  FROM pharma_orders po
  JOIN customers c ON po.customer_id = c.customer_id
  WHERE po.pharmacy_name = 'Горздрав'
  GROUP BY c.customer_id, full_name
  ORDER BY order_count DESC
  LIMIT 10
),
zdravsiti_clients AS (
  SELECT
    c.customer_id,
    CONCAT_WS(' ', c.last_name, c.first_name, c.second_name) AS full_name,
    COUNT(*) AS order_count
  FROM pharma_orders po
  JOIN customers c ON po.customer_id = c.customer_id
  WHERE po.pharmacy_name = 'Здравсити'
  GROUP BY c.customer_id, full_name
  ORDER BY order_count DESC
  LIMIT 10
)
SELECT 'Горздрав' AS pharmacy, * FROM gorzdrav_clients
UNION ALL
SELECT 'Здравсити' AS pharmacy, * FROM zdravsiti_clients;