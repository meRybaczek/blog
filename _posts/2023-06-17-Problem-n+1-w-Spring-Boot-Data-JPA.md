---
published: false
---
W dzisiejszych aplikacjach opartych na Spring Boot Data JPA, efektywne zarządzanie relacjami i pobieranie danych jest kluczowym aspektem projektowania i optymalizacji wydajności. Jednym z często spotykanych problemów, który może negatywnie wpływać na wydajność aplikacji, jest tzw. problem n+1.

Problem n+1 występuje, gdy aplikacja wykonuje n+1 zapytań SQL do bazy danych w celu pobrania danych związanych z danym zapytaniem do tabeli nadrzędnej. Innymi słowy, dla każdego rekordu z tabeli nadrzędnej, zwróconego przez zapytanie, aplikacja wykonuje dodatkowe zapytania, aby pobrać powiązane dane z tabeli będącej w relacji. To może prowadzić do dużej liczby zapytań do bazy danych i spowolnić działanie aplikacji.

Jak zwykle, najlepiej przedstawię to na przykładzie. Czyli klasycznie mamy klasę z zamówieniem BookOrder, a w niej użytkownika (Long userId), który składa zamówienie na książki (List<Book> books), oraz nazwę zamówienia (String name). No i oczywiście indetyfikator zamówienia Long id:
