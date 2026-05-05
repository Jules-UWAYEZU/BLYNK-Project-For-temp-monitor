# BLYNK-Project-For-temp-monitor
sample
// ======== BLYNK CONFIG (REQUIRED) ========
#define BLYNK_TEMPLATE_ID "TMPL2S-xjL4c1"
#define BLYNK_TEMPLATE_NAME "TEMP MONITOR"
#define BLYNK_AUTH_TOKEN "m7yX8ao8G2hoch5xS9rOTLwKHo3rBkaI"
#define BLYNK_PRINT Serial

// ======== LIBRARIES ========
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>

// ======== WIFI ========
char ssid[] = "Paradox";
char pass[] = "paradoxx";

// ======== GPIO ========
#define DHTPIN 4      // GPIO4 (D2)
#define LED_PIN 5     // GPIO5 (D1)
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);
BlynkTimer timer;

// ======== FLAGS ========
bool wifiStarted = false;
bool wifiConnectedPrinted = false;

// ======== LED CONTROL ========
BLYNK_WRITE(V1)
{
  int ledState = param.asInt();
  digitalWrite(LED_PIN, ledState);

  Serial.print("LED: ");
  Serial.println(ledState ? "ON" : "OFF");
}

// ======== TEMPERATURE ========
void sendTemperature()
{
  if (!Blynk.connected()) return;

  float temp = dht.readTemperature();

  if (isnan(temp)) {
    Serial.println("❌ DHT read failed");
    return;
  }

  Blynk.virtualWrite(V0, temp);

  Serial.print("🌡 Temp: ");
  Serial.print(temp);
  Serial.println(" °C");
}

// ======== CONNECTION HANDLER ========
void connectToBlynk()
{
  if (!wifiStarted) {
    Serial.println("📡 Starting WiFi...");
    WiFi.begin(ssid, pass);
    wifiStarted = true;
    return;
  }

  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("⏳ Waiting WiFi...");
    return;
  }

  if (!wifiConnectedPrinted) {
    Serial.println("✅ WiFi Connected");
    Serial.print("IP: ");
    Serial.println(WiFi.localIP());
    wifiConnectedPrinted = true;
  }

  if (!Blynk.connected()) {
    Serial.println("🔄 Connecting to Blynk...");
    if (Blynk.connect(1000)) {
      Serial.println("✅ Blynk Connected");
    } else {
      Serial.println("❌ Blynk Failed");
    }
  }
}

// ======== SETUP ========
void setup()
{
  Serial.begin(115200);
  delay(100);

  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);

  dht.begin();

  timer.setInterval(3000L, connectToBlynk);
  timer.setInterval(2000L, sendTemperature);

  Serial.println("✅ System initialized");
}

// ======== LOOP ========
void loop()
{
  if (Blynk.connected()) {
    Blynk.run();
  }

  timer.run();
}
