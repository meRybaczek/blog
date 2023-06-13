---
published: false
---
## Jak odczytać w Javie dane wysyłane przez mikrokontroler platformy Arduino.

Projekt mojego mikroserwisu pogodowego posiada w kontrolerze endpoint zwracający między innymi bieżącą temperaturę na zewnątrz. Aktualnie temperatura ta jest pobierana z innego serwisu pogodowego (openweather.com -polecam, bardzo dobrze udokumentowane API)z którym łączy się mój serwis. Po zmapowaniu odpowiedzi z openweather na moją encje, odpowiedź wraca do klienta. Postanowiłem pójść krok dalej i jako alternatywne źródło danych wykorzystać własny czujnik temperatury. A raczej to czujnik będzie głownym źródłem a openweather backupem. 

Wykorzystałem do tego platformę Arduino Uno oraz czujnik temperatury (i wilgotności w jednym) DHT11. Na załączonym zdjęciu to ten niebieski elemennt. Poza nim znajduje się jeszcze fotorezytor do pomiarów natężenia oświetlenia - ale to temat na osobne zajęcia.

Serwis zbudowany jest w Javie, z wykorzystaniem Spring Boot. Jako zależność dodałem bibliotekę  jSerialComm, dzięki której dane z Arduino trafią wprost do serwisu. jSerialComm jest biblioteką Javy, która umożliwia komunikację z urządzeniami szeregowymi (RS-232/UART) za pomocą interfejsu szeregowego. Biblioteka jSerialComm dostarcza prosty interfejs API, który umożliwia otwieranie portów szeregowych, przesyłanie danych oraz odbieranie danych z urządzenia. Ja skorzysta jedynie z odbierania:


