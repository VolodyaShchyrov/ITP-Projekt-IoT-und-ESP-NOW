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

// --- Configuration ---
const char* ssid = "vova_esp";
const char* password = "123456qwerty";

WebServer server(80);

typedef struct {
  float distance;
} DataPacket;

DataPacket receivedData;

// --- Global Variables ---
float distanceHistory[12]; // Stores 12 readings (5 sec each = 1 min total)
int historyIndex = 0;
unsigned long lastHistoryUpdate = 0;
float latestDistance = 0;

bool isEspNowOnline = false;
unsigned long lastIncomingPacketTime = 0;

// ---------- HTML UI (English) ----------
String getHTML() {
  return R"rawliteral(
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>ESP32 Distance Monitor</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; text-align: center; background: #eceff1; color: #333; }
        .card { max-width: 700px; margin: 40px auto; background: white; padding: 30px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        .status-box { padding: 5px 10px; border-radius: 5px; font-weight: bold; }
        .online { background: #e8f5e9; color: #2e7d32; }
        .offline { background: #ffebee; color: #c62828; }
        .value-display { font-size: 2em; color: #0288d1; margin: 10px 0; }
    </style>
</head>
<body>
    <div class="card">
        <h2>Ultrasonic Sensor Monitor</h2>
        <div id="statusIndicator" class="status-box offline">STATUS: OFFLINE</div>
        <div class="value-display"><span id="dist">0.0</span> cm</div>
        <canvas id="distanceChart"></canvas>
    </div>

<script>
let chart;
const timeLabels = ['55s','50s','45s','40s','35s','30s','25s','20s','15s','10s','5s','Now'];

function initChart(){
    const ctx = document.getElementById('distanceChart').getContext('2d');
    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: timeLabels,
            datasets: [{
                label: 'Distance (cm)',
                data: Array(12).fill(0),
                borderColor: '#0288d1',
                backgroundColor: 'rgba(2, 136, 209, 0.1)',
                fill: true,
                tension: 0.4
            }]
        },
        options: {
            scales: { y: { beginAtZero: true } },
            animation: { duration: 800 }
        }
    });
}

async function fetchData(){
    try {
        const response = await fetch('/data');
        const data = await response.json();
        
        // Update Chart
        chart.data.datasets[0].data = data.values;
        chart.update();

        // Update Text & Status
        const currentVal = data.values[data.values.length - 1];
        document.getElementById('dist').innerText = currentVal.toFixed(1);
        
        const statusDiv = document.getElementById('statusIndicator');
        if(data.online) {
            statusDiv.innerText = "STATUS: ONLINE";
            statusDiv.className = "status-box online";
        } else {
            statusDiv.innerText = "STATUS: OFFLINE";
            statusDiv.className = "status-box offline";
        }
    } catch(err) { console.error("Fetch error:", err); }
}

initChart();
setInterval(fetchData, 5000); // Request data every 5 seconds
fetchData(); 
</script>
</body>
</html>
)rawliteral";
}

// ---------- JSON Endpoint ----------
void handleDataRequest() {
  String json = "{\"values\":[";
  // Reorder history so the newest data is at the end of the array
  for (int i = 0; i < 12; i++) {
    int idx = (historyIndex + i) % 12;
    json += String(distanceHistory[idx]);
    if (i < 11) json += ",";
  }
  json += "],\"online\":";
  json += isEspNowOnline ? "true" : "false";
  json += "}";

  server.send(200, "application/json", json);
}

// ---------- ESP-NOW Callback ----------
void onDataReceive(const esp_now_recv_info_t *info, const uint8_t *incomingData, int len) {
  memcpy(&receivedData, incomingData, sizeof(receivedData));
  
  latestDistance = receivedData.distance;
  isEspNowOnline = true;
  lastIncomingPacketTime = millis();
  
  // Print incoming data details to Serial
  Serial.print("[ESP-NOW] New Data from: ");
  for (int i = 0; i < 6; i++) {
    Serial.printf("%02X%s", info->src_addr[i], (i < 5) ? ":" : "");
  }
  Serial.printf(" | Distance: %.2f cm\n", latestDistance);
}

void setup() {
  Serial.begin(115200);
  delay(1000);

  Serial.println("\n--- ESP32 Setup Started ---");

  // Initialize WiFi in Access Point + Station mode
  WiFi.mode(WIFI_AP_STA);
  WiFi.softAP(ssid, password, 1); 

  // Print AP Information
  Serial.print("[WiFi] Access Point Created. SSID: ");
  Serial.println(ssid);
  Serial.print("[WiFi] IP Address: ");
  Serial.println(WiFi.softAPIP());

  // Initialize ESP-NOW
  if (esp_now_init() != ESP_OK) {
    Serial.println("[Error] ESP-NOW Initialization Failed");
    return;
  }
  Serial.println("[ESP-NOW] Initialized Successfully");

  // Register Receive Callback
  esp_now_register_recv_cb(onDataReceive);

  // Configure Web Server Routes
  server.on("/", [](){ server.send(200, "text/html", getHTML()); });
  server.on("/data", handleDataRequest);
  server.begin();
  
  Serial.println("[Server] HTTP Server Started");

  // Clear history buffer
  for (int i = 0; i < 12; i++) distanceHistory[i] = 0;
  
  Serial.println("--- Setup Complete ---\n");
}

void loop() {
  server.handleClient();

  // Update history buffer every 5 seconds
  if (millis() - lastHistoryUpdate >= 5000) {
    lastHistoryUpdate = millis();
    distanceHistory[historyIndex] = latestDistance;
    historyIndex = (historyIndex + 1) % 12;

    // Print periodic status report to Serial
    Serial.print("[System Report] Status: ");
    Serial.print(isEspNowOnline ? "ONLINE" : "OFFLINE");
    Serial.print(" | Last Recorded Value: ");
    Serial.print(latestDistance);
    Serial.println(" cm");
  }

  // Check if peer is offline (no data for 10 seconds)
  if (millis() - lastIncomingPacketTime > 10000) {
    if (isEspNowOnline) {
      isEspNowOnline = false;
      Serial.println("[Warning] Connection Lost (Timeout)");
    }
  }
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


[[1] ESP32 - OLED](https://esp32io.com/tutorials/esp32-oled)

[[2] Random Nerd Tutorials. *ESP32 ESP-NOW Tutorial*.](https://randomnerdtutorials.com/esp-now-esp32-arduino-ide/) 

[[3] „How To Create a Color Picker“](https://www.w3schools.com/howto/howto_html_colorpicker.asp)
