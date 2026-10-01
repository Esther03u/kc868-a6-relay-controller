# ESP32 + ESPHome + Firebase Realtime Database + GitHub Pages Dashboard

คู่มือสรุปการทำระบบ IoT ส่งข้อมูลจากเซ็นเซอร์จำลอง (อุณหภูมิ, ความชื้น, แสง) ด้วย ESP32 ไปยัง Firebase Realtime Database และแสดงผลแบบ Realtime บน GitHub Pages Web Dashboard

---

## 🏗️ โครงสร้างการทำงานของระบบ (System Architecture)

```
+-------------------------------------------------------+
|                    ESP32 (ESPHome)                    |
|                                                       |
|  - เซ็นเซอร์จำลอง (Template Sensors):                 |
|    • temp:  25.0 - 35.0 °C  (สุ่มทุก 5 วินาที)         |
|    • humi:  50.0 - 80.0 %   (สุ่มทุก 5 วินาที)         |
|    • light: 100 - 1000 lx   (สุ่มทุก 5 วินาที)         |
|  - เวลามาตรฐาน: SNTP (Asia/Bangkok) -> Unix timestamp  |
+---------------------------+---------------------------+
                            |
                            | Wi-Fi + HTTPS REST API (ทุก 10 วินาที)
                            v
+-------------------------------------------------------+
|              Firebase Realtime Database               |
|                                                       |
|  /lab/esp32                                           |
|    ├── latest   (HTTP PUT: อัปเดตทับค่าปัจจุบัน)         |
|    │     ├── temp: 29.07                              |
|    │     ├── humi: 55.90                              |
|    │     ├── light: 628.1                             |
|    │     └── timestamp: 1789883588                    |
|    └── history  (HTTP POST: เพิ่มบันทึกประวัติย้อนหลัง)    |
|          ├── -P1xL4...                                |
|          ├── -P1xL6...                                |
|          └── ...                                      |
+---------------------------+---------------------------+
                            |
                            | Realtime Fetch / REST API
                            v
+-------------------------------------------------------+
|             GitHub Pages (Web Dashboard)              |
|                                                       |
|  ไฟล์: index.html                                     |
|  URL: https://<username>.github.io/<repo>/            |
|  - การ์ดแสดงค่า Realtime: อุณหภูมิ, ความชื้น, แสง     |
|  - คำนวณค่าผลต่าง (Delta ▲/▼), ค่าต่ำสุด, ค่าสูงสุด   |
|  - กราฟเส้นแบบ Real-time Smooth Spline                |
|  - ปุ่มตั้งค่า URL Firebase และปุ่มโหมดจำลอง Demo    |
+-------------------------------------------------------+
```

---

## 🚀 ขั้นตอนการติดตั้งและใช้งานทีละขั้นตอน

### ขั้นตอนที่ 1: ตั้งค่า Firebase Realtime Database
1. เข้าไปที่ [Firebase Console](https://console.firebase.google.com/)
2. สร้างโปรเจกต์ใหม่ (เช่น `iot-project`)
3. ไปที่เมนู **Build** > **Realtime Database** > คลิก **Create Database**
4. เลือก Region: **Singapore (`asia-southeast1`)**
5. ไปที่แท็บ **Rules** และกำหนดสิทธิ์ดังนี้ จากนั้นกด **Publish**:
   ```json
   {
     "rules": {
       ".read": false,
       ".write": false,
       "lab": {
         ".read": true,
         ".write": true
       }
     }
   }
   ```
6. คัดลอก **Database URL** ไว้ เช่น:  
   `https://iot-xxx-default-rtdb.asia-southeast1.firebasedatabase.app`

---

### ขั้นตอนที่ 2: ตั้งค่าและ Flash ESPHome ลง ESP32
1. เปิดไฟล์ `secrets.yaml` แล้วใส่ชื่อ Wi-Fi และรหัสผ่าน:
   ```yaml
   wifi_ssid: "ชื่อ_WIFI_ของคุณ"
   wifi_password: "รหัสผ่าน_WIFI"
   ```
2. เปิดไฟล์ `firebase-l5.yaml` และแก้ไข `YOUR-DATABASE-URL` ให้เป็น URL จริงของโปรเจกต์คุณ (ไม่ต้องใส่ `https://` ซ้ำในโค้ดถ้ามีแล้ว) เช่น:
   ```yaml
   url: "https://iot-xxx-default-rtdb.asia-southeast1.firebasedatabase.app/lab/esp32/latest.json"
   ```
   และ
   ```yaml
   url: "https://iot-xxx-default-rtdb.asia-southeast1.firebasedatabase.app/lab/esp32/history.json"
   ```
3. Compile และ Flash ลง ESP32
4. ดู Log ใน ESPHome Serial/Web Log จะต้องปรากฏ:
   ```
   === WIFI CONNECTED ===
   === TIME SYNCHRONIZED ===
   Send temp=29.1 humi=64.2 light=530
   Latest HTTP status = 200
   History HTTP status = 200
   ```

---

### ขั้นตอนที่ 3: อัปโหลด Web Dashboard ขึ้น GitHub Pages
1. ไปที่ [GitHub.com](https://github.com/) แล้วสร้าง Repository ใหม่ (ตั้งชื่อเช่น `firebase` หรือ `esp32-dashboard`)
2. เลือกตั้งค่าเป็น **Public**
3. อัปโหลดไฟล์ [`index.html`](file:///c:/ESPHome/6_ESPHome_Firebase_Dashboard/index.html) ไปยัง Root ของ Repository
4. ไปที่แท็บ **Settings** ของ Repository > เมนูซ้ายมือเลือก **Pages**
5. ในส่วน **Build and deployment > Branch**:
   - เลือก Branch: `main`
   - เลือกโฟลเดอร์: `/(root)`
   - คลิก **Save**
6. รอประมาณ 1-2 นาที จะได้ URL เช่น:  
   `https://<username>.github.io/firebase/`

---

### ขั้นตอนที่ 4: การใช้งาน Dashboard
- เปิด URL บนเบราว์เซอร์
- หากต้องการเปลี่ยน URL หรือ Node Path สามารถคลิกปุ่ม **⚙️ ตั้งค่า Firebase** ที่มุมขวาบน
- มีปุ่ม **🧪 โหมดจำลอง Demo** สำหรับทดสอบการแสดงผลและกราฟได้ทันทีแม้ยังไม่ได้เปิด ESP32

---

## 📝 การส่งงาน
ส่ง **1 URL** ของ GitHub Pages Dashboard เช่น:
`https://<username>.github.io/firebase/`
