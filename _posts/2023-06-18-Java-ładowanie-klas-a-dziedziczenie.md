---
published: false
---

W językach takich jak C czy C++ programy są ładowane w całości podczas startu. W języku Java proces ten wygląda nieco inaczej. Skompilowane już klasy wczytywane są do pamięci dopiero gdy są potrzebne. A są potrzebne gdy zostaje wywołany konstruktor klasy lub pojawia się odniesienie do jego ststycznego pola lub statycznej metody. To właśnie wtedy zostają zainicjalizowane statyczne pola i statyczne bloki w porządku tekstowym.

Przeanalizujmy proces inicjalizacji na przykładzie klasy bazowej Insect.class i klasy dziedziczącej Beetle.class

Insect.class

