# 🔄 Transakcje: Przenoszenie danych do 1NF przy dużej liczbie rekordów (MariaDB)

W tym dokumencie przedstawiamy **praktyczne podejście do przenoszenia danych z tabel niespełniających 1NF** do znormalizowanych struktur – z uwzględnieniem dużych wolumenów danych. Zamiast ręcznych `INSERT`ów, korzystamy z **kursorów i procedur SQL w MariaDB**.

---

## 🧱 Przykład 1: Lista wartości w jednej kolumnie (np. produkty w zamówieniu)

### ❌ Tabela źródłowa `raw_orders`
| customer_id | customer_name  | orders                        |
|-------------|----------------|-------------------------------|
| 1           | Anna Kowalska  | TV:2, Laptop:1                |
| 2           | Jan Nowak      | Phone:3, TV:1                 |

### ✅ Docelowa tabela `normalized_orders`
| order_id | customer_id | customer_name  | product_name | quantity |
|----------|-------------|----------------|--------------|----------|
| 1001     | 1           | Anna Kowalska  | TV           | 2        |
| 1002     | 1           | Anna Kowalska  | Laptop       | 1        |
| 1003     | 2           | Jan Nowak      | Phone        | 3        |
| 1004     | 2           | Jan Nowak      | TV           | 1        |

### 🔁 Automatyczna transakcja w MariaDB:
```sql
-- 🔧 Tworzymy procedurę SQL, która zrealizuje całą migrację
DELIMITER //
CREATE PROCEDURE normalize_raw_orders()
BEGIN
  DECLARE done INT DEFAULT FALSE;
  DECLARE cid INT;
  DECLARE cname VARCHAR(100);
  DECLARE ord TEXT;
  -- 👁️‍🗨️ Kursor do odczytu danych z tabeli źródłowej
  DECLARE cur CURSOR FOR SELECT customer_id, customer_name, orders FROM raw_orders;
  -- 🛑 Gdy skończy się FETCH, ustawiamy flagę 'done' jako TRUE
  -- 🛑 Handler ustawia 'done = TRUE', gdy nie ma więcej wierszy do pobrania
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

  DECLARE item TEXT;
  DECLARE prod_name VARCHAR(100);
  DECLARE qty INT;

  -- 🔢 Ustawiamy początkową wartość order_id
  SET @oid := 1001;

  -- ✅ Rozpoczynamy transakcję
  -- ✅ Rozpoczynamy transakcję
  START TRANSACTION;
  OPEN cur;
  read_loop: LOOP
    -- ⏬ Pobieramy kolejny wiersz z kursora
    FETCH cur INTO cid, cname, ord;
    IF done THEN LEAVE read_loop; END IF;

    -- 🔁 Pętla rozdzielająca poszczególne pozycje z kolumny 'orders'
    WHILE LOCATE(',', ord) > 0 OR LENGTH(ord) > 0 DO
      -- 🧩 Pobieramy pierwszy element z listy (np. "TV:2")
      SET item = TRIM(SUBSTRING_INDEX(ord, ',', 1));
      -- 🔄 Usuwamy przetworzony element z listy 'ord'
      SET ord = TRIM(SUBSTRING(ord, LENGTH(item) + 2));
      -- 🏷️ Rozbijamy element na nazwę produktu i ilość
      SET prod_name = TRIM(SUBSTRING_INDEX(item, ':', 1));
            SET qty = TRIM(SUBSTRING_INDEX(item, ':', -1));

      -- 💾 Wstawiamy znormalizowany wiersz do docelowej tabeli
      INSERT INTO normalized_orders (order_id, customer_id, customer_name, product_name, quantity)
      VALUES (@oid, cid, cname, prod_name, qty);

      -- ➕ Zwiększamy identyfikator zamówienia
      SET @oid := @oid + 1;
      -- 🚪 Jeśli lista 'ord' jest pusta, przerywamy wewnętrzną pętlę
      IF ord = '' THEN LEAVE; END IF;
    END WHILE;
    END LOOP;
  -- 🔒 Zamykanie kursora po zakończeniu przetwarzania
  -- 🔒 Zamykanie kursora po zakończeniu przetwarzania
  CLOSE cur;
  -- 💾 Zatwierdzamy całą transakcję
  -- 💾 Zatwierdzamy wszystkie zmiany jako jedną transakcję
  COMMIT;
END //
DELIMITER ;

CALL normalize_raw_orders();
DROP PROCEDURE normalize_raw_orders;
```

---

## 🧱 Przykład 2: Zagnieżdżony adres jako JSON

### ❌ Tabela `raw_customers`
| customer_id | customer_name  | address                                      |
|-------------|----------------|----------------------------------------------|
| 1           | Anna Kowalska  | { "street": "Zielona", "city": "Łódź" }     |
| 2           | Jan Nowak      | { "street": "Lipowa", "city": "Warszawa" }  |

