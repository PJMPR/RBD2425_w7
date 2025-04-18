# 📘 Wprowadzenie do relacyjnych baz danych: normalizacja i transakcje

Czy zawsze mamy dobrze zaprojektowaną bazę danych? 🤔 Często spotykamy się z projektami, które na pierwszy rzut oka wyglądają poprawnie, ale w rzeczywistości zawierają wiele ukrytych problemów związanych z organizacją danych.

Poniżej kilka przykładów struktur tabel, które **nie spełniają zasad normalizacji**:

### ❌ Przykład 1 – Redundancja i brak rozdzielenia danych:
```sql
CREATE TABLE orders (
    order_id INT,
    customer_name VARCHAR(100),
    customer_address TEXT,
    product_name VARCHAR(100),
    quantity INT,
    order_date TEXT
);
```
🔎 **Pytania kontrolne:**
- Czy klient powinien być reprezentowany tylko imieniem i adresem?
- Czy produkt powinien być wpisany ręcznie w każdym zamówieniu?
- Czy `order_date` jako TEXT to dobre rozwiązanie?

---

### ❌ Przykład 2 – Powielanie danych:
```sql
CREATE TABLE customers (
    customer_id INT,
    name VARCHAR(100),
    favorite_product_1 VARCHAR(100),
    favorite_product_2 VARCHAR(100),
    favorite_product_3 VARCHAR(100)
);
```
🔎 **Pytania kontrolne:**
- Czy dodanie kolejnego ulubionego produktu wymaga modyfikacji struktury tabeli?
- Czy to podejście umożliwia efektywne zapytania?

---

### ❌ Przykład 3 – Wiele wartości w jednej kolumnie:
```sql
CREATE TABLE inventory (
    product_id INT,
    warehouses TEXT -- np. "Warszawa:50,Kraków:25,Łódź:30"
);
```
🔎 **Pytania kontrolne:**
- Czy taka struktura pozwala na prostą aktualizację danych magazynowych?
- Czy można na niej efektywnie wykonywać zapytania?

---

Na powyższych przykładach widać, jak **złe projektowanie** tabel może prowadzić do:
- ❌ Redundancji
- ❌ Problemów z aktualizacjami
- ❌ Błędów logicznych i danych trudnych w analizie

---

## 🧠 Kluczowe pojęcia

Zanim przejdziemy do głównego celu materiału, warto zrozumieć kilka podstawowych pojęć, które będą często pojawiać się w dalszej części:

- 🔁 **Redundancja danych** – sytuacja, w której te same informacje są przechowywane wielokrotnie w różnych miejscach bazy danych. Prowadzi to do niepotrzebnego powielania danych, ryzyka niespójności i problemów przy aktualizacji.

- 🔗 **Zależność funkcyjna** – związek między kolumnami, gdzie wartość jednej kolumny jednoznacznie determinuje wartość innej. To podstawa przy określaniu normalnych form.

- 🔐 **Integralność danych** – zestaw reguł, które zapewniają spójność i poprawność danych w bazie (np. dzięki kluczom głównym i obcym).

- 📐 **Normalizacja** – proces dzielenia tabel i tworzenia relacji w celu eliminacji nadmiarowości i zapewnienia integralności danych.

- 🧾 **Transakcja** – grupa operacji na danych, które muszą być wykonane w całości lub wcale. Gwarantuje spójność nawet w przypadku awarii.

---

## 🎯 Cel tego materiału

W tym materiale skupimy się na dwóch kluczowych aspektach relacyjnych baz danych:

- 🧹 **Normalizacja danych** – jak uporządkować dane i poprawić strukturę bazy.
- 🔐 **Transakcje** – jak zapewnić spójność i niezawodność operacji na danych.

---

## 🗂️ Przykładowa baza danych

Do ilustracji tych zagadnień posłużymy się bazą zawierającą informacje o:

- 👤 klientach
- 📦 zamówieniach
- 🛍️ produktach
- 🏬 stanach magazynowych
- 🏠 adresach

Struktura tej bazy została zdefiniowana w plikach SQL dołączonych do repozytorium.

---

Na koniec tej lekcji:
- ✅ zaprezentujemy proces **normalizacji**, krok po kroku
- ✅ pokażemy, jak działają **transakcje** w SQL na praktycznych przykładach

Zaczynajmy! 🚀

