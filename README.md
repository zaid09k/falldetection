# # 🚀 Fall Detection & GPS Alert System using ESP32

## 📌 Project Overview

This project is a **Fall Detection System** built using **ESP32**, **MPU6050 (accelerometer + gyroscope)**, and **NEO-6M GPS module**.

It detects sudden falls using acceleration data and:

* Triggers a **buzzer alert**
* Displays **location (real or demo)** on Serial Monitor

---

## 🎯 Features

* 📉 Fall detection using MPU6050
* 📍 Real-time GPS location (if available)
* 📍 Demo location (if GPS not fixed)
* 🔊 Buzzer alert (5 seconds)
* 💻 Serial Monitor output (no Bluetooth needed)
* 🔋 Battery compatible (portable)

---

## 🧰 Components Used

* ESP32 WROOM
* MPU6050 Sensor
* NEO-6M GPS Module
* Buzzer
* 3.7V Battery (optional)

---

## 🔌 Wiring Diagram

### 🧭 MPU6050 → ESP32

| MPU6050 | ESP32  |
| ------- | ------ |
| VCC     | 3.3V   |
| GND     | GND    |
| SDA     | GPIO21 |
| SCL     | GPIO22 |

---

### 📍 GPS → ESP32

| GPS | ESP32    |
| --- | -------- |
| VCC | VIN (5V) |
| GND | GND      |
| TX  | GPIO16   |
| RX  | GPIO17   |

---

### 🔊 Buzzer → ESP32

| Buzzer | ESP32  |
| ------ | ------ |
| +      | GPIO25 |
| -      | GND    |

---

## ⚠️ Important Notes

* ❌ Do NOT use GPIO12 (causes boot issues)
* ✔ Upload code without connecting sensors
* ✔ Connect hardware after upload
* ✔ GPS works best outdoors

---

## 💻 Source Code

```cpp
#include <Wire.h>
#include <MPU6050_light.h>
#include <TinyGPS++.h>

// -------- MPU --------
MPU6050 mpu(Wire);

// -------- GPS --------
TinyGPSPlus gps;
HardwareSerial gpsSerial(1);

// -------- BUZZER --------
#define BUZZER 25

bool fallDetected = false;
unsigned long lastFallTime = 0;

// Demo location
float demoLat = 19.0760;
float demoLng = 72.8777;

void setup() {
  Serial.begin(115200);
  delay(1000);

  Serial.println("STARTING...");

  Wire.begin(21,22);
  byte status = mpu.begin();

  if (status != 0) {
    Serial.println("MPU FAIL ❌");
  } else {
    Serial.println("MPU OK ✅");
    delay(1000);
    mpu.calcOffsets();
  }

  gpsSerial.begin(9600, SERIAL_8N1, 16, 17);

  pinMode(BUZZER, OUTPUT);
  digitalWrite(BUZZER, LOW);

  Serial.println("SYSTEM READY 🚀");
}

void loop() {

  while (gpsSerial.available()) {
    gps.encode(gpsSerial.read());
  }

  mpu.update();

  float acc = sqrt(
    pow(mpu.getAccX(),2) +
    pow(mpu.getAccY(),2) +
    pow(mpu.getAccZ(),2)
  );

  Serial.print("ACC: ");
  Serial.println(acc);

  if (acc > 2.2 && !fallDetected) {

    Serial.println("Impact detected ⚠️");
    delay(300);

    mpu.update();

    float acc2 = sqrt(
      pow(mpu.getAccX(),2) +
      pow(mpu.getAccY(),2) +
      pow(mpu.getAccZ(),2)
    );

    if (acc2 < 1.2 && millis() - lastFallTime > 5000) {

      Serial.println("FALL DETECTED 🚨");

      fallDetected = true;
      lastFallTime = millis();

      digitalWrite(BUZZER, HIGH);
      delay(5000);
      digitalWrite(BUZZER, LOW);

      float lat, lng;

      if (gps.location.isValid()) {
        Serial.println("REAL GPS ✅");
        lat = gps.location.lat();
        lng = gps.location.lng();
      } else {
        Serial.println("DEMO LOCATION ⚠️");
        lat = demoLat;
        lng = demoLng;
      }

      Serial.print("Lat: ");
      Serial.println(lat, 6);

      Serial.print("Lng: ");
      Serial.println(lng, 6);

      Serial.print("Maps: https://maps.google.com/?q=");
      Serial.print(lat, 6);
      Serial.print(",");
      Serial.println(lng, 6);
    }
  }

  if (millis() - lastFallTime > 6000) {
    fallDetected = false;
  }

  delay(300);
}
```

---

## 🧪 Output Example

### Normal:

```
ACC: 0.98
```

### Fall Detected:

```
Impact detected
FALL DETECTED
DEMO LOCATION
Lat: 19.076000
Lng: 72.877700
```

---

## 🔋 Future Improvements

* 📲 Mobile App integration
* 📩 SMS alert system
* ⌚ Wearable design
* ☁️ Cloud data logging

---

## 👨‍💻 Author
PROF.MANILA GUPTA
ZAID KHAN 
SHAHID KHAN
AHMED SHAIKH 
SHAIHD CHOUDHARY


---

## ⭐ Conclusion

This project provides a **real-time safety solution** using IoT, combining **motion sensing and location tracking** for emergency situations.