> Ponieważ MariaDB nie ma pełnego wsparcia dla typu `JSON`, w praktyce dane te traktuje się jako tekst i przetwarza parserem po stronie aplikacji (np. w Pythonie).

### ✅ Struktura docelowa:
- `addresses(address_id, street, city)`
- `customers(customer_id, customer_name, address_id)`

W tym przypadku polecane jest:
- przygotowanie skryptu w Pythonie, który sparsuje JSON,
- wygeneruje `INSERT`y do `addresses`,
- zmapuje `address_id` i wstawi dane do `customers`.

---

## 🧱 Przykład 3: Kolumny powtarzające się (ulubione produkty)

### ❌ Tabela `raw_favorites`
| customer_id | customer_name  | favorite_product_1 | favorite_product_2 | favorite_product_3 |
|-------------|----------------|---------------------|---------------------|---------------------|
| 1           | Anna Kowalska  | TV                  | Laptop              | Phone               |
| 2           | Jan Nowak      | Tablet              | Monitor             | NULL                |

### ✅ Docelowa tabela `favorite_products`
| customer_id | customer_name  | product_name |
|-------------|----------------|---------------|
| 1           | Anna Kowalska  | TV            |
| 1           | Anna Kowalska  | Laptop        |
| 1           | Anna Kowalska  | Phone         |
| 2           | Jan Nowak      | Tablet        |
| 2           | Jan Nowak      | Monitor       |

### 🔁 Automatyzacja przez kursory:
```sql
-- 🔧 Tworzymy procedurę SQL, która znormalizuje dane ulubionych produktów
DELIMITER //
CREATE PROCEDURE normalize_favorites()
BEGIN
  DECLARE done INT DEFAULT FALSE;
  DECLARE cid INT;
  DECLARE cname VARCHAR(100);
  DECLARE p1 VARCHAR(100);
  DECLARE p2 VARCHAR(100);
  DECLARE p3 VARCHAR(100);

  -- 👁️‍🗨️ Kursor do odczytu danych z tabeli źródłowej
  DECLARE cur CURSOR FOR SELECT customer_id, customer_name, favorite_product_1, favorite_product_2, favorite_product_3 FROM raw_favorites;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

  START TRANSACTION;
  OPEN cur;
  read_loop: LOOP
    -- ⏬ Pobieramy dane z kursora do zmiennych lokalnych
    FETCH cur INTO cid, cname, p1, p2, p3;
    IF done THEN LEAVE read_loop; END IF;

    -- 📌 Dla każdej niepustej kolumny ulubionych produktów wykonujemy INSERT
    IF p1 IS NOT NULL THEN
            INSERT INTO favorite_products VALUES (cid, cname, p1);
    END IF;
    IF p2 IS NOT NULL THEN
            INSERT INTO favorite_products VALUES (cid, cname, p2);
    END IF;
    IF p3 IS NOT NULL THEN
            INSERT INTO favorite_products VALUES (cid, cname, p3);
    END IF;
  END LOOP;
  CLOSE cur;
  COMMIT;
END //
DELIMITER ;

CALL normalize_favorites();
DROP PROCEDURE normalize_favorites;
```

---

## ℹ️ Co to jest kursor w SQL?

**Kursor** to mechanizm w SQL (np. w MariaDB), który pozwala na **przetwarzanie danych wiersz po wierszu**, tak jakbyśmy iterowali po kolekcji w języku programowania.

Używa się go wtedy, gdy chcemy:
- pracować z danymi sekwencyjnie,
- wykonać wiele operacji dla każdego rekordu,
- zapisać logikę krok po kroku w procedurze.

W naszym przypadku:
- kursor `cur` odczytuje każdy rekord z `raw_orders`,
- każdy rekord jest przetwarzany wewnątrz pętli,
- dopiero po wykonaniu całego przetwarzania zatwierdzamy zmiany (`COMMIT`).

Kursory są **mniej wydajne niż operacje zbiorowe**, ale oferują dużą kontrolę i są przydatne przy złożonym przetwarzaniu danych.

---

## ✅ Podsumowanie

- W przypadku dużych zbiorów danych stosujemy **procedury i kursory**, zamiast ręcznego `INSERT`.
- Transakcje zapewniają bezpieczeństwo operacji i możliwość cofnięcia całości przy błędzie.
- W przypadku skomplikowanych struktur (np. JSON) najlepiej zastosować **wsparcie aplikacyjne** (np. Python z biblioteką `json`).

🔧 Dzięki takim rozwiązaniom normalizacja staje się realna do przeprowadzenia w środowiskach produkcyjnych.

