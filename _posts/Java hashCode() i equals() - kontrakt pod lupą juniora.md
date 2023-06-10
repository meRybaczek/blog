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
