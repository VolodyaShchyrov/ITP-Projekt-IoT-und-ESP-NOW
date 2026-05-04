# ITP Projekt IoT und ESP-NOW
## Abstandsmesssystem

**Gruppenmitglieder:** Volodymyr Shchyrov, Petar Pavic-Nikolic  
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

Zunächst wurden die benötigten Komponenten ausgewählt und vorbereitet. Dazu gehörten zwei ESP32-Mikrocontroller, ein Ultraschallsensor, ein OLED-Display, eine RGB-LED sowie ein Buzzer.  

Anschließend wurde der Ultraschallsensor mit dem ersten ESP32 verbunden, sodass Distanzmessungen durchgeführt werden konnten. Die gemessenen Werte wurden lokal auf dem OLED-Display angezeigt. Zusätzlich wurden visuelle Signale über eine RGB-LED sowie akustische Signale über einen Buzzer ausgegeben, um unterschiedliche Abstände darzustellen.  

Im nächsten Schritt wurde die Kommunikation zwischen zwei ESP32-Geräten mithilfe von ESP-NOW eingerichtet. Der erste ESP32 fungiert dabei als Sender und überträgt die gemessenen Distanzwerte. Der zweite ESP32 empfängt diese Daten.  

Abschließend wurde auf dem zweiten ESP32 ein Webinterface implementiert, über welches die empfangenen Daten im Browser angezeigt werden können.


---

### Code

Der verwendete Code umfasst die Initialisierung der Sensoren, die Berechnung der Distanz sowie die drahtlose Übertragung mittels ESP-NOW.  

#### ESP32 Sender code

```c++
#include <WiFi.h>
#include <esp_now.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// ---------- pins ----------
#define TRIG_PIN 5
#define ECHO_PIN 18
#define LED_R 25
#define LED_G 26
#define LED_B 27
#define SDA_PIN 17
#define SCL_PIN 16
#define SOUND 33 

// ---------- OLED ----------
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// ---------- ESP-NOW ----------
typedef struct {
  float distance;
} DataPacket;

DataPacket data;
bool espNowConnected = false;

// MAC resiever
uint8_t receiverMAC[] = {0x00, 0x70, 0x07, 0x19, 0x89, 0x58};

// ---------- CALLBACK ----------
void onDataSent(const wifi_tx_info_t *info, esp_now_send_status_t status) {
  espNowConnected = (status == ESP_NOW_SEND_SUCCESS);
}

// ---------- sensor ----------
float readDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  float distance = duration * 0.034 / 2.0;

  return (distance <= 0 || distance > 400) ? 0 : distance;
}

// ---------- RGB ----------
void setColor(int r, int g, int b) {
  analogWrite(LED_R, r);
  analogWrite(LED_G, g);
  analogWrite(LED_B, b);
}

void updateRGB(float d) {
  if (d > 40)      setColor(0, 255, 0);
  else if (d > 20) setColor(255, 120, 0);
  else if (d > 0)  setColor(255, 0, 0);
  else             setColor(0, 0, 50);
}

// ---------- sound ----------
void updateSound(float d) {
  digitalWrite(SOUND, (d < 20 && d > 0));
}

// ---------- OLED ----------
void updateOLED(float d) {
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);

  display.setTextSize(1);
  display.setCursor(0, 0);
  display.print("ESP-NOW: ");
  display.println(espNowConnected ? "OK" : "ERROR");

  display.setTextSize(2);
  display.setCursor(0, 18);
  display.print("Distance:");

  display.setTextSize(3);
  display.setCursor(0, 40);
  if (d > 0) {
    display.print(d, 1);
    display.print("cm");
  } else {
    display.print("----");
  }

  display.display();
}

// ---------- send ----------
void sendData(float d) {
  data.distance = d;
  esp_now_send(receiverMAC, (uint8_t *)&data, sizeof(data));
}

// ---------- SETUP ----------
void setup() {
  Serial.begin(115200);

  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(LED_R, OUTPUT);
  pinMode(LED_G, OUTPUT);
  pinMode(LED_B, OUTPUT);
  pinMode(SOUND, OUTPUT);

  Wire.begin(SDA_PIN, SCL_PIN);

  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);

  WiFi.mode(WIFI_STA);

  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW init error");
    return;
  }

  esp_now_register_send_cb(onDataSent);

  
  esp_now_peer_info_t peerInfo = {};
  memcpy(peerInfo.peer_addr, receiverMAC, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;

  if (esp_now_add_peer(&peerInfo) != ESP_OK) {
    Serial.println("Peer add error");
    return;
  }

  Serial.println("Transmitter ready");
}


void loop() {
  float dist = readDistance();

  Serial.printf("Distance: %.1f cm\n", dist);

  updateRGB(dist);
  updateSound(dist);
  updateOLED(dist);
  sendData(dist);

  delay(500);
}

```
#### ESP32 Empfänger

