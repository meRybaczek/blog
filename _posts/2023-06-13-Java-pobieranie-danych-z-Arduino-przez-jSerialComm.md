---
published: true
---
![]({{site.baseurl}}/images/dht11.jpg)

Projekt mojego mikroserwisu pogodowego posiada w kontrolerze endpoint zwracający między innymi bieżącą temperaturę na zewnątrz. Aktualnie temperatura ta jest zwracana z wykorzystaniem zewnętrznego API openweather.com,z którym łączy się serwis. Po zmapowaniu odpowiedzi z openweather na encje, odpowiedź wraca do klienta. Postanowiłem pójść krok dalej i jako alternatywne źródło danych wykorzystać własny czujnik temperatury. A raczej od teraz to czujnik będzie głownym źródłem a openweather backupem. 

Wykorzystałem do tego platformę Arduino Uno oraz czujnik temperatury (i wilgotności w jednym) DHT11. Na załączonym zdjęciu to ten niebieski element. Poza nim znajduje się jeszcze fotorezytor do pomiarów natężenia oświetlenia - ale to temat na osobne zajęcia.

Serwis zbudowany jest w Javie, z wykorzystaniem Spring Boot. Jako zależność dodałem bibliotekę  jSerialComm, dzięki której dane z Arduino trafią wprost do serwisu. 
{% highlight java %}
		<dependency>
			<groupId>com.fazecast</groupId>
			<artifactId>jSerialComm</artifactId>
			<version>2.9.3</version>
		</dependency>
{% endhighlight %}
jSerialComm jest biblioteką Javy, która umożliwia komunikację z urządzeniami szeregowymi (RS-232/UART) za pomocą interfejsu szeregowego. Biblioteka jSerialComm dostarcza prosty interfejs API, który umożliwia otwieranie portów szeregowych, przesyłanie danych oraz odbieranie danych z urządzenia. Ja skorzystam jedynie z odbierania. 
Klasa serwisu obsługująca pobieranie danych wygląda tak:

{% highlight java %}

import com.fazecast.jSerialComm.SerialPort;
import org.springframework.stereotype.Service;

import java.io.InputStream;

@Service
public class ArduinoDataReceiver {
    private static final int PORT_NO = 0;
    private static final int RECEIVING_PACKET_SEQUENCE_MILLIS = 2000;
    private final StringBuilder measurment;
    private final SerialPort comPort = SerialPort.getCommPorts()[PORT_NO];


    public ArduinoDataReceiver(StringBuilder measurments) {
        this.measurment = measurments;
    }

    public StringBuilder getMeasurment() {
        openPort();
        InputStream in = comPort.getInputStream();

        try {
            int bytesPerPacket = getBytesPerPacket();
            for (int j = 0; j < bytesPerPacket; ++j) {
                measurment.append((char) in.read());
            }
            in.close();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            comPort.closePort();
        }

        return measurment;
    }

    private void openPort() {
        comPort.openPort();
        comPort.setComPortTimeouts(SerialPort.TIMEOUT_READ_SEMI_BLOCKING, 0, 0);
        comPort.flushIOBuffers();
    }

    private int getBytesPerPacket() throws InterruptedException {
        Thread.sleep(RECEIVING_PACKET_SEQUENCE_MILLIS);
        return comPort.bytesAvailable();
    }

}

{% endhighlight %}


Czujnink DHT11 pozwala na pomiary w odstępie 2s, dlatego też mikrokontroler na płytce Arduino został tak zapropogramowany aby wysyłał pakiet danych w interwale 2s. Idąć w ślad za tym, sekwencja odczytu danych z portu również uwzględnia ten czas. 
Statyczna metoda SerialPort.getCommPorts() zwraca talicę dostępnych portów. W moim przypadku mam tylko jeden, więc znajduje się on pod indeksem [0], i przypisuję go do zmiennej SerialPort comPort.

Jeżeli u siebie masz więcej portów, można przeiterować tablicę portów, wyciągając z niej np nazwy portów, metodą getSystemPortName(), i przypisać odpowiedni port do zmiennej comPort.

Kolejnym krokiem jest otwarcie portu comPort.openPort() i ustawienie timeout-ów(opcjonalnie). Pierwszy pakiet danych w moim przypadku zazwyczaj był nieprawidłowy. Spowodowane to było niezsynchonizowaniem uruchomienia strumienia wejścia (comPort.getInputStream()) z otrzymanym już pakietem danych. Bufor zawierał wówczas niepełne dane, co generowało inicjacyjny błąd. Pomogła metoda flushIOBuffers(), która opróżniła bufor. 

Następnie prywatna metoda getBytesPerPacket() sprawdza ile bajtów zawiera odbierany pakiet danych, po to aby określić długość pętli odczytującej bajt po bajcie. Zanim metoda zostaje uruchomiona czekam 2s aby pakiet zdążył się wysłać, inaczej metoda ta zwróciłaby 0.

Na otwartym strumieniu w pętli odczytywany jest bajt metodą in.read() i rzutowany do char, który łańcuchowo dopisywany jest do obiektu StringBuilder measurment.
Zamykamy port, zamykamy strumień wejścia i tyle. Stringbuilder zawiera ciąg znaków, który następnie możemy odpowiednio zmapować.
