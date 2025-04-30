# Анализ данных электронной коммерции (Pet-проект)

В этом pet-проекте я провёл исследование открытого набора данных транзакций в электронной коммерции с использованием SQL-запросов и визуализировал ключевые инсайты с помощью интерактивного дешборда в **Yandex DataLens**.

---

## 📊 [Дешборд](https://datalens.yandex/4bj9wa8hg4ukq)

В **Yandex DataLens** я создал интерактивный дешборд, который включает:

- **Распределение покупателей по регионам**  
- **Объём покупок по регионам**  
- **Топ-10 самых покупаемых товаров**  
- **Средний чек по категориям**  
- **Динамика среднего чека по регионам и месяцам**  
- **Средний объём покупок по месяцам**

---

## 🗂️ Описание данных

Для анализа использовались три связанные таблицы из [набора данных о транзакциях в eCommerce](https://github.com/shubhamwolf/eCommerce-Transactions-Dataset):

| Таблица         | Кол-во строк | Основные поля                                              | Примечания                                                  |
|------------------|---------------|------------------------------------------------------------|-------------------------------------------------------------|
| **Transactions** | 1 000         | TransactionID, CustomerID, ProductID, TransactionDate, Quantity, TotalValue, Price | Без пропусков. Дата приведена к типу `DATE`.               |
| **Products**     | 100           | ProductID, ProductName, Category, Price                  | Без пропусков.                                              |
| **Customers**    | 200           | CustomerID, CustomerName, Region, SignupDate            | Без пропусков. Дата приведена к типу `DATE`.               |

Все таблицы связаны через `CustomerID` и `ProductID`.

---

## ❓ Вопросы и гипотезы

### 1. Анализ клиентов

**1.1.** Топ-10 клиентов по объёму покупок:  
```sql
SELECT
  CustomerName,
  ROUND(SUM(TotalValue), 2) AS TotalSpend
FROM Customers
INNER JOIN Transactions USING(CustomerID)
GROUP BY CustomerName
ORDER BY TotalSpend DESC
LIMIT 10;
```

**1.2.** Количество клиентов в каждом регионе:  
```sql
SELECT
  Region,
  COUNT(*) AS NumCustomers
FROM Customers
GROUP BY Region
ORDER BY NumCustomers DESC;
```

**1.3.** Сколько новых клиентов регистрируется каждый месяц:  
```sql
SELECT
  strftime('%Y-%m', SignupDate) AS YearMonth,
  COUNT(CustomerID)         AS NewCustomers
FROM Customers
GROUP BY YearMonth
ORDER BY YearMonth DESC;
```

**1.4.** Средний чек на клиента:  
```sql
SELECT
  CustomerName,
  ROUND(AVG(TotalValue), 2) AS AvgOrderValue
FROM Customers
INNER JOIN Transactions USING(CustomerID)
GROUP BY CustomerName
ORDER BY AvgOrderValue DESC;
```

**1.5.** Клиенты с наибольшим количеством заказов:  
```sql
SELECT
  CustomerName,
  COUNT(*) AS OrderCount
FROM Customers
INNER JOIN Transactions USING(CustomerID)
GROUP BY CustomerName
ORDER BY OrderCount DESC
LIMIT 10;
```

---

### 2. Анализ товаров

**2.1.** Топ-10 самых продаваемых товаров по количеству:  
```sql
SELECT
  ProductName,
  SUM(Quantity) AS TotalSold
FROM Products
INNER JOIN Transactions USING(ProductID)
GROUP BY ProductName
ORDER BY TotalSold DESC
LIMIT 10;
```

**2.2.** Категории товаров с наибольшей выручкой:  
```sql
SELECT
  Category,
  ROUND(SUM(TotalValue), 2) AS CategoryRevenue
FROM Products
INNER JOIN Transactions USING(ProductID)
GROUP BY Category
ORDER BY CategoryRevenue DESC;
```

**2.3.** Средний чек по категориям:  
```sql
SELECT
  Category,
  ROUND(AVG(TotalValue), 2) AS AvgOrderValue
FROM Products
INNER JOIN Transactions USING(ProductID)
GROUP BY Category
ORDER BY AvgOrderValue DESC;
```

**2.4.** Есть ли связь между ценой и объёмом продаж:  
```sql
SELECT
  PriceBracket,
  COUNT(*) AS TransactionsCount
FROM (
  SELECT
    CASE
      WHEN Price <= 100 THEN '<=100'
      WHEN Price BETWEEN 100 AND 200 THEN '100-200'
      WHEN Price BETWEEN 200 AND 300 THEN '200-300'
      WHEN Price BETWEEN 300 AND 400 THEN '300-400'
      ELSE '>400'
    END AS PriceBracket
  FROM Transactions
) AS Brackets
GROUP BY PriceBracket
ORDER BY TransactionsCount DESC;
```

---

### 3. Продажи во времени

**3.1.** Общая выручка по месяцам:  
```sql
SELECT
  strftime('%Y-%m', TransactionDate) AS YearMonth,
  ROUND(SUM(TotalValue), 2)      AS MonthlyRevenue
FROM Transactions
GROUP BY YearMonth
ORDER BY YearMonth DESC;
```

**3.2.** Месяцы с максимальной выручкой:  
```sql
SELECT
  strftime('%Y-%m', TransactionDate) AS YearMonth,
  ROUND(SUM(TotalValue), 2)      AS MonthlyRevenue
FROM Transactions
GROUP BY YearMonth
ORDER BY MonthlyRevenue DESC;
```

**3.3.** Объём продаж по дням недели:  
```sql
SELECT
  strftime('%w', TransactionDate) AS WeekDay,
  COUNT(*)                       AS SalesCount
FROM Transactions
GROUP BY WeekDay
ORDER BY SalesCount DESC;
```

**3.4.** Часы пиковых покупок:  
```sql
SELECT
  strftime('%H', TransactionDate) AS HourOfDay,
  COUNT(*)                       AS SalesCount
FROM Transactions
GROUP BY HourOfDay
ORDER BY SalesCount DESC;
```

---

### 4. Сквозной анализ

**4.1.** Популярные категории по регионам:  
```sql
SELECT
  Region,
  Category,
  SUM(Quantity) AS TotalSold
FROM Transactions
INNER JOIN Customers  USING(CustomerID)
INNER JOIN Products   USING(ProductID)
GROUP BY Region, Category
ORDER BY Region, TotalSold DESC;
```

**4.2.** Средний чек по регионам:  
```sql
SELECT
  Region,
  ROUND(AVG(TotalValue), 2) AS AvgOrderValue
FROM Transactions
INNER JOIN Customers USING(CustomerID)
GROUP BY Region
ORDER BY AvgOrderValue DESC;
```

---

### 5. Поведенческий анализ

**5.1.** Есть ли сезонность в объёме и выручке:  
```sql
SELECT
  strftime('%Y-%m', TransactionDate) AS YearMonth,
  SUM(Quantity)                  AS TotalQuantity,
  ROUND(SUM(TotalValue), 2)      AS TotalRevenue
FROM Transactions
GROUP BY YearMonth
ORDER BY YearMonth;
```

**5.2.** Категории с выраженной сезонностью:  
```sql
SELECT
  Category,
  strftime('%m', TransactionDate) AS Month,
  SUM(Quantity)                  AS TotalSold
FROM Transactions
INNER JOIN Products USING(ProductID)
GROUP BY Category, Month
ORDER BY Category, Month;
```

**5.3.** Доля клиентов с одной покупкой и повторных:  
```sql
SELECT
  100.0 * SUM(order_count = 1) / COUNT(*) AS OneTimePct,
  100.0 * SUM(order_count > 1) / COUNT(*) AS RepeatPct
FROM (
  SELECT
    CustomerID,
    COUNT(*) AS order_count
  FROM Transactions
  GROUP BY CustomerID
) AS CustomerOrders;
```

**5.4.** Топ-10 клиентов по LTV:  
```sql
SELECT
  CustomerName,
  ROUND(SUM(TotalValue), 2) AS LTV
FROM Customers
INNER JOIN Transactions USING(CustomerID)
GROUP BY CustomerName
ORDER BY LTV DESC
LIMIT 10;
```

---

## 📈 Основные выводы

- **Количество покупателей**:  
  Южная Америка лидирует (~60), далее Европа (~50), Северная Америка и Азия (~45).
- **Объём покупок по регионам**:  
  Южная Америка ~35%, Европа ~28%, Северная Америка ~21%, Азия ~16%.
- **Популярные товары**:  
  **ActiveWear Smartwatch** и **SoundWave Headphones** — лидеры продаж.
- **Средний чек по категориям**:  
  Одежда (~750 у.е.) и книги (~720 у.е.) — самые высокие средние чеки.
- **Тренды во времени**:  
  - В Азии в январе наблюдается всплеск среднего чека (~1500), затем стабилизация.  
  - Южная Америка демонстрирует стабильный средний чек (~800–900).  
  - Декабрь — месяц с наибольшим объёмом продаж.

Все результаты представлены в интерактивном дешборде для дальнейшего анализа и фильтрации данных.

---

## ⚙️ Возможности для развития

- Добавить сегментацию по демографическим признакам.  
- Учитывать сезонные события и промо-периоды.  
- Построить модель прогнозирования LTV клиента.  

---
