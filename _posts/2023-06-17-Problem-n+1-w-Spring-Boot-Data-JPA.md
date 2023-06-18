---
published: true
---
![]({{site.baseurl}}/images/znak3.png)

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
  
  
Klasa repozytorium zawiera jedną metodę, którą wykorzystamy w klasie testowej:

{% highlight java %}
public interface BookOrderRepository extends JpaRepository<BookOrder, Long> {
    List<BookOrder> findByUserId(Long userId);
}
{% endhighlight %}
  
    
Do utworzenia tabel oraz wypełnienia ich danymi testowymi skorzystałem z narzędzia Flyway (zależność do Flyway można znaleźć w Spring initializer):

Tworzę obie tabele:
  
V1__Create_Tables.sql:  
<pre><code class="java">/* 
CREATE TABLE BOOK_ORDER(ID BIGINT AUTO_INCREMENT PRIMARY KEY, USER_ID BIGINT, NAME VARCHAR(255));
CREATE TABLE BOOK(ID BIGINT AUTO_INCREMENT PRIMARY KEY,BOOK_ORDER_ID BIGINT,FOREIGN KEY (BOOK_ORDER_ID) REFERENCES BOOK_ORDER, NAME VARCHAR(255), PRICE DOUBLE);
</code></pre>
Oraz wypełniam je danymi testowymi:

V2__data.sql:
<pre><code class="java">/* 
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
</code></pre>  

W klasie testowej, przeprowadzamy jedną opercję, szukam ile książek w sumie zakupił klient o userId=1:
  
{% highlight java %}
@SpringBootTest
public class SomeTest {
    @Autowired
    private BookOrderRepository bookOrderRepository;

    @Test
    @Transactional
    void shouldCountBooksFromUserId(){

        List<BookOrder> byUserId = bookOrderRepository.findByUserId(1L);

        List<Book> list = byUserId.stream()
                .map(BookOrder::getBooks)
                .flatMap(Collection::stream)
                .toList();

        assertEquals(10,list.size());
    }
  }
{% endhighlight %}
  

## Logowanie zapytań SQL
  
Aby unaocznić, ile faktycznie zapytań Hibernate wysłał do bazy danych w pliku properties należy ustawić właściwość:
  
spring.jpa.show-sql=true

Po uruchomieniu test w logach zobaczymy, iż Hibernate wykonał aż 7 zapytań:
  
<code-fence lang="java" heading="Console">

Hibernate: select b1_0.id,b1_0.name,b1_0.user_id from book_order b1_0 where b1_0.user_id=?
Hibernate: select b1_0.book_order_id,b1_0.id,b1_0.name,b1_0.price from book b1_0 where b1_0.book_order_id=?
Hibernate: select b1_0.book_order_id,b1_0.id,b1_0.name,b1_0.price from book b1_0 where b1_0.book_order_id=?
Hibernate: select b1_0.book_order_id,b1_0.id,b1_0.name,b1_0.price from book b1_0 where b1_0.book_order_id=?
Hibernate: select b1_0.book_order_id,b1_0.id,b1_0.name,b1_0.price from book b1_0 where b1_0.book_order_id=?
Hibernate: select b1_0.book_order_id,b1_0.id,b1_0.name,b1_0.price from book b1_0 where b1_0.book_order_id=?
Hibernate: select b1_0.book_order_id,b1_0.id,b1_0.name,b1_0.price from book b1_0 where b1_0.book_order_id=?

</code-fence> 
  
Pierwsze zapytanie jest o wszystkie zamówienia dla klienta userId=1, było ich 6 i dodatkowo dla każdego z 6 zamówień zostało wykonane zapytanie o listę książek. Mamy przykłd niewydajnego problemu n+1 (6+1 = 7)
  
W relacji OneToMany dane z tabeli zależnej są "dociągane" gdy następuje do nich odwołanie na otwartej transakcji. Jest to tzw Lazy Loading. Takie ustawienie jest domyślne dla tego typu relacji. 
  
Sprawdźmy zatem ile będzie zapytań gdy zmienimy w klasie encji BookOrder Lazy na Eager:
  
<pre><code class="java">/*   
@OneToMany(fetch = FetchType.EAGER)
@JoinColumn(name = "BOOK_ORDER_ID")
List<Book> books = new ArrayList<>();
</code></pre>     

Nie będę wklejał logów ponownie, ale uwierzcie, że są identyczne jak w przypadku Lazy. Taka sama liczba zapytań wysłanych do bazy danych. Nie dość, że dane z tabeli zależnej zostałyby pobrane nawet bez odwołania się do nich, to dodatkowo problem n+1 nie zniknął.
  
## Rozwiązanie problemu n+1
  
**Metoda 1.**
Problem możemy rozwiązać wymuszając zapytanie z łączeniem obu tabel. Możemy zatem zmienić naszą metodę w repozytorium:
<pre><code class="java">/*  
@Query("select distinct b from BookOrder b join fetch b.books where b.userId = ?1")
List<BookOrder> findByUserId(Long userId);
</code></pre> 

Uruchamiając test ponownie widzimy już tylko jedno zapytanie:

<pre><code class="java">/*  
Hibernate: select distinct b1_0.id,b2_0.book_order_id,b2_0.id,b2_0.name,b2_0.price,b1_0.name,b1_0.user_id from book_order b1_0 join book b2_0 on b1_0.id=b2_0.book_order_id where b1_0.user_id=?
</code></pre>

Problem n+1 rozwiązany. Wprawdzie dane z tabeli podrzędnej pobrane zostałyby nawet bez odwołania się do nich (ładownie leniwe tutaj nie zadziałało) ale przynajmniej wykonało się to optymalnie jednym zapytaniem.
  
**Metoda 2.**
Jeżeli powyższy sposób nie końca odpowiada, i nie zawsze będziemy potrzebowali, aby dane z tabeli podrzędnej były pobierane, możemy przyjżeć się innemu rozwiązaniu. Pomoże nam w tym adnotacja pochodząca już bezpośrednio z Hibernate @Fetch oraz odpowiedni FetchMode.

Usuwam w repozytorium adnotację @Query a dodaję w encji nadrzędnej BookOrder adnotacje @Fetch:
<pre><code class="java">/*
@OneToMany
@Fetch(FetchMode.SUBSELECT)
@JoinColumn(name = "BOOK_ORDER_ID")
List<Book> books = new ArrayList<>();
</code></pre>  

W logach widać teraz dwa zapytania:

<pre><code class="java">/*  
Hibernate: select b1_0.id,b1_0.name,b1_0.user_id from book_order b1_0 where b1_0.user_id=?
Hibernate: select b2_0.book_order_id,b2_0.id,b2_0.name,b2_0.price from book b2_0 where b2_0.book_order_id in(select b1_0.id from book_order b1_0 where b1_0.user_id=?)
  
  
Pierwsze zapytanie jest o zamówienia, drugie zapytania pojawiło się gdy następiło odwołanie do tabeli podrzędnej. Drugie zapytanie widać, że korzysta z wyniku podzapytania. 
  
Mamy zatem przykład leniwego ładowania oraz braku problemy n+1. Powinno to zadowolić tego, któremu nie do końca pasowało rozwiązanie pierwsze.
