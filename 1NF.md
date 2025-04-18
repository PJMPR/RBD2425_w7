# 🔹 Pierwsza postać normalna (1NF)

## ❌ Przykład tabeli niespełniającej założeń 1NF

Wyobraźmy sobie strukturę danych klientów i ich zamówień:

| customer_id | customer_name  | orders                        |
|-------------|----------------|-------------------------------|
| 1           | Anna Kowalska  | TV:2, Laptop:1                |
| 2           | Jan Nowak      | Phone:3, TV:1                 |

🔍 **Co jest nie tak?**
- Kolumna `orders` zawiera **wiele wartości** – narusza to zasadę atomowości danych.
- Trudności w analizie danych (np. ile razy kupiono TV?)
- Brak możliwości tworzenia relacji z innymi tabelami (np. `products`).

---

## ✅ Tabela spełniająca 1NF

Ta sama informacja może być przedstawiona w sposób zgodny z 1NF:

| order_id | customer_id | customer_name  | product_name | quantity |
|----------|-------------|----------------|--------------|----------|
| 1        | 1           | Anna Kowalska  | TV           | 2        |
| 2        | 1           | Anna Kowalska  | Laptop       | 1        |
| 3        | 2           | Jan Nowak      | Phone        | 3        |
| 4        | 2           | Jan Nowak      | TV           | 1        |

🔍 **Dlaczego to działa?**
- Każda kolumna zawiera jedną, niepodzielną wartość (wartość atomowa).
- Dane są łatwiejsze w analizie i dalszej normalizacji.

---

## 📖 Definicja: Pierwsza postać normalna (1NF)

Tabela znajduje się w **pierwszej postaci normalnej**, jeśli:

- wszystkie kolumny zawierają **wartości atomowe** (niepodzielne),
- każdy rekord ma jednoznacznie określoną wartość dla każdej kolumny,
- nie zawiera zagnieżdżonych list, powtarzających się kolumn lub struktur.

---

## 🧪 Skrypty SQL

### 🔧 Tabela niespełniająca 1NF:
```sql
CREATE TABLE raw_orders (
    customer_id INT,
    customer_name VARCHAR(100),
    orders TEXT
);

INSERT INTO raw_orders VALUES
(1, 'Anna Kowalska', 'TV:2, Laptop:1'),
(2, 'Jan Nowak', 'Phone:3, TV:1');
```

### ✅ Tabela w 1NF:
```sql
CREATE TABLE normalized_orders (
    order_id INT,
    customer_id INT,
    customer_name VARCHAR(100),
    product_name VARCHAR(100),
    quantity INT
);

INSERT INTO normalized_orders VALUES
(1, 1, 'Anna Kowalska', 'TV', 2),
(2, 1, 'Anna Kowalska', 'Laptop', 1),
(3, 2, 'Jan Nowak', 'Phone', 3),
(4, 2, 'Jan Nowak', 'TV', 1);
```

---