```c++
#include <WiFi.h>
#include <WebServer.h>
#include <esp_now.h>

const char* ssid = "vova_esp";
const char* password = "123456qwerty";

WebServer server(80);

typedef struct {
  float distance;
} DataPacket;

DataPacket receivedData;

float lastMinute[12];
int indexMinute = 0;

bool espNowOnline = false;
unsigned long lastPacketTime = 0;

// ---------- HTML ----------
String getHTML() {
  return R"rawliteral(
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
<h2>ESP Distanz</h2>
<p>Status: <span id="status">OFFLINE</span></p>
<p>Distanz: <span id="dist">0</span> cm</p>
<canvas id="chart"></canvas>

<script>
let chart;

function createChart(){
 const ctx=document.getElementById('chart').getContext('2d');
 chart=new Chart(ctx,{
  type:'line',
  data:{labels:[0,5,10,15,20,25,30,35,40,45,50,55],
  datasets:[{data:Array(12).fill(0)}]}
 });
}

async function load(){
 try{
  const r=await fetch('/data');
  const j=await r.json();

  chart.data.datasets[0].data=j.values;
  chart.update();

  document.getElementById('dist').innerText =
    j.values[j.values.length-1];

  document.getElementById('status').innerText =
    j.online ? "ONLINE" : "OFFLINE";
 }catch{}
}

createChart();
setInterval(load,2000);
</script>
</body>
</html>
)rawliteral";
}

// ---------- JSON ----------
void handleData() {
  String json = "{\"values\":[";
  for (int i = 0; i < 12; i++) {
    json += String(lastMinute[i]);
    if (i < 11) json += ",";
  }
  json += "],\"online\":";
  json += espNowOnline ? "true" : "false";
  json += "}";

  server.send(200, "application/json", json);
}

// ---------- ESP-NOW ----------
void onDataRecv(const esp_now_recv_info_t *info,
                const uint8_t *incomingData,
                int len) {

  memcpy(&receivedData, incomingData, sizeof(receivedData));

  lastMinute[indexMinute] = receivedData.distance;
  indexMinute = (indexMinute + 1) % 12;

  espNowOnline = true;
  lastPacketTime = millis();

  Serial.print("Received: ");
  Serial.println(receivedData.distance);
}

void setup() {
  Serial.begin(115200);

  
  WiFi.mode(WIFI_AP_STA);
  WiFi.softAP(ssid, password, 1); 

  Serial.print("IP: ");
  Serial.println(WiFi.softAPIP());

  for (int i = 0; i < 12; i++) lastMinute[i] = 0;

  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW error");
    return;
  }

  esp_now_register_recv_cb(onDataRecv);

  server.on("/", [](){ server.send(200, "text/html", getHTML()); });
  server.on("/data", handleData);
  server.begin();
}

void loop() {
  server.handleClient();

  // OFFLINE
  if (millis() - lastPacketTime > 10000)
    espNowOnline = false;
}

```
---

### Bilder und Schaltungen

Die Schaltung besteht aus einem ESP32, der mit einem Ultraschallsensor, einem OLED-Display, einer RGB-LED sowie einem Buzzer verbunden ist.  

Eine zweite ESP32-Einheit dient als Empfänger der Daten und stellt diese über ein Webinterface dar.  

![Schaltung](img/Screenshot%202026-05-04%20103905.png)

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
