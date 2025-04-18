# 🔹 Trzecia postać normalna (3NF)

## 🔄 Przypomnienie 2NF

Tabela spełnia drugą postać normalną (2NF), gdy wszystkie kolumny zależą od całego klucza głównego. W poprzednim przykładzie oddzieliliśmy dane klientów, produktów i pozycji zamówień do osobnych tabel, eliminując zależności częściowe.

---

## ❌ Przykład tabeli niespełniającej założeń 3NF

| customer_id | customer_name  | city       | zip_code |
|-------------|----------------|------------|----------|
| 1           | Anna Kowalska  | Warszawa   | 00-001   |
| 2           | Jan Nowak      | Kraków     | 31-002   |
| 3           | Ewa Malinowska | Warszawa   | 00-001   |

🔍 **Co jest nie tak?**
- Kolumna `zip_code` zależy od `city`, a nie bezpośrednio od `customer_id`.
- Mamy tu **zależność przechodnią**: `customer_id` → `city` → `zip_code`
- Jeśli kod pocztowy miasta się zmieni, musimy go aktualizować w wielu rekordach.

---

## ✅ Tabele spełniające 3NF

Aby spełnić założenia 3NF, rozdzielamy dane na:

### 👤 Tabela: `customers`

| customer_id | customer_name  | city_id |
|-------------|----------------|---------|
| 1           | Anna Kowalska  | 1       |
| 2           | Jan Nowak      | 2       |
| 3           | Ewa Malinowska | 1       |

### 🏙️ Tabela: `cities`

| city_id | city      | zip_code |
|---------|-----------|----------|
| 1       | Warszawa  | 00-001   |
| 2       | Kraków    | 31-002   |

🔍 **Dlaczego to działa?**
- Wszystkie kolumny zależą **bezpośrednio od klucza głównego**.
- Usunięto zależność przechodnią.
- Łatwiej aktualizować dane miasta/kodu – wystarczy jedna zmiana w tabeli `cities`.

---

## 📖 Definicja: Trzecia postać normalna (3NF)

Tabela znajduje się w **trzeciej postaci normalnej**, jeśli:
- spełnia warunki **2NF**, oraz
- nie zawiera **zależności przechodnich** — czyli każda kolumna niebędąca częścią klucza zależy **tylko i wyłącznie** od klucza głównego.

---

## 🧪 Skrypty SQL

### 🔧 Tabela niespełniająca 3NF:
```sql
CREATE TABLE raw_customers (
    customer_id INT,
    customer_name VARCHAR(100),
    city VARCHAR(100),
    zip_code VARCHAR(10)
);

INSERT INTO raw_customers VALUES
(1, 'Anna Kowalska', 'Warszawa', '00-001'),
(2, 'Jan Nowak', 'Kraków', '31-002'),
(3, 'Ewa Malinowska', 'Warszawa', '00-001');
```

### ✅ Tabele w 3NF:
```sql
CREATE TABLE cities (
    city_id INT PRIMARY KEY,
    city VARCHAR(100),
    zip_code VARCHAR(10)
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    city_id INT,
    FOREIGN KEY (city_id) REFERENCES cities(city_id)
);

INSERT INTO cities VALUES
(1, 'Warszawa', '00-001'),
(2, 'Kraków', '31-002');

INSERT INTO customers VALUES
(1, 'Anna Kowalska', 1),
(2, 'Jan Nowak', 2),
(3, 'Ewa Malinowska', 1);
```

---

## 💡 Ciekawostka: Wyższe postaci normalne

Oprócz 1NF, 2NF i 3NF istnieją także **wyższe formy normalne**, takie jak:
- **BCNF** (Boyce-Codd Normal Form)
- **4NF**, **5NF** i **6NF**

Stosuje się je głównie w **specjalistycznych przypadkach**, np. w hurtowniach danych, systemach analitycznych lub w zastosowaniach wymagających ekstremalnej integralności. W praktyce, większość aplikacji dobrze funkcjonuje przy normalizacji do **3NF**, która zapewnia dobry balans między integralnością a wydajnością i prostotą zapytań.

---

