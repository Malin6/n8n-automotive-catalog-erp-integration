# Zautomatyzowany ETL do integracji katalogów części motoryzacyjnych (n8n)

Kompleksowy proces integracyjny ETL zbudowany w środowisku **n8n**, automatyzujący pozyskiwanie, normalizację, kalkulację marż oraz eksport danych o części nieznalezione na stronie WWW od wielu dostawców motoryzacyjnych bezpośrednio do formatów obsługiwanych przez systemy ERP.

---

## Główne funkcjonalności

* **Orkiestracja REST API:** Pełna obsługa autoryzacji `OAuth 2.0` (Client Credentials) oraz asynchroniczne odpytywanie endpointów katalogowych i cenowych dystrybutorów (m.in. Inter Cars WebAPI).
* **Fallback Web Scraping:** Ekstrakcja danych w locie z odpowiedzi HTML przy użyciu biblioteki `Cheerio` dla dostawców nieudostępniających pełnego REST API.
* **Integracja z PostgreSQL:** Bezpośrednie kwerendy SQL dopasowujące cenniki, grupy towarowe oraz automatyczny audyt pozycji nieodnalezionych (`INSERT INTO nieznalezione`).
* **Algorytmy normalizacji i deduplikacji:** Skrypty w JavaScript (ES6+) generujące permutacje indeksów (kody OEM, formatowanie z myślnikami, formaty Bosch, dynamiczne prefiksy marek) oraz eliminujące duplikaty.
* **Silnik kalkulacji marż i rabatów:** Dynamiczne łączenie danych wejściowych z tabelami rabatowymi marek oraz automatyczne wyliczanie cen zakupu i sprzedaży.
* **Generowanie plików ERP:** Przygotowanie i automatyczny eksport wielokolumnowych zestawień w formacie `.xls` (w układzie kartotek głównych towaru) na dysk Google Drive, z ktorego pliki trafiają bezposrednio do systemu ERP.

---

## Tech Stack

| Kategoria | Technologie |
| **Silnik integracji** | n8n |
| **Języki skryptowe** | JavaScript (Node.js / ES6+), SQL |
| **Baza danych** | PostgreSQL |
| **Biblioteki** | Cheerio |
| **Protokoły i formaty** | REST API, OAuth 2.0, HTTP, JSON, XLS |
| **Integracje chmurowe** | Google Drive API |

---

## Architektura przepływu (Workflow Architecture)

1. **Pobranie i przygotowanie wsadu:**
   * Pobranie arkusza roboczego wrzuconego przez system ERP(części szukane przez klientów na stronie WWWW z wartością zwrotną "nie znalezione") z Google Drive, ekstrakcja wierszy i wstępne czyszczenie ciągów znaków (usunięcie białych znaków, deduplikacja po kodzie części).
2. **Kaskadowe odpytywanie źródeł (Cascade Lookup):**
   * **Poziom 1 (API):** Generowanie wariantów indeksu z prefiksami i odpytanie API dystrybutora. W przypadku trafienia — pobranie aktualnych wycen netto przez dedykowany endpoint wycen oraz innych danych potrzebnych do prawidłowego zaczytu w systemie ERP.
   * **Poziom 2 (Scraping):** Jeśli API nie zwróci wyników, przepływ kieruje zapytanie do następnego dostawcy m.in do katalogów webowych, przeszukując strukturę tabeli za pomocą `Cheerio`.
   * **Poziom 3 (Baza SQL):** Równoległe przeszukiwanie lokalnych tabel produktowych dostawców alternatywnych w bazie PostgreSQL.
3. **Mapowanie i rekombinacja danych:**
   * Dopasowanie kodów dostawców do ujednoliconego słownika marek (OE / Aftermarket).
   * Dołączenie wartości procentowych rabatów z pliku macierzystego.
4. **Eksport i logowanie:**
   * Podział strumienia na dedykowane pliki importowe, mające na celu prawidłowe wgranie do systemu ERP.
   * Konwersja do `.xls` i automatyczny upload na Google Drive, z ktorego pliki trafiają bezposrednio do systemu ERP.
   * Zapis kodów części całkowicie nieodnalezionych do bazy danych w celach audytowych.



> **Informacja o anonimizacji:**  
> Wszelkie wrażliwe dane produkcyjne — w tym klucze API, tokeny dostępowe, pliki cookies sesji, identyfikatory kont oraz adresy baz danych — zostały usunięte lub zastąpione wartościami zastępczymi (`PLACEHOLDER`) na potrzeby publicznej prezentacji projektu.
