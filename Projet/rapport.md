 MODEL COMPANY

## Commentaires

We cannot know if an order is paid or not. 


## Ventes

- **Le nombre de produits vendus par catégorie et par mois, avec comparaison et taux d'évolution par rapport au même mois de l'année précédente.**

    Note: Taux des produits existentes l'année précédente.
```sql
WITH all_data AS (

	WITH months AS(
		WITH RECURSIVE months_r AS(
			SELECT 1 AS month
			UNION ALL
			SELECT month + 1 FROM months_r WHERE month < 12
		) 
			SELECT month FROM months_r
	),
	years AS(
		SELECT DISTINCT YEAR(orders.orderDate) AS year FROM orders
	),
	categories AS (
		SELECT productlines.productLine AS category FROM productlines
	),
	min_date AS (
	SELECT 
		YEAR(MIN(orders.orderDate)) AS min_year,
		MONTH(MIN(orders.orderDate)) AS min_month
	FROM
		orders
	),
	totals AS(
		SELECT 
			YEAR(ord.orderDate) AS year_n, 
			MONTH(ord.orderDate) AS month_n, 
			pd.productLine AS category_n, 
			SUM(od.quantityOrdered) AS quantity
		FROM orders AS ord
			INNER JOIN orderdetails AS od
				ON ord.orderNumber = od.orderNumber
			INNER JOIN products as pd			
				ON od.productCode = pd.productCode
		GROUP BY year_n, month_n, category_n 
	)
	SELECT 
		year,
		month,
		category,
		COALESCE(quantity, 0) as quantity,
		COALESCE(
				LAG (quantity, 1, -1) 
				OVER (
						PARTITION BY 
							category,
							month 
						ORDER BY
							year ASC
					),
				0
				)
				as last_year_quantity 
	FROM years
		LEFT JOIN months ON TRUE
		LEFT JOIN categories ON TRUE
		LEFT JOIN totals
			ON year = year_n
			AND month = month_n
			AND category = category_n
		,
		min_date
	WHERE 
		(	year <= YEAR(NOW()) AND NOT(
			year = YEAR (NOW()) AND month >= MONTH(NOW()))
		) AND
		(	
			year >= min_date.min_year  AND NOT(
			year = min_date.min_year AND month < min_date.min_month )
		)
)
SELECT 
	*
     ,(quantity - last_year_quantity)*100/last_year_quantity AS evolution_rate
FROM all_data
WHERE
	
	 last_year_quantity <> -1
     AND 
     last_year_quantity <> 0
;
```

- **Ventes - Quantité des produtis -  par pays par années**

```sql
SELECT 
	YEAR(ord.orderDate) AS year_n, 
    offices.country AS country,
	SUM(od.quantityOrdered) AS quantity
FROM offices
	INNER JOIN employees as emp
			ON offices.officeCode = emp.officeCode
    INNER JOIN customers as cust
			ON emp.employeeNumber = cust.salesRepEmployeeNumber
	INNER JOIN orders as ord
			ON cust.customerNumber = ord.customerNumber
	INNER JOIN orderdetails AS od
		ON ord.orderNumber = od.orderNumber
GROUP BY 
	year_n, 
    country
ORDER BY
	country
; 

```

- **Ventes - Quantité des orders -  par pays par années**

```sql
SELECT 
	YEAR(ord.orderDate) AS year_n, 
    offices.country AS country,
	COUNT(ord.orderNumber) AS nb_orders
FROM offices
	INNER JOIN employees as emp
			ON offices.officeCode = emp.officeCode
    INNER JOIN customers as cust
			ON emp.employeeNumber = cust.salesRepEmployeeNumber
	INNER JOIN orders as ord
			ON cust.customerNumber = ord.customerNumber
GROUP BY 
	year_n, 
    country
ORDER BY
	country
; 

```


## Finances 

