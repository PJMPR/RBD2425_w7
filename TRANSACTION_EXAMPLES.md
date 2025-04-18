# 🧪 Przykłady transakcji na przykładowej bazie danych

Poniżej znajdziesz zestaw praktycznych przykładów transakcji SQL pokazujących różne sytuacje i poziomy izolacji. Wszystkie bazują na strukturze naszej przykładowej bazy danych (`orders`, `order_items`, `inventory`, `customers`, `products`).

---

## ✅ Przykład 1 – Prosta transakcja zakończona sukcesem

**Opis:** Utworzenie nowego zamówienia wraz z aktualizacją stanu magazynowego.

```sql
BEGIN;

INSERT INTO orders (order_id, customer_id, order_date)
VALUES (21, 2, CURRENT_DATE);

INSERT INTO order_items (order_id, product_id, quantity)
VALUES (21, 10, 1);

UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 10;

COMMIT;
```

📌 **Efekt:** Wszystkie zmiany zostaną zatwierdzone, dane są spójne.

---

## ❌ Przykład 2 – Transakcja z błędem i wycofaniem (ROLLBACK)

**Opis:** Próba utworzenia zamówienia na produkt, którego nie ma w magazynie.

```sql
BEGIN;

INSERT INTO orders (order_id, customer_id, order_date)
VALUES (22, 1, CURRENT_DATE);

INSERT INTO order_items (order_id, product_id, quantity)
VALUES (22, 105, 1); -- nieistniejący produkt

UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 105;

ROLLBACK;
```

📌 **Efekt:** Cała transakcja zostaje anulowana – żadne dane nie zostały zapisane.

---

## 🔄 Przykład 3 – Transakcje równoległe i izolacja: READ COMMITTED

**Opis:** Transakcja A czyta dane, które transakcja B aktualizuje – ale jeszcze nie zatwierdziła.

```sql
-- Sesja A
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN;
SELECT quantity FROM inventory WHERE product_id = 101;
-- Oczekuje COMMIT z sesji B

-- Sesja B
BEGIN;
UPDATE inventory SET quantity = quantity - 5 WHERE product_id = 101;
-- Brak COMMIT
```

📌 **Efekt:** Sesja A nie widzi zmian dopóki sesja B nie zatwierdzi danych.

---

## 🧱 Przykład 4 – SERIALIZABLE i blokada phantom reads

**Opis:** Zabezpieczenie przed phantom reads – żadna nowa pozycja nie pojawi się między zapytaniami.

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN;

SELECT * FROM order_items WHERE quantity > 1;
-- W tym czasie inna transakcja próbuje INSERT – zostanie zablokowana

-- Po analizie
COMMIT;
```

📌 **Efekt:** Transakcja blokuje możliwość wstawienia nowych danych pasujących do warunku.

---

## 🧠 Podsumowanie

- Transakcje pozwalają zapewnić bezpieczeństwo i integralność danych.
- Różne poziomy izolacji chronią przed różnymi typami błędów.
- `ROLLBACK` cofa wszystkie zmiany w obrębie transakcji.

