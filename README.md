# northwind-full-analysis
# northwind-full-analysis

📊 **Northwind Dataset ile Kapsamlı Veri Analizi**

Bu projede Northwind veri seti kullanılarak SQL, DAX, Python (EDA) ve Power BI ile kapsamlı bir analiz gerçekleştirilmiştir. Satış, müşteri, ürün ve kategori verileri detaylı olarak incelenmiş, veriler hem sorgulanmış hem de görselleştirilmiştir.

## ⚙️ Kullanılan Araçlar
- SQL (Veri sorgulama ve analiz)
- Python & Pandas (Keşifsel Veri Analizi - EDA)
- Power BI (Görsel raporlama ve dashboard)
- DAX (Ölçü hesaplama ve filtreleme)

## 📌 Projede Ele Alınan Konular
- En çok satış yapan ürünler ve kategoriler
- Müşteri segmentasyonu
- Yıllık ve aylık satış trendleri
- Bölgelere göre satış performansı
- En kârlı siparişler

## 📁 Veri Seti
- Northwind Traders Dataset

## 🎯 Proje Hedefi
Veriyi uçtan uca analiz ederek, karar destek süreçlerine ışık tutmak; satış ve pazarlama ekiplerine sezgisel veri içgörüleri sunmaktır.

## 🔗 Proje Görselleri & Dashboard
👉 [Google Drive Bağlantısı]([https://drive.google.com/...](https://drive.google.com/drive/folders/1NRYCkCeSTCc-dJoo3_Jq6rJ7rFrDQJJV?usp=sharing))  
Projenin tüm Power BI görselleri ve detaylı anlatımı için yukarıdaki bağlantıyı inceleyebilirsiniz.

SQL KODLARI

 



Overwiev


1.	Brüt kazancımız nedir?

SELECT ROUND(CAST(SUM(unit_price*quantity) AS numeric),2) AS gross_profit
FROM order_details;

 

2.	Net kazancımız nedir?


SELECT ROUND((SUM((unit_price - unit_price * discount) * quantity))::numeric, 2) AS net_profit
FROM order_details;

  


3.	Toplam discount nedir?

SELECT ROUND(CAST(SUM(unit_price*discount*quantity) AS numeric),2) AS total_discount
FROM order_details;
 


4.	Sipariş sayısını hesaplayınız.

SELECT COUNT(*) AS total_orders FROM orders;
 

5.	Müşteri sayısını hesaplayınız.

SELECT * FROM customers;
SELECT COUNT(*) AS count_customers FROM customers;
 

6.	Kategori analizini yapınız.

WITH category_analysis AS(
SELECT c.category_name, COUNT(od.order_id) AS order_count,
ROUND(SUM(od.unit_price*od.quantity*od.discount)::numeric,2) AS total_discount,
ROUND(SUM(od.unit_price * od.quantity * (1 - od.discount))::numeric,2) AS net_profit,
ROUND(SUM(od.unit_price * od.quantity)::numeric,2) AS gross_profit
FROM categories AS c
JOIN products AS p ON p.category_id=c.category_id
JOIN order_details AS od ON od.product_id=p.product_id
JOIN orders AS o ON o.order_id=od.order_id
GROUP BY 1
ORDER BY 5 DESC
)
SELECT 
	category_name,
	order_count,
	gross_profit,
	net_profit,
	total_discount
FROM category_analysis 
ORDER BY 4 DESC;

 

7.	Ay bazında net satış nedir?

SELECT
TO_CHAR(o.order_date, 'Month') AS month_name,
EXTRACT(MONTH FROM o.order_date) AS months,
ROUND(SUM(od.quantity * od.unit_price)::numeric, 2) AS total_price
FROM orders AS o
JOIN order_details AS od ON o.order_id = od.order_id
GROUP BY 1,2
ORDER BY 2 DESC;

 

8.	Ülkelere göre sipariş sayılarını bulunuz.

SELECT c.country,
COUNT(o.order_id) AS total_orders
FROM customers AS c
JOIN orders AS  o ON c.customer_id = o.customer_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
 




9.	Kargo sürelerini bulunuz (ortalama).


WITH ShippingTime AS (
SELECT order_id, (shipped_date - order_date) AS ShippingTime
FROM orders
WHERE shipped_date IS NOT NULL AND order_date IS NOT NULL
)
SELECT ROUND(AVG(ShippingTime)::numeric,2) AS Avg_shipping_time
FROM ShippingTime;
 




Products

1.	Product sayısını hesaplayınız.

SELECT COUNT(*) AS total_products FROM products;
 

2.	Shipped orders ve discointed products hesaplayınız.

SELECT * FROM orders;
SELECT COUNT(shipped_date) AS shipped_products FROM orders;
 

SELECT * FROM products;
SELECT COUNT(discontinued) FROM products
WHERE discontinued = 1

 

3.	En çok sipariş verilen ilk 5 ürün nedir?

SELECT p.product_name,COUNT(od.order_id) AS order_count
FROM order_details AS od
JOIN products AS p ON od.product_id = p.product_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
 


4.	En az sipariş verilen ilk 5 ürün nedir?

SELECT p.product_name,COUNT(od.order_id) AS order_count
FROM order_details AS od
JOIN products AS p ON od.product_id = p.product_id
GROUP BY 1
ORDER BY 2 ASC
LIMIT 5;
 

5.	Ürünlerin toplam stok ve sipariş miktarını hesaplayınız.

SELECT p.product_name, p.unit_in_stock, 
SUM(od.quantity) AS total_ordered_quantity
FROM order_details AS od
JOIN products AS p ON od.product_id = p.product_id
GROUP BY 1,2;
 

6.	Kategori bazında toplam stok hesaplayınız.

SELECT c.category_name, SUM(p.unit_in_stock) AS total_stock
FROM products AS p
JOIN categories AS c ON p.category_id = c.category_id
GROUP BY 1;
 


7.	Aylara göre toplam ciro nedir hesaplayınız. (Python da görselleştirildi).


SELECT 
EXTRACT(MONTH FROM order_date) AS month_number, 
TO_CHAR(order_date, 'Month') AS month_name,
COUNT(o.order_id) AS total_orders,
ROUND(SUM(od.quantity * od.unit_price)::numeric, 2) AS total_price
FROM orders AS o
JOIN order_details AS od ON od.order_id = o.order_id
GROUP BY 1, 2
ORDER BY 1

 

8.	Kategori bazında en çok satan ürünler nelerdir? (Python da görselleştirildi).


WITH top_selling_category AS(
SELECT c.category_id, c.category_name, product_name,
SUM(od.quantity) AS top_product,
RANK() OVER (PARTITION BY c.category_name ORDER BY sum(od.quantity) DESC) AS RANK
FROM categories AS c
JOIN products AS p ON p.category_id=c.category_id
JOIN order_details AS od ON od.product_id=p.product_id
GROUP BY 1,2,3)
SELECT * FROM top_selling_category WHERE rank=1
ORDER BY top_product DESC;

 


Customers

1.	Müşteri bazında ülke sayısı ve şehir sayısını hesaplayınız.

SELECT COUNT(DISTINCT(country)) AS count_country FROM customers;
 

SELECT COUNT(DISTINCT(city)) AS count_city FROM customers;
 

2.	Müşteri ID’sine göre en çok sipariş veren müşterileri bulunuz.

SELECT COUNT(DISTINCT(o.order_id)) AS total_orders, customer_id
FROM orders AS o
JOIN order_details AS od ON od.order_id=o.order_id
GROUP BY 2
ORDER BY 1 DESC;
 
SELECT COUNT(o.order_id) AS total_orders, customer_id
FROM orders AS o
JOIN order_details AS od ON od.order_id=o.order_id
GROUP BY 2
ORDER BY 1 DESC;
 



3.	Ülkelere göre sipariş sayılarını bulunuz.

SELECT c.country,
COUNT(o.order_id) AS total_orders
FROM customers AS c
JOIN orders AS  o ON c.customer_id = o.customer_id
GROUP BY 1
ORDER BY 2 DESC;
 


4.	Müşterilerin ödediği kargo bedelini hesaplayınız.

SELECT c.customer_id, c.company_name, c.country,
SUM(o.freight) AS total_freight
FROM customers AS c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY 1,2,3
ORDER BY 4 DESC
LIMIT 10;
 


5.	Müşteri-Ülke analizini yapınız (Python da görselleştirildi).

SELECT c.country,
COUNT(o.order_id) AS order_count
FROM customers AS c
JOIN orders AS o ON c.customer_id = o.customer_id
GROUP BY 1
ORDER BY 2 DESC;
 


Shipping 

1.	Not shipped ve shippers sayılarını hesaplayınız.

SELECT COUNT(*) AS NotShipped
FROM orders
WHERE shipped_date IS NULL;
 

SELECT COUNT(DISTINCT(ship_via)) AS shippers
FROM orders;

 

2.	Toplam navlun tutarı nedir?

SELECT SUM(freight) AS total_freight FROM orders;
 


3.	Kargo firmalarının toplam maliyetini hesaplayınız.

SELECT sp.company_name, COUNT(order_id) AS total_orders, ROUND(SUM(freight)::numeric,2) AS total_freight
FROM orders AS o
JOIN shippers AS sp ON o.ship_via=sp.shipper_id
GROUP BY 1
ORDER BY 3 DESC;
 


4.	Ülkelere göre teslim sürelerini bulunuz.

WITH delivery_speed AS (
SELECT order_id, ship_country, (shipped_date - order_date) AS shipping_time
FROM orders
WHERE shipped_date IS NOT NULL AND order_date IS NOT NULL
)
SELECT ship_country, ROUND(AVG(shipping_time)::numeric,2) AS Avg_shipping_time
FROM delivery_speed
GROUP BY 1
ORDER BY 2 DESC;

 



5.	Ürün bazında teslim edilme durumlarını inceleyiniz.

SELECT s.company_name,
COUNT(CASE WHEN o.shipped_date IS NULL THEN 1 END) AS Not_Shipped,
COUNT(CASE WHEN o.shipped_date <= o.required_date THEN 1 END) AS On_Time,
COUNT(CASE WHEN o.shipped_date > o.required_date THEN 1 END) AS Late
FROM orders AS o
JOIN shippers AS s ON s.shipper_id = o.ship_via
GROUP BY 1
ORDER BY 1
 


6.	Kargo şirketi analizini yapınız. (Python da görselleştirildi).

SELECT s.shipper_id, s.company_name,
COUNT(DISTINCT o.order_id) AS total_orders,
ROUND(AVG(o.freight)::numeric,2) AS avg_freight,
ROUND(AVG(EXTRACT(DAY FROM (o.shipped_date - o.order_date) * interval '1 DAY'))::numeric,0) AS avg_shipping_days
FROM shippers AS s
JOIN orders AS o ON s.shipper_id = o.ship_via
GROUP BY 1,2

 



Employees

1.	Çalışan sayısı ve ofis sayısını hesaplayınız.

SELECT COUNT(*) AS count_employees FROM employees;
 


SELECT COUNT(DISTINCT(country)) AS offices
FROM employees;
 



2.	Çalışanların işe girdiği yaşı hesaplayınız ve ortalama bulunuz.


SELECT EXTRACT(YEAR FROM AGE(hire_date,birth_date)) AS age,first_name || ' ' || last_name AS employee_name
FROM employees;
 


SELECT ROUND(AVG(EXTRACT(YEAR FROM AGE(hire_date,birth_date))),2) AS avg_age
FROM employees;
 


3.	Çalışanların performansını hesaplayınız.


WITH employee_performance AS (
SELECT e.first_name || ' ' || e.last_name AS full_name, e.title AS title,
p.product_name,
COUNT(od.quantity) AS total_quantity,
ROUND(SUM((od.unit_price - od.unit_price * od.discount) * od.quantity)::numeric, 2) AS net_profit
FROM orders AS o
JOIN employees AS e ON o.employee_id=e.employee_id
JOIN order_details AS od ON o.order_id=od.order_id
JOIN products AS p ON od.product_id=p.product_id
GROUP BY 1,2,3
ORDER BY 5 DESC
)
SELECT full_name, title, COUNT(total_quantity) AS total_quantity,
SUM(net_profit) AS net_profit
FROM employee_performance
GROUP BY 1,2
ORDER BY 3 DESC;
 



