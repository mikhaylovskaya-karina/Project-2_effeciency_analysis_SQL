# Project-2: Business effeciency analysis SQL

Компания PP - e-commerce-компания, специализирующаяся на продаже офисной техники и канцелярских принадлежностей.

**Цель:** провести анализ деятельности компании PP с точки зрения ее эффективности и дать рекомендации по масштабированию бизнеса.

**Задачи:**
- Оценить динамику продаж и распределение выручки по товарам.
- Составить портрет клиента, а для этого — выяснить, какие клиенты приносят больше всего выручки.
- Проконтролировать логистику компании (определить, все ли заказы доставляются в срок и в каком штате лучше открыть офлайн-магазин).

***Requirements: PostgreSQL***

**Исходные данные представленны в виде:**

Таблица store_customers

| Column_name | Type    | Description                          | 
| :---------- | :------ | :----------------------------------- | 
| cust_id     | varchar | id клиента                           |
| cust_nm     | text    | имя клиента                          |
| category    | text    | тип клиента: 'Consumer', 'Corporate' |

Таблица store_products

| Column_name | Type    | Description                              | 
| :---------- | :------ | :--------------------------------------- | 
| product_id  | varchar | id товара                                |
| category    | text    | категория товара                         |
| subcategory | text    | подкатегория товара                      |
| product_nm  | text    | название, свойства (цвет, модель, бренд) |
| price       | numeric | цена товара                              |

Таблица store_сarts

| Column_name | Type    | Description                              | 
| :---------- | :------ | :--------------------------------------- | 
| id          | integer | id записи в таблице                      |
| order_id    | varchar | id заказа                                |
| product_id  | varchar | id товара                                |
| quantity    | integer | кол-во единиц отдельного товара в заказе |
| discount    | numeric | скидка                                   |

Таблица store_delivery

| Column_name | Type    | Description               | 
| :---------- | :------ | :------------------------ | 
| order_id    | varchar | id заказа                 |
| order_date  | date    | дата заказа               |
| ship_date   | date    | фактическая дата доставки |
| ship_mode   | text    | тип доставки              |
| state       | text    | штат доставки             |
| city        | text    | город доставки            |
| zip_code    | text    | индекс доставки           |
| cust_id     | varchar | id клиента                |


## 1. Анализ эффективности продаж компании PP

Рассматриваемый период - 4 года с 2017-01-03 по конец 2020-12-30. Объем выручки в рассматриваемом периоде составил 1 446 157 долл. США.

**1.1 Объемы продаж компании PP по месяцам**

```
  SELECT 
      date_trunc('month', sd.order_date)::date date, 
      round(sum(sc.quantity * sp.price * (1 - sc.discount))) revenue
  FROM sql.store_carts sc
  JOIN sql.store_products sp ON sp.product_id = sc.product_id
  JOIN sql.store_delivery sd ON sd.order_id = sc.order_id
  GROUP BY 1
  ORDER BY 1
```
*– запрос выводит сумму выручки компании по месяцам*

