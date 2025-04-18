# 🔐 Transakcje w relacyjnych bazach danych

## 📘 Co to jest transakcja?

**Transakcja** to zestaw jednej lub wielu operacji na bazie danych, które muszą być wykonane jako **nierozdzielna całość**. Jeśli którakolwiek z operacji się nie powiedzie, wszystkie wcześniejsze są **cofane** – tak, jakby nigdy nie zostały wykonane.

Najczęściej stosuje się je w sytuacjach, w których konieczna jest **spójność i bezpieczeństwo danych**, np. w bankowości, e-commerce czy rejestrach medycznych.

---

## 🧪 Przykład problemu bez transakcji

Załóżmy, że klient składa zamówienie i aktualizujemy:
1. tabelę `orders` – tworzymy nowe zamówienie,
2. tabelę `inventory` – odejmujemy ilość zamówionych produktów z magazynu.

Co się stanie, jeśli **pierwszy krok się uda**, ale drugi nie (np. z powodu błędu sieci)? Baza będzie w stanie niespójnym – zamówienie istnieje, ale magazyn nie został zaktualizowany!

---

## ✅ Przykład z użyciem transakcji (na naszej bazie danych)

```sql
BEGIN;

-- 1. Dodanie nowego zamówienia\INSERT INTO orders (order_id, customer_id, order_date) VALUES (5, 1, CURRENT_DATE);

-- 2. Dodanie pozycji zamówienia
INSERT INTO order_items (order_id, product_id, quantity) VALUES (5, 101, 2);

-- 3. Aktualizacja stanu magazynowego
UPDATE inventory SET quantity = quantity - 2 WHERE product_id = 101;

COMMIT;
```

Jeśli którykolwiek z tych kroków się nie powiedzie, możemy użyć:
```sql
ROLLBACK;
```
aby **cofnąć całą transakcję** i przywrócić stan bazy do momentu przed `BEGIN;`.

---

## 🧱 Cechy transakcji – model ACID

- ⚛ **Atomicity (Atomowość)** – wszystkie operacje w transakcji wykonują się w całości albo wcale.
- ✅ **Consistency (Spójność)** – dane pozostają w spójnym stanie przed i po transakcji.
- 🚦 **Isolation (Izolacja)** – transakcje nie zakłócają się nawzajem (np. równoległe zamówienia).
- 💾 **Durability (Trwałość)** – po zatwierdzeniu (COMMIT) zmiany są trwałe, nawet po awarii.

---

## 🚦 Poziomy izolacji transakcji

| Poziom izolacji     | Co chroni                      | Możliwe problemy         |
|---------------------|-------------------------------|---------------------------|
| READ UNCOMMITTED    | prawie nic                    | brudne odczyty (dirty reads) |
| READ COMMITTED      | odczytuje tylko zatwierdzone  | niedokładne dane przy wielokrotnym zapytaniu |
| REPEATABLE READ     | stałość odczytów              | phantom reads możliwe     |
| SERIALIZABLE        | pełna izolacja                | najbezpieczniejszy, ale wolniejszy |

---

## 🧩 Wyjaśnienie typowych problemów transakcyjnych

### 🟥 Dirty Reads (brudne odczyty)
Odczyt danych, które zostały **zmodyfikowane przez inną transakcję, ale jeszcze niezatwierdzone**. Jeśli ta druga transakcja zostanie wycofana (`ROLLBACK`), dane przeczytane wcześniej stają się nieprawdziwe.

### 🟨 Non-Repeatable Reads (niedokładne dane przy powtórnym zapytaniu)
Gdy transakcja odczytuje ten sam wiersz dwukrotnie i **uzyskuje różne wyniki**, ponieważ inna transakcja zmieniła te dane pomiędzy odczytami.

### 🟪 Phantom Reads (widma odczytów)
Gdy transakcja wykonuje zapytanie z określonym warunkiem (np. `WHERE quantity > 10`), a inna transakcja **dodaje nowe wiersze**, które spełniają ten warunek. Powtórne zapytanie zwraca więcej wyników niż poprzednie.

📌 **Phantom reads** wymagają poziomu izolacji `SERIALIZABLE`, by im zapobiec.

---



---

## 💡 Gdzie używa się transakcji?

- 🏦 Bankowość – przelewy, saldo konta
- 🛒 E-commerce – przetwarzanie koszyków i płatności
- 📚 Systemy rezerwacji (loty, hotele)
- 🧾 Systemy fakturowania i księgowości

---

## 🧠 Dobre praktyki pracy z transakcjami

- Zamykaj transakcję jak najszybciej – unikaj długich transakcji blokujących innych.
- Używaj `ROLLBACK` tylko gdy naprawdę trzeba – najlepiej łap błędy wcześniej.
- Testuj scenariusze błędów – co się stanie, jeśli jedna z operacji się nie powiedzie?
- Nie łącz logiki aplikacji z logiką transakcyjną – trzymaj transakcje „blisko danych”.

---


