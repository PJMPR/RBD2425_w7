# 🔹 Druga postać normalna (2NF)

## 🔄 Przypomnienie 1NF

Tabela spełnia pierwszą postać normalną (1NF), jeśli wszystkie kolumny zawierają wartości atomowe. Przykładem była tabela `normalized_orders`, gdzie dane o produktach i klientach zostały rozbite na pojedyncze wiersze.

---

## ❌ Przykład tabeli niespełniającej założeń 2NF

Załóżmy, że mamy następującą tabelę:

| order_id | customer_id | customer_name  | product_id | product_name | quantity |
|----------|-------------|----------------|------------|---------------|----------|
| 1        | 1           | Anna Kowalska  | 101        | TV            | 2        |
| 2        | 1           | Anna Kowalska  | 102        | Laptop        | 1        |
| 3        | 2           | Jan Nowak      | 103        | Phone         | 3        |
| 4        | 2           | Jan Nowak      | 101        | TV            | 1        |

🔍 **Co jest nie tak?**
- Informacje takie jak `customer_name` i `product_name` zależą tylko od części klucza złożonego (`customer_id`, `product_id`).
- Mamy **zależności częściowe**, czyli kolumny zależą od fragmentu klucza głównego.

---

## ✅ Tabele spełniające 2NF

Aby poprawić strukturę i spełnić 2NF, dzielimy dane na odrębne tabele:

### 🧾 Tabela: `orders`

| order_id | customer_id |
|----------|-------------|
| 1        | 1           |
| 2        | 1           |
| 3        | 2           |
| 4        | 2           |

### 👤 Tabela: `customers`

| customer_id | customer_name  |
|-------------|----------------|
| 1           | Anna Kowalska  |
| 2           | Jan Nowak      |

### 🛍️ Tabela: `products`

| product_id | product_name |
|------------|--------------|
| 101        | TV           |
| 102        | Laptop       |
| 103        | Phone        |

### 📦 Tabela: `order_items`

| order_id | product_id | quantity |
|----------|------------|----------|
| 1        | 101        | 2        |
| 2        | 102        | 1        |
| 3        | 103        | 3        |
| 4        | 101        | 1        |

🔍 **Dlaczego to działa?**
- Każda kolumna zależy od **całego** klucza głównego.
- Eliminujemy częściowe zależności.
- Ułatwiamy utrzymanie i aktualizację danych.

---

## 📖 Definicja: Druga postać normalna (2NF)

Tabela znajduje się w **drugiej postaci normalnej**, jeśli:
- spełnia założenia **1NF**,
- wszystkie kolumny **zależne są od całego klucza głównego**, a nie tylko jego części (brak zależności częściowych).

---

## 🧪 Skrypty SQL

### 🔧 Tabela niespełniająca 2NF:
```sql
CREATE TABLE order_data (
    order_id INT,
    customer_id INT,
    customer_name VARCHAR(100),
    product_id INT,
    product_name VARCHAR(100),
    quantity INT
);

INSERT INTO order_data VALUES
(1, 1, 'Anna Kowalska', 101, 'TV', 2),
(2, 1, 'Anna Kowalska', 102, 'Laptop', 1),
(3, 2, 'Jan Nowak', 103, 'Phone', 3),
(4, 2, 'Jan Nowak', 101, 'TV', 1);
```

### ✅ Tabele w 2NF:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

INSERT INTO customers VALUES
(1, 'Anna Kowalska'),
(2, 'Jan Nowak');

INSERT INTO products VALUES
(101, 'TV'),
(102, 'Laptop'),
(103, 'Phone');

INSERT INTO orders VALUES
(1, 1),
(2, 1),
(3, 2),
(4, 2);

INSERT INTO order_items VALUES
(1, 101, 2),
(2, 102, 1),
(3, 103, 3),
(4, 101, 1);
```

---


