---
published: false
---
## Jak odczytać w Javie dane wysyłane przez mikrokontroler platformy Arduino.

Projekt mojego mikroserwisu pogodowego posiada w kontrolerze endpoint zwracający między innymi bieżącą temperaturę na zewnątrz. Aktualnie temperatura ta jest pobierana z innego serwisu pogodowego (openweather.com -polecam, bardzo dobrze udokumentowane API)z którym łączy się mój serwis. Po zmapowaniu odpowiedzi z openweather na moją encje, odpowiedź wraca do klienta. Postanowiłem pójść krok dalej i jako alternatywne źródło danych wykorzystać własny czujnik temperatury. A raczej to czujnik będzie głownym źródłem a openweather backupem. 

Wykorzystałem do tego platformę Arduino Uno oraz czujnik temperatury (i wilgotności w jednym) DHT11. Na załączonym zdjęciu to ten niebieski elemennt. Poza nim znajduje się jeszcze fotorezytor do pomiarów natężenia oświetlenia - ale to temat na osobne zajęcia.

Serwis zbudowany jest w Javie, z wykorzystaniem Spring Boot. Jako zależność dodałem bibliotekę  jSerialComm, dzięki której dane z Arduino trafią wprost do serwisu. jSerialComm jest biblioteką Javy, która umożliwia komunikację z urządzeniami szeregowymi (RS-232/UART) za pomocą interfejsu szeregowego. Biblioteka jSerialComm dostarcza prosty interfejs API, który umożliwia otwieranie portów szeregowych, przesyłanie danych oraz odbieranie danych z urządzenia. Ja skorzysta jedynie z odbierania. Klasa serwisu obsługująca pobieranie danych wygląda tak:

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


napisac teraz tutaj jakie jsekwesncji wysyła dane czujnik,potem dlaczego i po co ten czas uwzgledniam, 



