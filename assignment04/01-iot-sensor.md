# Ingest and store real-time data from IoT sensors.

## MQTT Topic
ในโปรเจกต์นี้ใช้ MQTT สำหรับการสื่อสารข้อมูลกันระหว่าง CUCUMBER RS (ESP32) กับ MQTT Broker โดยมี MQTT Topic ชื่อว่า iot-frames สำหรับการส่งข้อมูลจากเซ็นเซอร์ไปยัง Broker ดังนี้

client.publish("iot-frames", jsonBuffer);


## MQTT Payload
ข้อมูลที่ส่งไปยัง MQTT Broker จะอยู่ในรูปแบบ JSON Payload ซึ่งประกอบไปด้วยข้อมูลของเซ็นเซอร์ต่าง ๆ และ timestamp 
เช่น กรณีของ Sensor 3 จะต้องส่ง payload ให้อยู่ตามรูปแบบ Pattern ดังนี้ 

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



## ESP32

```cpp

```