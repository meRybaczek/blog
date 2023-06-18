---
published: false
---

W językach takich jak C czy C++ programy są ładowane w całości podczas startu. W języku Java proces ten wygląda nieco inaczej. Skompilowane już klasy wczytywane są do pamięci dopiero gdy są potrzebne. A są potrzebne gdy zostaje wywołany konstruktor klasy lub pojawia się odniesienie do jego ststycznego pola lub statycznej metody. To właśnie wtedy zostają zainicjalizowane statyczne pola i statyczne bloki w porządku tekstowym.

Przeanalizujmy proces inicjalizacji na przykładzie klasy bazowej Insect.class i klasy dziedziczącej Beetle.class

Insect.class
{% highlight java %}  
public class Insect {
  private int i = 9;
  protected int j;
  
  Insect() {
      System.out.println("i= " + i + " j= " + j);
      j = 39;
  }
  private static int x1 = printInit("static Insect.x1 zainicjowana");
  
  static int printInit(String s) {
      System.out.println(s);
      return 47;
  }
}
{% endhighlight %}  

Beetle.class
{% highlight java %}  
public class Beetle extends Insect {
    private int k = printInit("Beetle.k zainicjowana");

    Beetle() {
        System.out.println("k = " + k);
        System.out.println("j = " + j);
    }

    private static int x2 = printInit("static Beetle.x2 zainicjowana");

    public static void main(String[] args) {
        System.out.println("Konstruktor klasy Beetle:");
        Beetle b = new Beetle();
    }
}
{% endhighlight %} 

No i zagadaka, co zobaczymy jak pierwsze po uruchomieniu metody main?
Nie, nie jest to wydruk z System.out-a.

{% highlight java %}  
static Insect.x1 zainicjowana
static Beetle.x2 zainicjowana
Konstruktor klasy Beetle:
i= 9 j= 0
Beetle.k zainicjowana
k = 47
j = 39
{% endhighlight %} 

A teraz postaram się wytłumaczyć kolejność działań inicjalizacji i wczytywania klas.

Uruchmoina zostaje statyczna metoda main z klasy Beetle. Zatem jako pierwsze JVM ładuje klasę Beetle, jednak zauważa(poprzez słowo extends), że klasa ta ma klasę bazową Insect, która jednak jako pierwsza zostaje załadowana do pamięci. Dzieje się tak niezależnie czy obiekt klasy bazowej jest tworzony czy nie. Gdyby klasa Insect posiadała swoją klasę bazową to znowu jako pierwsza zostałaby załadowana klasa bazowa itd. Ma ta kolejność sens, bo w końcu klasy dziedziczące zależą od swoich klas bazowych.

Skoro ładowana jest najpierw klasa Insect, inicjalizowane są na początku jej pola statyczne, a więc pole int x1, co doświadczamy pierwszym wydrukiem w konsoli: "static Insect.x1 zainicjowana" i przypisaniem do x1 wartości 47.

Następnie inicjalizowane są pola statyczne klasy wywołującej(dziedziczącej) Beetle. A więc podobnie jak poprzednio inicjalizowane jest pole statyczne int x2, czego doświadczamy wydrukiem w konsoli : "static Beetle.x2 zainicjowana", a do zmiennej x2 przypisywana jest wartość 47.

Gdy już obie klasy są załadowane i zainicjowane zostały pola statyczne pojawia się w końcu jawna linijka z metody main w wydrukiem w konsoli: "Konstruktor klasy Beetle:"

Następnie wywołany zostaje konstruktor klasy Beetle poprzez komendę new Beetle(). Pamiętamy, że pierwszą niejawną komendą w konstruktorze klasy Beetle jest wywołanie super(). A zatem jako pierwsze zostaje wywołany konstruktor klasy Insect.

Zanim konstruktor klasy Insect zostaje wywołany, wszystkie pola (poza tymi statycznymi, które już zostały zainicjalizowane) zostają zainicjalizowane. Te, które nie mają przypisanych wartości, zostają zainicjalizowane wartościami domyślnymi (do int j zostaje przypisana wartość 0). Uruchomiona zostaje w końcu pierwsza linijka konstruktora i w konsoli widizmy kolejny wydruk: "i= 9 j= 0", po czym wartość 39 zostaje przypisana do zmiennej j.

W końcu możemy wrócić do realizowania dalej konstruktora klasy Beetle. Zanim jednak zostaną wywoływane komendy z jego ciała, najpierw zostają zainicjowane pola obiektu Beetle. Do zmiennej int k zostaje przypisana wartość 47 i wywołany zostaje wydruk w konsoli: ""Beetle.k zainicjowana".

Dalej wykonywane są kolejne linijki z ciała konstruktora Beetle, co widzimy na wydruku:"k = 47
j = 39"

Program kończy swoje działanie.

Na tym dość urozmaiconym przykładzie bardzo przejżyście można poznać kolejność inicjalizacji i ładowania klas w jęzku Java. Zdecydowanie polecam przerabiać samemu takie przykłady. W moim przypadku sucha teoria się nie sprawdza. Powyższy przykład z robakami pochodzi z książki **Thinking in Java Bruce'a Eckela**. Pozycja może i wyciągnięta z pawlacza, i ma odstraszjące 1248 stron (i bez obrazków) ale znajdziecie w niej niejeden tak ciekawy przykład.




