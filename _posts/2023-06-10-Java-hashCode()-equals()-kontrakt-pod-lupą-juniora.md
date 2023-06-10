---
published: true
---
## Java hashCode() i equals() - kontrakt pod lupą juniora
![]({{site.baseurl}}/images/mala-czerwona-zabawka-samochodowa-dla-dziecka-bawiacego-sie-z-nia-dziecka.jpg)
Gdy moje dziecięce resoraki miały przyciemniane szyby, przez które nie można było zajrzeć do środka, ciekawość skutecznie namawiała mnie do demontażu takiej zabawki. Teraz, ucząc się Javy, jest podobnie. Nieraz mam potrzebę przeanalizowania zagadnienia bardziej szczegółowo niż dostarcza mi to zdawkowy tutorial czy przerabiany temat kursu. Klikam wtedy lawinowo w kolejne klasy bazowe i interfejsy, szukając źródła implementacji i testując kod na różne sposoby. Tym razem pod lupę trafił temat kontraktu pomiędzy metodą equals() i hashCode(). 

## Co wiemy z podstaw
To, że metoda equals() powinna być zaimplementowana w klasie naszego obiektu, aby móc poprawnie porównywać ze sobą dwie instancje, jest oczywiste. To, że powinniśmy zaimplementować w naszej klasie obiektu metodę equals() wraz z metodą hashCode(), aby móc poprawnie operować na zbiorach lub mapach tych obiektów, też wiemy. Co się za tym kryje i dlaczego nie wystarczy do porównywania jedynie sama metoda equals() i po co angażować do tego hashCode(), lub odwrotnie? Odpowiedź związana jest z optymalizacją procesu porównywania obiektów.

## Nauka przez doświadczenie
Proponuje wykonać szybki test, który pozwoli unaocznić zasadę działania kontraktu equals() - hashCode().

Moje obiekty testowe będą instancjami klasy Person:
{% highlight java %}
public class Person {
    private final String firstName;
    private final String lastName;
    private final Integer age;

    @Override
    public boolean equals(Object o) {
        System.out.println("equals method initialized for " + firstName);
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return Objects.equals(firstName, person.firstName) &&
                Objects.equals(lastName, person.lastName) && Objects.equals(age, person.age);
    }

    @Override
    public int hashCode() {
        System.out.println("hashcode method initialized for " + firstName);
        return Objects.hash(firstName, lastName, age);
    }
   {% endhighlight %}
 
W metodzie equals() oraz hashCode() dodałem linijkę z System.out-em aby doświadczyć w konsoli kolejność działań. Możnaby było skorzystać z debuggera, ale w tym przypadku dosłownie czarno na białym zobaczymy obie metody w akcji.

Teraz czas na eksperyment. W klasie PersonApp dodajemy do zbioru nowe obiekty klasy Person, dwie unikatowe i trzecia taka sama jak pierwsza:
{% highlight java %}
public class PersonApp {
    public static void main(String[] args) {

        Set<Person> persons = new HashSet<>();

        Person lebowski1 = new Person("Jeffrey", "Lebowski", 45);
        Person sobczak = new Person("Walter", "Sobchack", 45);
        Person lebowski2 = new Person("Jeffrey", "Lebowski", 45);

        System.out.println("Dodano lebowski1: " + persons.add(lebowski1) + "\n");
        System.out.println("Dodano sobczak: " + persons.add(sobczak) + "\n");
        System.out.println("Dodano lebowski2: " + persons.add(lebowski2) + "\n");

        System.out.println("Ilosc dodanych osob: " + persons.size());
    }
}
{% endhighlight %}