![image](https://github.com/mikhaylovskaya-karina/Project-2_effeciency_analysis_SQL/blob/main/visual/%D0%94%D0%B8%D0%BD%D0%B0%D0%BC%D0%B8%D0%BA%D0%B0%20%D0%B2%D1%8B%D1%80%D1%83%D1%87%D0%BA%D0%B8%20PP%20%D0%BF%D0%BE%20%D0%BC%D0%B5%D1%81%D1%8F%D1%86%D0%B0%D0%BC.png)

График 1. Динамика показателя выручки компании PP по месяцам за период с 2017 г. по конец 2020 г.

Динамика продаж демонстрирует тенденцию роста, но носит сезонный характер - падение в начале года, рост в последнем квартале.


**1.2 Сумма выручки по различным категориям и подкатегориям продукции PP**

```
  SELECT 
      sp.category,
      sp.subcategory,
      round(sum(sc.quantity * sp.price * (1 - sc.discount))) revenue
  FROM sql.store_products sp
  JOIN sql.store_carts sc ON sp.product_id = sc.product_id
  GROUP BY 1, 2
  ORDER BY 3 DESC
```
*– запрос выводит сумму выручки компании по категориям и подкатегориям товаров компании*

Top-5 категорий продукции компании PP в выручке компании:

- Furniture-Chairs - 16,3%
- Technology-Phones - 15,3%
- Office supplies-Storage - 12,4%
- Technology-Accessories - 8,7%
- Furniture-Tables - 7,9%

Наименьшая доля выручки приходится на категорию Office supplies-Fasteners - 0,2%

![image](https://github.com/mikhaylovskaya-karina/Project-2_effeciency_analysis_SQL/blob/main/visual/%D0%9E%D0%B1%D1%8A%D0%B5%D0%BC%20%D0%BF%D1%80%D0%BE%D0%B4%D0%B0%D0%B6%20%D0%BF%D0%BE%20%D0%BA%D0%B0%D1%82%D0%B5%D0%B3%D0%BE%D1%80%D0%B8%D1%8F%D0%BC%20%D0%BF%D1%80%D0%BE%D0%B4%D1%83%D0%BA%D1%86%D0%B8%D0%B8.png)

Диаграмма 1. Объемы продаж компании PP по категориям продукции.

Данные говорят о том, что покупатели компании PP предпочитают хорошие офисные кресла и качественную технику связи.

**1.3 Топ-25 продуктов PP по объему продаж**

```
  WITH total_revenue AS
  (SELECT 
      round(sum(sc.quantity * sp.price * (1 - sc.discount)), 2) total_revenue
   FROM sql.store_products sp
   JOIN sql.store_carts sc ON sp.product_id = sc.product_id)
```
*– CTE выводит общую сумму выручки компании*

```
  SELECT 
      DISTINCT sp.product_nm,
      round(sum(sc.quantity * sp.price * (1 - sc.discount)), 2) revenue,
      round(sum(sc.quantity), 2) quantity,
      round(sum(sc.quantity * sp.price * (1 - sc.discount)) / tr.total_revenue * 100, 2) percent_from_total
  FROM sql.store_products sp
  JOIN sql.store_carts sc ON sp.product_id = sc.product_id
  CROSS JOIN total_revenue tr
  GROUP BY 1, tr.total_revenue
  ORDER BY 2 DESC
  LIMIT 25
```
*– запрос выводит данные по топ-25 товарам компании по объему выручки (наименование товара, сумма выручки,проданное количество, процент от общей выручки)*

На топ-25 по объемам продаж продукции компании PP приходится 18,13% от всех продаж.
Самым продаваемым в объеме выручки товаром является копировальный аппарат Canon imageCLASS 2200 Advanced Copier - 2,56%.
Самым продаваемым товаром в категории Furniture является кресло - HON 5400 Series Task Chairs for Big and Tall.
Можно предположить, что данные товары представляют больший интерес для B2B-клиентов.


## 2. Определение портрета клиента компании PP

**2.1 Объемы продаж компании PP по категориям клиентов**

```
  SELECT 
      cust.category,
      count(DISTINCT cust.cust_id) cust_cnt,
      round(sum(c.quantity * p.price * (1 - c.discount))) revenue
  FROM sql.store_carts c
  JOIN sql.store_products p ON p.product_id = c.product_id
  JOIN sql.store_delivery d ON d.order_id = c.order_id
  JOIN sql.store_customers cust ON cust.cust_id = d.cust_id
  GROUP BY 1
  ORDER BY 3 DESC
```
*– запрос выводит данные по количеству и объему выручки по категориям клиентов компании*

| CUST_CATEGORY | CUST_QTY | REVENUE       | PERCENTAGE OF REVENUE |
| :------------ | -------: | ------------: | --------------------: |
| Corporate     |      645 | 1 172 009,08$ |                 81,04 |
| Consumer      |      148 |   274 147,64$ |                 18,96 |

![image](https://github.com/mikhaylovskaya-karina/Project-2_effeciency_analysis_SQL/blob/main/visual/%D0%94%D0%BE%D0%BB%D1%8F%20%D0%BA%D0%BB%D0%B8%D0%B5%D0%BD%D1%82%D0%BE%D0%B2%20%D0%B2%20%D0%BE%D0%B1%D1%8A%D0%B5%D0%BC%D0%B5%20%D0%BF%D1%80%D0%BE%D0%B4%D0%B0%D0%B6%20%D0%BF%D0%BE%20%D0%BA%D0%B0%D1%82%D0%B5%D0%B3%D0%BE%D1%80%D0%B8%D1%8F%D0%BC.png)

Диаграмма 2. Доля клиентов в объеме продаж компании PP по категориям клиентов.

Доля B2B-клиентов намного больше доли B2C-клиентов - 81,04% от объема выручки компании за рассматриваемый период. 
Таким образом, B2B-клиенты являются наиболее ценными клиентами для компании PP.


**2.2 Динамика появления новых B2B-клиентов компании PP по месяцам**

```
  WITH first_order_table AS
  (SELECT 
      d.cust_id,
      date_trunc('month', min(d.order_date))::date first_order_date
   FROM sql.store_delivery d
   JOIN sql.store_customers cust ON cust.cust_id = d.cust_id
   WHERE cust.category = 'Corporate'
   GROUP BY 1)
```
*– CTE выводит данные по B2B-клиентам и месяце совершения ими первой покупки*

```
  SELECT 
      fot.first_order_date month,
      count(fot.cust_id) new_custs_cnt
  FROM first_order_table fot
  GROUP BY 1
  ORDER BY 1
```
*– запрос выводит данные по количеству новых B2B-клиентов по месяцам*

![image](https://github.com/mikhaylovskaya-karina/Project-2_effeciency_analysis_SQL/blob/main/visual/%D0%94%D0%B8%D0%BD%D0%B0%D0%BC%D0%B8%D0%BA%D0%B0%20%D0%BF%D0%BE%D1%8F%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F%20%D0%BD%D0%BE%D0%B2%D1%8B%D1%85%20B2B-%D0%BA%D0%BB%D0%B8%D0%B5%D0%BD%D1%82%D0%BE%D0%B2%20%D0%BF%D0%BE%20%D0%BC%D0%B5%D1%81%D1%8F%D1%86%D0%B0%D0%BC.png)

График 2. Динамика появления новых корпоративных клиентов компании PP за период с 2017 г. по конец 2020 г.

Динамика появления новых B2B-клиентов показывает значительное снижение с начала 2018 г. В 2017 г. было привлечено много новых B2B-клиентов, а с 2018 г. привлечение новых B2B-клиентов практически не происходило.

Таким образом, основная доля клиентов это “старые” B2B-клиенты, что говорит о лояльности клиентов компании PP.
Также, можно сделать промежуточный вывод, что, поскольку более 80% выручки компании приносят именно B2B-клиенты, привлечение новых корпоративных клиентов могло бы стать новой точкой роста для компании PP.


**2.3 Основные показатели по продажам корпоративным клиентам компании PP**

```
  WITH order_table AS
  (SELECT 
      d.cust_id,
      d.order_id,
      count(DISTINCT c.product_id) product_qty,
      sum(c.quantity * p.price * (1-c.discount)) order_amt
   FROM sql.store_carts c
   JOIN sql.store_products p ON p.product_id = c.product_id
   JOIN sql.store_delivery d ON d.order_id = c.order_id
   JOIN sql.store_customers cust ON d.cust_id = cust.cust_id
   WHERE cust.category = 'Corporate'
   GROUP BY 1, 2),
```
*– CTE выводит данные о B2B-клиентах, их заказах, количестве продуктов в заказе и сумме заказа*

```
  cust_addresses AS
  (SELECT 
      cust.cust_id,
      count(DISTINCT d.zip_code) address_num
   FROM sql.store_customers cust
   JOIN sql.store_delivery d ON d.cust_id = cust.cust_id
   WHERE cust.category = 'Corporate'
   GROUP BY 1)
```
*– CTE выводит данные о B2B-клиентах и количестве их различных офисов*

```
  SELECT
      round(avg(ot.product_qty), 1) avg_product_qty,
      round(avg(ot.order_amt), 1) avg_order_amount,
      round(avg(ca.address_num), 1) avg_address_number
  FROM order_table ot
  CROSS JOIN cust_addresses ca
```
*– запрос выводит информация по основным показателям по B2B-клиентам (среднее количество в заказе, средний чек, среднее количество различных офисов клиентов)*

- Среднее количество различных продуктов в заказах корпоративных клиентов - 2,0 шт.
- Средняя сумма заказа корпоративных клиентов - 285,9 долл. США.
- Среднее количество офисов корпоративных клиентов - 6,2 шт.

Итак, основным клиентом компании PP является корпоративный клиент (B2B). На долю B2B-клиентов приходится более 80% продаж за рассматриваемый период.
B2B-клиенты представляют наибольшую ценность, потому что это наиболее лояльные клиенты, о чем свидетельствует динамика продаж и динамика появления новых B2B-клиентов. Динамика продаж имеют тенденцию роста, несмотря на значительную тенденцию снижения числа новых B2B-клиентов.
Кроме того, в среднем количество офисов B2B-клиентов - 6 шт. Что указывает на бОльшую потребность B2B-клиентов в продукции PP, чем B2C-клиентов.

Рекомендуется рассмотреть варианты привлечения новых корпоративных клиентов. Большая потребность в офисных товарах и географическая распределенность B2B-клиентов может стать новой точкой роста для компании PP.


## 3. Анализ эффективности доставок

**3.1 Определение доли заказов, доставленных клиентам с задержкой**

```
  WITH late_orders_table AS
  (SELECT 
      DISTINCT ship_mode,
      count(order_id) OVER (PARTITION BY ship_mode) orders_cnt,
      CASE
          WHEN ship_mode = 'Standard Class' THEN count(order_id) filter (WHERE    ship_date - order_date > 6) OVER (PARTITION BY ship_mode)
          WHEN ship_mode = 'Second Class' THEN count(order_id) filter (WHERE ship_date - order_date > 4) OVER (PARTITION BY ship_mode)
          WHEN ship_mode = 'First Class' THEN count(order_id) filter (WHERE ship_date - order_date > 3) OVER (PARTITION BY ship_mode)
          WHEN ship_mode = 'Same Day' THEN count(order_id) filter (WHERE ship_date - order_date > 0) OVER (PARTITION BY ship_mode)
          END late_orders_cnt
   FROM sql.store_delivery
   GROUP BY ship_mode, order_id, ship_date, order_date)
```
*– CTE выводит данные о типе доставки, количестве доставок данного типа и счетчик доставок с задержками*

```
  SELECT 
      ship_mode,
      orders_cnt,
      late_orders_cnt,
      round((orders_cnt - late_orders_cnt)::numeric/orders_cnt * 100, 2) AS "% success"
  FROM late_orders_table
  ORDER BY 4
 ```
*– запрос выводит данные по типам доставки (количество доставленных заказов, количество заказов доставленных с задержкой, процент вовремя доставленных заказов)*

| SHIP_MODE      | ORDER_CNT | LATE_ORDER_CNT | % SUCCESS |
| :------------- | --------: | -------------: | --------: |
| Second Class   |       964 |            202 |     79,05 |
| Standard Class |      2994 |            309 |     89,68 |
| Same Day       |       264 |             12 |     95,45 |
| First Class    |       787 |              1 |     99,87 |

Можно сказать, что доставки заказов осуществляются эффективно. В среднем 91,01% от всех доставок были осуществлены вовремя.
В процентном соотношении задержки чаще происходят при отправлении товаров Вторым Классом (Second Class), доставка которым должна осуществляться в течение 4 дней.

Необходимо провести анализ систематичности задержек перед тем, как делать выводы.


**3.2 Определение систематичности задержек заказов, отправленный Вторым Классом**

```
  WITH sc_status AS
  (SELECT 
      order_id,
      date_trunc('quarter', ship_date)::date quarter,
      CASE
          WHEN ship_date - order_date > 4 THEN 'late'
          ELSE 'on time'
          END dlv_status
   FROM sql.store_delivery
   WHERE ship_mode = 'Second Class'
   GROUP BY order_id, ship_date, order_date)
```
*– CTE выводит данные о заказах доставленных Вторым Классом, квартале, когда заказ был доставлен, и статус доставки вовремя или с задержкой*

```
  SELECT
      quarter,
      count(order_id) filter (WHERE dlv_status = 'late') delay_number
  FROM sc_status
  GROUP BY 1
  ORDER BY 1
```
*– запрос выводит данные о количестве задержек заказов, доставленных Вторым Классом*

![image](https://github.com/mikhaylovskaya-karina/Project-2_effeciency_analysis_SQL/blob/main/visual/%D0%94%D0%B8%D0%BD%D0%B0%D0%BC%D0%B8%D0%BA%D0%B0%20%D0%B7%D0%B0%D0%B4%D0%B5%D1%80%D0%B6%D0%B5%D0%BA%20%D0%B4%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BE%D0%BA%20Second%20Class.png)

График 3. Динамика задержек доставок заказов, отправленных Вторым Классом, по кварталам за период с 2017 г. по конец 2020 г.

Динамика задержек доставок Вторым Классом имеет постоянный систематический характер. Наибольшее число задержек происходит в четвертом квартале, когда происходит сезонный рост продаж. Можно сделать вывод о корреляции роста продаж и задержек Вторым Классом.
Поскольку доставка Вторым Классом является популярный типом доставки, который выбирают клиенты (2-ое место по числу доставок), необходимо рассмотреть возможности привлечения больших сил для доставки данным типом, либо рассмотреть другие транспортные компании.
Иначе, увеличение объема продаж может привести к увеличению числа задержек Вторым Классом и возможно недовольству клиентов. Так, в четвертом квартале 2020 г. количество задержек приняло максимальное значение за рассматриваемый период, так же как и объем продаж компании PP.


**3.3 Выбор места для открытия офлайн-магазина компании PP**

```
  SELECT 
      DISTINCT state,
      count(order_id) dlv_cnt
  FROM sql.store_delivery
  GROUP BY 1
  ORDER BY 2 DESC
```
*– запрос выводит данные по количеству доставленных заказов покупателям по штатам*

География доставок заказов компании PP обширна и распространяется на 49 штатов США.

![image](https://github.com/mikhaylovskaya-karina/Project-2_effeciency_analysis_SQL/blob/main/visual/%D0%93%D0%B5%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%8F%20%D1%80%D0%B0%D1%81%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F%20%D0%B4%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BE%D0%BA.png)

Рисунок 1. География распределения доставок компании PP по штатам США в период с 2017 г. по конец 2020 г.

Наиболее популярным штатом по количеству доставок является Калифорния (California). Самым же популярным городом является Нью-Йорк (New York City).
Доля Калифорнии по доставкам составляет более 23%. Также на долю Калифорнии приходится наибольшее число доставок B2B-клиентам.

```
  SELECT 
      DISTINCT d.state,
      count(DISTINCT d.cust_id) b2b_cnt
  FROM sql.store_delivery d
  JOIN sql.store_customers c ON c.cust_id = d.cust_id
  WHERE c.category = 'Corporate'
  GROUP BY d.state
  ORDER BY 2 DESC
```
*– запрос выводит данные по количеству доставок заказов B2B-клиентов по штатам*

![image](https://github.com/mikhaylovskaya-karina/Project-2_effeciency_analysis_SQL/blob/main/visual/%D0%93%D0%B5%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%8F%20%D1%80%D0%B0%D1%81%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F%20%D0%B4%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BE%D0%BA%20b2b.png)

Рисунок 2. География распределения доставок B2B-клиентам компании PP по штатам США в период с 2017 г. по конец 2020 г.

Кроме того, на Калифорнию приходится наибольшая доля выручки компании PP за рассматриваемый период - 20,2%.

```
  SELECT 
      DISTINCT d.state,
      round(sum(c.quantity * p.price * (1-c.discount)) OVER (PARTITION BY d.state), 2) revenue
  FROM sql.store_delivery d
  JOIN sql.store_carts c ON c.order_id = d.order_id
  JOIN sql.store_products p ON p.product_id = c.product_id
  ORDER BY 2 DESC
```
*– запрос выводит данные по распределению выручки компании по штатам*

![image](https://github.com/mikhaylovskaya-karina/Project-2_effeciency_analysis_SQL/blob/main/visual/%D0%A0%D0%B0%D1%81%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5%20%D0%B2%D1%8B%D1%80%D1%83%D1%87%D0%BA%D0%B8%20%D0%BF%D0%BE%2025%20%D1%88%D1%82%D0%B0%D1%82%D0%B0%D0%BC.png)

Диаграмма 3. Распределение выручки компании PP по 25 штатам США, приносящим наибольшую выручку, за период с 2017 г. по конец 2020 г.

Таким образом, на долю Калифорнии приходятся наибольшее количество клиентов, в т.ч. B2B-клиентов, и наибольшая доля выручки компании PP. Это делает Калифорнию наиболее выгодным и привлекательным штатом для расположению офлайн-магазина.
Также, из-за близости клиентов расположение офлайн-магазина в Калифорнии поможет решить проблему с задержками доставок.

## ЗАКЛЮЧЕНИЕ

Анализ деятельности компании PP показывает, что в целом компания работает эффективно, но есть проблемы, решение которых может привести к росту.

Динамика продаж в рассматриваемом периоде имеет тенденцию роста, однако носит сезонный характер. Наблюдается падение продаж в начале года и рост в последнем квартале. Компании PP стоит рассмотреть варианты стимулирования продаж в сезонные периоды спада.

Анализ продаж по категориям и подкатегориям продукции компании показывает, что самыми популярными категориями является Furniture-Chairs и Technology-Phones, что говорит о предпочтения клиентов компании - удобные кресла и техника связи. Также, в общем объеме выручки наиболее продаваемым товаром является копировальный аппарат Canon imageCLASS 2200 Advanced Copier. Это товар с длительным сроком службы, т.е. компания не ожидает скорых повторных продаж данного товара. Эту информации можно использовать для определения дальнейшей маркетинговой стратегии.

Основными клиентами компании PP является B2B-клиенты. На данных клиентов приходится более 80% от общего объема выручки компании за рассматриваемый период. В 2017 г. компания активно привлекала новых B2B-клиентов, но с 2018 г. динамика появления новых B2B-клиентов имеет явную тенденцию спада. Можно сделать вывод, что B2B-клиенты компании довольны товарами и предоставляемым сервисом. Таким образом, портрет клиента компании PP следующий - лояльные B2B-клиенты. Однако, компании стоит рассмотреть варианты привлечения новых B2B-клиентов, что может стать новой точкой роста.

Анализ логистики компании также в целом свидетельствует об ее эффективности. Более 90% всех доставок заказов были осуществлены вовремя. 

Чаще всего задержки происходят при доставке заказов Вторым Классом. Примерно 20% доставок осуществлялись с задержками. Задержки носят сезонных характер и коррелируют с ростом продаж. Необходимо принять меры по решению данной проблемы, потому что рост продаж приведет к росту числа задержек, что может вызвать недовольство клиентов.

География распределения клиентов компании обширна. У компании есть клиенты в 49 штатах США. Наибольшее число клиентов, в т.ч. и B2B-клиентов, находятся в штате Калифорния. Также, наибольшую выручку приносят именно клиенты из Калифорнии. Что делает Калифорнию наиболее привлекательной для открытия там офлайн-магазина. Открытие магазина в Калифорнии также поможет решить проблему задержек доставок заказов благодаря близости основных клиентов компании.
