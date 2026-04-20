# ITP Projekt IoT und ESP-NOW
## Abstandsmesssystem

**Gruppenmitglieder:** Volodymyr Shchyrov, Peter Pavic-Nikolic  
**Datum:** 20.04.2026  

---

## 1. Einführung

Im Bereich des Internets der Dinge (IoT) werden zunehmend Mikrocontroller eingesetzt, um Sensordaten zu erfassen und drahtlos zu übertragen. Eine effiziente und energiearme Kommunikationsmethode stellt dabei ESP-NOW dar, welches eine direkte Kommunikation zwischen ESP-Geräten ohne klassisches WLAN ermöglicht.  

In diesem Projekt wird ein System entwickelt, das Distanzdaten misst und diese drahtlos an ein weiteres Gerät überträgt, wo sie für den Benutzer visualisiert werden.

---

## 2. Projektbeschreibung

Im Rahmen dieses Projekts wurde eine IoT-basierte Distanzmessstation realisiert. Ein ESP32 misst mithilfe eines Ultraschallsensors den Abstand zu einem Objekt und zeigt diesen lokal auf einem OLED-Display sowie über visuelle und akustische Signale an.  

Die gemessenen Daten werden anschließend über ESP-NOW an einen zweiten ESP32 übertragen, welcher die Daten über ein Webinterface zur Verfügung stellt.

---

## 3. Theorie

Das Internet der Dinge (IoT) beschreibt die Vernetzung von Geräten, die über Sensoren verfügen und Daten erfassen, verarbeiten und austauschen können. Mikrocontroller wie der ESP32 spielen dabei eine zentrale Rolle, da sie kostengünstig, energieeffizient und vielseitig einsetzbar sind.  

ESP-NOW ist ein Kommunikationsprotokoll, das von Espressif entwickelt wurde. Es ermöglicht die direkte drahtlose Kommunikation zwischen mehreren ESP-Geräten ohne die Notwendigkeit eines WLAN-Routers. Dadurch werden geringe Latenzzeiten und ein niedriger Energieverbrauch erreicht.  

Zur Abstandsmessung wird ein Ultraschallsensor verwendet. Dieser sendet Schallwellen aus und misst die Zeit, bis das Echo zurückkommt. Anhand dieser Zeit kann die Entfernung zum Objekt berechnet werden.  

Zusätzlich werden Ausgabekomponenten wie ein OLED-Display, eine RGB-LED sowie ein akustischer Signalgeber verwendet, um die gemessenen Daten für den Benutzer verständlich darzustellen.

---

## 4. Arbeitsschritte

////////////////

---

### Code

Der verwendete Code umfasst die Initialisierung der Sensoren, die Berechnung der Distanz sowie die drahtlose Übertragung mittels ESP-NOW.  

///////////////////////////////

---

### Bilder und Schaltungen

Die Schaltung besteht aus einem ESP32, der mit einem Ultraschallsensor, einem OLED-Display, einer RGB-LED sowie einem Buzzer verbunden ist.  

Eine zweite ESP32-Einheit dient als Empfänger der Daten und stellt diese über ein Webinterface dar.  

/////////////////////////////////////////////

---

### Tabellen

| Komponente            | Funktion                                  |
|----------------------|------------------------------------------|
| ESP32 (Sender)       | Erfassung und Übertragung der Daten       |
| ESP32 (Empfänger)    | Empfang und Anzeige im Webinterface       |
| Ultraschallsensor    | Messung der Distanz                      |
| OLED-Display         | Lokale Anzeige der Messwerte             |
| RGB-LED              | Visuelle Darstellung der Distanz         |
| Buzzer               | Akustisches Signal bei geringem Abstand  |

Die Tabelle zeigt die verwendeten Komponenten und deren jeweilige Funktion im System.

---

## 5. Zusammenfassung

Im Rahmen dieses Projekts wurde ein IoT-System zur Distanzmessung entwickelt. Die erfassten Daten konnten erfolgreich lokal angezeigt sowie drahtlos über ESP-NOW übertragen werden.  

Eine Herausforderung stellte insbesondere die Einrichtung der Kommunikation zwischen den beiden ESP32-Geräten dar. Dennoch konnte ein funktionierendes System realisiert werden, welches die Messwerte zuverlässig überträgt und visualisiert.  

Das Projekt zeigt die Möglichkeiten moderner Mikrocontroller im Bereich IoT und drahtloser Kommunikation.

---

## 6. Quellen

[1] Espressif Systems. *ESP-NOW User Guide*.  
[2] Random Nerd Tutorials. *ESP32 ESP-NOW Tutorial*.  
[3] Technische Dokumentation – Wikipedia.  