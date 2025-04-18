# 🔁 Denormalizacja – kiedy porządek nie wystarcza

## 📘 Co to jest denormalizacja?

**Denormalizacja** to proces **celowego łączenia danych z wielu znormalizowanych tabel w jedną tabelę**, nawet jeśli prowadzi to do powtórzeń danych. Głównym celem jest **zwiększenie wydajności odczytu danych** i uproszczenie zapytań.

---

## 🧩 Przykład – wersja znormalizowana (3NF)

Wyobraźmy sobie typowe znormalizowane tabele:

### 👤 Tabela: `customers`
| customer_id | customer_name |
|-------------|----------------|
| 1           | Anna Kowalska  |
| 2           | Jan Nowak      |

### 🛒 Tabela: `orders`
| order_id | customer_id | product | quantity |
|----------|-------------|---------|----------|
| 101      | 1           | TV      | 2        |
| 102      | 1           | Laptop  | 1        |
| 103      | 2           | Phone   | 3        |

Aby pobrać dane o zamówieniach z nazwą klienta, trzeba wykonać **JOIN**:
```sql
SELECT o.order_id, c.customer_name, o.product, o.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

---

## 🧾 Przykład – wersja zdenormalizowana

W wersji zdenormalizowanej dane wyglądają tak:

| order_id | customer_id | customer_name | product | quantity |
|----------|-------------|----------------|---------|----------|
| 101      | 1           | Anna Kowalska  | TV      | 2        |
| 102      | 1           | Anna Kowalska  | Laptop  | 1        |
| 103      | 2           | Jan Nowak      | Phone   | 3        |

Zalety:
- brak potrzeby JOINów,
- prostsze zapytania,
- szybszy odczyt w systemach analitycznych.

Wady:
- jeśli Anna Kowalska zmieni nazwisko, trzeba zaktualizować wiele rekordów,
- większe ryzyko błędów i niespójności danych.

---

## 🧠 Kiedy warto stosować denormalizację?

✅ Gdy system obsługuje **wiele zapytań odczytujących** dane,

✅ Gdy dane są używane **głównie do raportowania lub analityki**, a nie do intensywnego edytowania,

✅ Gdy zależy nam na **wydajności, niekoniecznie na czystości modelu danych**.**wydajności, niekoniecznie na czystości modelu danych**.

---

## 🛠️ Skrypt SQL – zdenormalizowana tabela
```sql
CREATE TABLE denormalized_orders (
    order_id INT,
    customer_id INT,
    customer_name VARCHAR(100),
    product VARCHAR(100),
    quantity INT
);

INSERT INTO denormalized_orders VALUES
(101, 1, 'Anna Kowalska', 'TV', 2),
(102, 1, 'Anna Kowalska', 'Laptop', 1),
(103, 2, 'Jan Nowak', 'Phone', 3);
```

---

## 🔍 Podsumowanie

- Denormalizacja to **świadomy kompromis** między integralnością danych a wydajnością.
- Nie zastępuje dobrego projektowania – jest jego **uzupełnieniem** w sytuacjach, gdzie **czas odpowiedzi systemu jest kluczowy**.

🧠 **Dobra praktyka:** Zacznij od normalizacji, a dopiero później – jeśli to konieczne – zdecyduj o denormalizacji wybranych fragmentów.

Chcesz zobaczyć porównanie wydajności albo inne przykłady z życia? 😎

