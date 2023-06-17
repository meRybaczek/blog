---
published: false
---
W dzisiejszych aplikacjach opartych na Spring Boot Data JPA, efektywne zarządzanie relacjami i pobieranie danych jest kluczowym aspektem projektowania i optymalizacji wydajności. Jednym z często spotykanych problemów, który może negatywnie wpływać na wydajność aplikacji, jest tzw. problem n+1.

Problem n+1 występuje, gdy aplikacja wykonuje n+1 zapytań SQL do bazy danych w celu pobrania danych związanych z danym zapytaniem do tabeli nadrzędnej. Innymi słowy, dla każdego rekordu z tabeli nadrzędnej, zwróconego przez zapytanie, aplikacja wykonuje dodatkowe zapytania, aby pobrać powiązane dane z tabeli będącej w relacji. To może prowadzić do dużej liczby zapytań do bazy danych i spowolnić działanie aplikacji.

## Jak zwykle, najlepiej przedstawić problem i jego rozwiązanie na przykładzie.

Czyli klasycznie mamy klasę z zamówieniem BookOrder, a w niej użytkownika (Long userId), który składa zamówienie na książki (List<Book> books), oraz nazwę zamówienia (String name). No i oczywiście indetyfikator zamówienia Long id:
  
{% highlight java %}
@Entity
public class BookOrder {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;

    Long userId;

    String name;

    @OneToMany
    @JoinColumn(name = "BOOK_ORDER_ID")
    List<Book> books = new ArrayList<>();

    public Long getUserId() {
        return userId;
    }

    public String getName() {
        return name;
    }

    public List<Book> getBooks() {
        return books;
    }

    @Override
    public String toString() {
        return "BookOrder{" +
                "id=" + id +
                ", userId=" + userId +
                ", name='" + name + '\'' +
                ", items=" + books +
                '}';
    }
}
{% endhighlight %}

  
Oraz mamy encje podrzędną Book:

{% highlight java %}
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;

    String name;
    Double price;

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public Double getPrice() {
        return price;
    }

    @Override
    public String toString() {
        return "Item{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
{% endhighlight %}

Do utworzenia tabel oraz wypełnienia ich danymi testowymi skorzystałem z narzędzia Flyway (zależność do Flyway można znaleźć w Spring initializer):

Tworzę obie tabele:
  
CREATE TABLE BOOK_ORDER(ID BIGINT AUTO_INCREMENT PRIMARY KEY, USER_ID BIGINT, NAME VARCHAR(255));
CREATE TABLE BOOK(ID BIGINT AUTO_INCREMENT PRIMARY KEY,BOOK_ORDER_ID BIGINT,FOREIGN KEY (BOOK_ORDER_ID) REFERENCES BOOK_ORDER, NAME VARCHAR(255), PRICE DOUBLE);
  
Oraz wypełniam je danymi testowymi:
  
INSERT INTO BOOK_ORDER (USER_ID, NAME) VALUES (1, 'Order 1');
INSERT INTO BOOK_ORDER (USER_ID, NAME) VALUES (2, 'Order 2');
INSERT INTO BOOK_ORDER (USER_ID, NAME) VALUES (1, 'Order 3');
INSERT INTO BOOK_ORDER (USER_ID, NAME) VALUES (1, 'Order 4');
INSERT INTO BOOK_ORDER (USER_ID, NAME) VALUES (1, 'Order 5');
INSERT INTO BOOK_ORDER (USER_ID, NAME) VALUES (3, 'Order 6');
INSERT INTO BOOK_ORDER (USER_ID, NAME) VALUES (1, 'Order 7');
INSERT INTO BOOK_ORDER (USER_ID, NAME) VALUES (1, 'Order 8');

INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (1, 'Book1', 15.15);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (2, 'Book2', 14.14);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (3, 'Book3', 25.25);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (1, 'Book4', 46.25);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (1, 'Book5', 57.35);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (3, 'Book6', 24.35);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (3, 'Book7', 25.25);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (4, 'Book8', 234.24);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (5, 'Book9', 23.34);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (6, 'Book10', 23.22);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (7, 'Book11', 54.22);
INSERT INTO BOOK (BOOK_ORDER_ID, NAME, PRICE) VALUES (8, 'Book12', 234.22);
  