Le chiffre d'affaires des commandes des deux derniers mois de la base
de données par pays. Deux derniers mois à partir du premier jour du mois en cours.

    Note: Pas de TVA donc payment = qte * price

- CA Country - Office  - Last Two months
```sql

-- COUNTRY - OFFICE
SELECT 
	offices.country AS country,
	SUM(orderdetails.quantityOrdered * orderdetails.priceEach) AS last_two_monthly_profits
FROM offices
INNER JOIN employees
	ON 
		offices.officeCode = employees.officeCode
INNER JOIN customers
	ON
		employees.employeeNumber = customers.salesRepEmployeeNumber
INNER JOIN orders
	ON
		customers.customerNumber = orders.customerNumber
INNER JOIN orderdetails
	ON
		orders.orderNumber = orderdetails.orderNumber
WHERE
	orders.orderDate >= DATE_FORMAT(date_add(DATE_FORMAT(NOW(), '%Y-%m-01'), interval -2 month), "%Y-%m-01")
    AND 
    orders.orderDate < DATE_FORMAT(NOW(), '%Y-%m-01')
    AND 
    orders.status <> 'Cancelled'
GROUP BY
	offices.country
; 
```

- **CA par pays derniers 12 months**
```sql
-- ---------------------------------- Ventes par pays par années
SELECT 
	YEAR(ord.orderDate) AS year_n, 
    offices.country AS country,
	SUM(od.quantityOrdered) AS quantity
FROM offices
	INNER JOIN employees as emp
			ON offices.officeCode = emp.officeCode
    INNER JOIN customers as cust
			ON emp.employeeNumber = cust.salesRepEmployeeNumber
	INNER JOIN orders as ord
			ON cust.customerNumber = ord.customerNumber
	INNER JOIN orderdetails AS od
		ON ord.orderNumber = od.orderNumber
	INNER JOIN products as pd			
		ON od.productCode = pd.productCode
GROUP BY 
	year_n, 
    country
ORDER BY
	country
;
```


- **Total impayée par office mois**

- **Calcul customer - orders - payments**


## Logistique: 

- **Le stock des 5 produits les plus commandés.**

```sql
-- 5 PRODUITS PLUS COMMANDES
SELECT 
	products.productName AS productName,
    products.quantityInStock AS stock,
    SUM(orderdetails.quantityOrdered) AS total_qte
FROM orderdetails  
INNER JOIN products
	ON 
		products.productCode = orderdetails.productCode
GROUP BY
	products.productCode
ORDER BY
	total_qte DESC
LIMIT 5
;
```

- **Alert of products with low stock** 

## Ressources humaines: 

- **Chaque mois, les 2 vendeurs avec le CA le plus élevé.**


```sql
WITH sellers_data AS (
-- all sales amount of every employee every month 
	SELECT 
		YEAR(orders.orderDate) AS year,
		MONTH(orders.orderDate) AS month,
        CONCAT(employees.lastName," ", employees.firstName) AS seller,
		SUM(orderdetails.quantityOrdered * orderdetails.priceEach)AS total,
		RANK() OVER(
				PARTITION BY
					YEAR(orders.orderDate),
					MONTH(orders.orderDate)
				ORDER BY
					SUM(orderdetails.quantityOrdered * orderdetails.priceEach) DESC
				) as ranking
	FROM employees
	JOIN customers
		ON employees.employeeNumber = customers.salesRepEmployeeNumber
	JOIN orders
		ON customers.customerNumber = orders.customerNumber
	JOIN orderdetails
		ON orders.orderNumber = orderdetails.orderNumber
	WHERE
		employees.jobTitle = "Sales Rep"
	GROUP BY
		YEAR(orders.orderDate),
		MONTH(orders.orderDate),
		employees.employeeNumber
)
-- The two best sellers for month 
SELECT 
    year,
    month,
    seller,
    ranking
FROM sellers_data
WHERE ranking <= 2
ORDER BY
	year,
    month,
    ranking DESC
;
```

- ** Chaque mois, les vendeurs avel le CA le moins élevé.**

```sql
-- ------------------------ TWO WORST CA-SELLERS BY MONTH 
WITH sellers_data AS (
-- all sales amount of every employee every month 
	SELECT 
		CONCAT(employees.lastName," ", employees.firstName) AS seller,
		YEAR(orders.orderDate) AS year,
		MONTH(orders.orderDate) AS month,
		SUM(orderdetails.quantityOrdered * orderdetails.priceEach) AS total,
		RANK() OVER(
				PARTITION BY
					YEAR(orders.orderDate),
					MONTH(orders.orderDate)
				ORDER BY
					SUM(orderdetails.quantityOrdered * orderdetails.priceEach) ASC
				) as ranking
	FROM employees
	JOIN customers
		ON employees.employeeNumber = customers.salesRepEmployeeNumber
	JOIN orders
		ON customers.customerNumber = orders.customerNumber
	JOIN orderdetails
		ON orders.orderNumber = orderdetails.orderNumber
	WHERE
		employees.jobTitle = "Sales Rep"
	GROUP BY
		YEAR(orders.orderDate),
		MONTH(orders.orderDate),
		employees.employeeNumber
)
-- The two worst sellers for month 
SELECT 
    year,
    month,
    seller,
    ranking
FROM sellers_data
WHERE ranking <= 2
ORDER BY
	year,
    month,
	ranking
;
```

- **Les vendeurs avec plus des orders.**

```sql
WITH sellers_data AS (
-- all sales amount of every employee every month 
	SELECT 
		YEAR(orders.orderDate) AS year,
		MONTH(orders.orderDate) AS month,
		CONCAT(employees.lastName," ", employees.firstName) AS seller,
		COUNT(orders.orderNumber) AS total,
		RANK() OVER(
				PARTITION BY
					YEAR(orders.orderDate),
					MONTH(orders.orderDate)
				ORDER BY
					COUNT(orders.orderNumber) DESC
				) as ranking
	FROM employees
	JOIN customers
		ON employees.employeeNumber = customers.salesRepEmployeeNumber
	JOIN orders
		ON customers.customerNumber = orders.customerNumber
	WHERE
		employees.jobTitle = "Sales Rep"
	GROUP BY
		YEAR(orders.orderDate),
		MONTH(orders.orderDate),
		employees.employeeNumber
)
-- The two best sellers for month 
SELECT 
    year,
    month,
    seller,
    ranking
FROM sellers_data
WHERE ranking <= 2
ORDER BY
	year,
    month,
    total DESC
;
```

- **Les deux vendeux avec le moins de orders par mois 
```sql
-- ------------------------ TWO WORST QTE-ORDERS-SELLERS BY MONTH 
WITH sellers_data AS (
-- all sales amount of every employee every month 
	SELECT 
		YEAR(orders.orderDate) AS year,
		MONTH(orders.orderDate) AS month,
		CONCAT(employees.lastName," ", employees.firstName) AS seller,
		COUNT(orders.orderNumber) AS total,
		RANK() OVER(
				PARTITION BY
					YEAR(orders.orderDate),
					MONTH(orders.orderDate)
				ORDER BY
					COUNT(orders.orderNumber) ASC
				) as ranking
	FROM employees
	JOIN customers
		ON employees.employeeNumber = customers.salesRepEmployeeNumber
	JOIN orders
		ON customers.customerNumber = orders.customerNumber
	WHERE
		employees.jobTitle = "Sales Rep"
	GROUP BY
		YEAR(orders.orderDate),
		MONTH(orders.orderDate),
		employees.employeeNumber
)
-- The two worst sellers for month 
SELECT 
    year,
    month,
    seller,
    ranking
FROM sellers_data
WHERE ranking <= 2
ORDER BY
	year,
    month,
    ranking DESC
;

```



