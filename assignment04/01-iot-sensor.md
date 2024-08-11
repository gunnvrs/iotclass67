# Ingest and store real-time data from IoT sensors.

## MQTT Topic
ในโปรเจกต์นี้ใช้ MQTT สำหรับการสื่อสารข้อมูลระหว่าง CUCUMBER RS (ESP32) กับ MQTT Broker โดยมี MQTT Topic ชื่อว่า `iot-frames` สำหรับการส่งข้อมูลจากเซ็นเซอร์ไปยัง Broker ดังนี้

```python
client.publish("iot-frames", jsonBuffer);
```

## MQTT Payload
ข้อมูลที่ส่งไปยัง MQTT Broker จะอยู่ในรูปแบบ JSON Payload ซึ่งประกอบไปด้วยข้อมูลของเซ็นเซอร์ต่าง ๆ และ timestamp 
เช่น กรณีของ Sensor 3 จะต้องส่ง payload ให้อยู่ตามรูปแบบ Pattern ดังนี้ 

```json
{
  "id": "43245253",
  "name": "iot_sensor_3",
  "place_id": "42343243",
  "date": "2024-07-15T10:30:00Z",  
  "timestamp": 1626346200000,
  "payload": {
    "temperature": 25.6,
    "humidity": 50.2,
    "pressure": 1013.25,
    "luminosity": 230
  }
}
```

## ESP32

```cpp

#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BMP280.h>
#include <SensirionI2CSht4x.h>
#include <Adafruit_HTS221.h>
#include <Adafruit_MPU6050.h>
#include <PubSubClient.h>
#include <WiFi.h>
#include <ArduinoJson.h>
#include <WiFiUdp.h>
#include <ESPNtpClient.h>

// WiFi credentials
const char* ssid = "TP-Link_CA30";
const char* password = "29451760";

// MQTT Broker settings
const char* mqtt_server = "172.16.46.37";
const int mqtt_port = 1883;
const char* mqtt_user = ""; // Replace with your MQTT username if needed
const char* mqtt_password = ""; // Replace with your MQTT password if needed

const PROGMEM char* ntpServer = "172.16.46.37";

#define NTP_TIMEOUT 5000

// MQTT Client
WiFiClient espClient;
PubSubClient client(espClient);

// Sensors
Adafruit_BMP280 bmp;
SensirionI2CSht4x sht4x;
Adafruit_HTS221 hts;
Adafruit_MPU6050 mpu;

// LED pin
const int ledPin = 2;

// ldr GPIO 5
const int ldrPin = 5;  // GPIO5 for LDR

// MQTT callback function only one sensor
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  
  String message;
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println(message);

  // Parse JSON
  StaticJsonDocument<200> doc;
  DeserializationError error = deserializeJson(doc, message);
  
  if (error) {
    Serial.print("deserializeJson() failed: ");
    Serial.println(error.f_str());
    return;
  }

  const char* sensor_name = doc["name"];

  // Check if the sensor name is "iot_sensor_3"
  if (strcmp(sensor_name, "iot_sensor_3") == 0) {
    Serial.print("----------------------------Filtered message: ");
    Serial.println(message);
  }
}

void setup_wifi() {
  delay(10);
  // Define the desired IP address and subnet mask
  IPAddress ip(172, 16, 46, 31);  // Desired IP address
  IPAddress gateway(172, 16, 46, 1);  // Default gateway IP address
  IPAddress subnet(255, 255, 255, 0);  // Subnet mask
  
  // Connect to Wi-Fi network with specified IP configuration
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.config(ip, gateway, subnet);  // Set static IP configuration

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (client.connect("iot_sensor_3", mqtt_user, mqtt_password)) {
      Serial.println("connected");
      // Subscribe to topic
      client.subscribe("iot-frames");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setupHardware() {
  Wire.begin(41, 40, 100000);
  if (bmp.begin(0x76)) {
    Serial.println("BMP280 sensor ready");
  }

  sht4x.begin(Wire, 0x44);
  Serial.println("SHT4x sensor initialized");

  // if (hts.begin_I2C(0x5F)) {
  //   Serial.println("HTS221 sensor ready");
  // } else {
  //    Serial.println("HTS221 sensor NOT ready!!!");
  // }

  if (mpu.begin(0x68)) {
    Serial.println("MPU6050 sensor ready");
  } 

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, HIGH); 

  pinMode(ldrPin, INPUT);  // prepare LDR

}

void setup() {
  Serial.begin(115200);
  setupHardware();
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
  // timeClient.begin();

  NTP.setTimeZone(TZ_Asia_Bangkok);
  NTP.setInterval(600);
  NTP.setNTPTimeout(NTP_TIMEOUT);
  NTP.begin(ntpServer);


  Serial.println("Starting!!!");
}

unsigned long Get_Epoch_Time(){
  time_t now;
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) {
    return 0;
  }
  time(&now);
  return now;
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  static uint32_t prev_millis = 0;
  const size_t capacity = JSON_OBJECT_SIZE(6) + 200;
  StaticJsonDocument<capacity> doc;

  if (millis() - prev_millis > 15000) {
    prev_millis = millis();

    float p = bmp.readPressure();

    int ldrValue = analogRead(ldrPin); // Read LDR value from GPIO5

    float temperature, humidity;
    int16_t error = sht4x.measureHighPrecision(temperature, humidity);
    if (error) {
      Serial.print("Error trying to execute measureHighPrecision(): ");
      Serial.println(error);
    } else {
  
    unsigned long epochTime = Get_Epoch_Time();

    doc["id"] = "43245253";
    doc["name"] = "iot_sensor_3";
    doc["place_id"] = "42343243";
    // doc["date"] = formattedDate; // Use real-time date
    doc["date"] = NTP.getTimeDateString(time(NULL), "%Y-%m-%dT%H:%M:%S");
    doc["timestamp"] = epochTime; // Convert to milliseconds
    doc["payload"]["temperature"] = temperature;
    doc["payload"]["humidity"] = humidity; // Update with correct humidity reading
    doc["payload"]["pressure"] = p;
    doc["payload"]["luminosity"] = ldrValue;

    // Serialize JSON to char array
    char jsonBuffer[capacity];
    serializeJson(doc, jsonBuffer);

    // Publish JSON to MQTT topic
    client.publish("iot-frames", jsonBuffer);
  }

  delay(2000);
  }
}

```