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
- nie zawiera:
  - 🔁 **zagregowanych list** (np. wartości oddzielonych przecinkami),
  - 📦 **zagnieżdżonych struktur** (np. JSON lub tablic w kolumnach),
  - 🧱 **powtarzających się kolumn** (np. kolumny o nazwach `product1`, `product2`, `product3`).

### 🧩 Przykłady naruszenia zasad 1NF

Poniżej trzy przykłady tabel, które naruszają różne założenia pierwszej postaci normalnej (1NF):

#### 🔁 Zagregowana lista w kolumnie:

| customer_id | customer_name  | orders                        |
|-------------|----------------|-------------------------------|
| 1           | Anna Kowalska  | TV:2, Laptop:1, Phone:3        |
| 2           | Jan Nowak      | Monitor:2, Tablet:1            |

➡️ Kolumna `orders` zawiera wartości złożone – to zagregowana lista, trudna do filtrowania czy relacji z innymi tabelami. są zapisane w jednej kolumnie jako lista – trudne do analizy.

#### 📦 Zagnieżdżona struktura (np. JSON):

| customer_id | customer_name  | address                                      |
|-------------|----------------|----------------------------------------------|
| 1           | Anna Kowalska  | { "street": "Zielona", "city": "Łódź" }     |
| 2           | Jan Nowak      | { "street": "Lipowa", "city": "Warszawa" }  |

➡️ Kolumna `address` zawiera zagnieżdżoną strukturę JSON – trudna do analizy i zapytań SQL. – nie można łatwo sortować, filtrować czy łączyć po poszczególnych polach.

#### 🧱 Powtarzające się kolumny:

| customer_id | customer_name  | favorite_product_1 | favorite_product_2 | favorite_product_3 |
|-------------|----------------|---------------------|---------------------|---------------------|
| 1           | Anna Kowalska  | TV                  | Laptop              | Phone               |
| 2           | Jan Nowak      | Tablet              | Monitor             | NULL                |

➡️ Powtarzające się kolumny powodują trudności w rozbudowie, analizie i relacjach z tabelą produktów. – trudna w skalowaniu i analizie, np. brak możliwości prostego JOIN po ulubionych produktach.

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



