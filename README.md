# KC868-A6 MQTT Relay System

ระบบ IoT ควบคุมรีเลย์ 6 ช่องบนบอร์ด **KinCony KC868-A6** (ESP32) ผ่าน **MQTT**
พร้อมเว็บแดชบอร์ดและแอปมือถือ (PWA ติดตั้งขึ้นหน้าจอโฮมได้), ระบบตั้งเวลาเปิด/ปิดรายช่อง
และคำสั่งเปิด/ปิด/toggle

**MQTT broker ที่ใช้อยู่ตอนนี้: `test.mosquitto.org`** (broker สาธารณะ ไม่ต้อง login,
ใช้ทดสอบได้ทันทีไม่ต้องตั้ง server เอง) — topic ทั้งหมดอยู่ใต้ prefix `pkru/19_04_10_28`
ซึ่งตั้งให้ไม่ซ้ำกับคนอื่นที่ใช้ broker สาธารณะตัวเดียวกัน ถ้าต้องการความเป็นส่วนตัว/ความ
ปลอดภัยมากขึ้นในระยะยาว ให้เปลี่ยนไปใช้ broker ของตัวเอง (มี Mosquitto พร้อมรันให้ใน
`docker-compose.yml` — ดูหัวข้อ "ทางเลือก: รัน broker เอง" ด้านล่าง)

## โครงสร้างไฟล์

```
C:\ESPHome\
├── kc868-a6.yaml           ESPHome firmware config (flash ลงบอร์ด)
├── secrets.yaml.example    เทมเพลต secrets — คัดลอกเป็น secrets.yaml แล้วกรอกค่าจริง
├── docker-compose.yml      (ทางเลือก) รัน MQTT broker ของตัวเองบน PC/เซิร์ฟเวอร์
├── mosquitto/
│   └── mosquitto.conf      (ทางเลือก) ตั้งค่า broker เอง
└── dashboard/
    ├── index.html          เว็บแดชบอร์ด + ตัวแอปมือถือ (PWA)
    ├── manifest.json        ทำให้ติดตั้งเป็นแอปบนมือถือได้ (Add to Home Screen)
    ├── sw.js                 Service worker สำหรับแคช/เปิดเร็ว/ใช้แบบออฟไลน์
    └── icon.svg              ไอคอนแอป
```

## 1) Flash เฟิร์มแวร์ลงบอร์ด

1. ติดตั้ง ESPHome: `pip install esphome` (หรือใช้ ESPHome Dashboard/Home Assistant Add-on)
2. คัดลอก `secrets.yaml.example` → `secrets.yaml`:
   ```powershell
   cd C:\ESPHome
   copy secrets.yaml.example secrets.yaml
   ```
3. เปิด `secrets.yaml` แล้วกรอกอย่างน้อย `wifi_ssid` และ `wifi_password` ของ WiFi ที่จะ
   ให้บอร์ดต่อ (ค่า `encryption_key` และ `mqtt_broker` มีค่าใช้ได้อยู่แล้ว ไม่ต้องแก้ก็รันได้)
4. เข้าโหมด flash: กดปุ่ม **S2** ค้างไว้ พร้อมกับเสียบไฟ 12V เข้าบอร์ด แล้วต่อ USB-C เข้า PC
5. รันคำสั่ง:
   ```powershell
   cd C:\ESPHome
   esphome run kc868-a6.yaml
   ```
6. หลัง flash เสร็จ บอร์ดจะต่อ WiFi และเชื่อม `test.mosquitto.org` อัตโนมัติ

หลังจากนี้สามารถอัปเดตเฟิร์มแวร์ผ่าน WiFi (OTA) ได้โดยไม่ต้องเสียบสาย USB อีก

## 2) ใช้งานเว็บแดชบอร์ด

เปิดไฟล์ `dashboard/index.html` ด้วยเว็บเซิร์ฟเวอร์ใดก็ได้ เช่น:

```powershell
cd C:\ESPHome\dashboard
python -m http.server 8080
```

แล้วเปิด `http://localhost:8080` — กดไอคอนเฟือง ⚙ เพื่อดู/ตั้งค่า MQTT host/port
(default ไว้เป็น `test.mosquitto.org` พอร์ต `8080` ซึ่งเป็น WebSocket ของ broker นี้อยู่แล้ว
ไม่ต้องกรอก username/password เพราะเป็น broker สาธารณะ ไม่ต้อง login) กด **Connect** ได้เลย

หน้าแดชบอร์ดจะแสดงสถานะรีเลย์ทั้ง 6 ช่องแบบเรียลไทม์ (จุดสีเขียว = เปิด) และมีปุ่ม
**ON / OFF / TOGGLE** ต่อช่อง พร้อมช่องตั้งเวลาเปิด-ปิดและสวิตช์เปิดใช้งานตารางเวลา

## 3) ใช้งานเป็นแอปมือถือ

หน้าแดชบอร์ดเดียวกันนี้เป็น **PWA (Progressive Web App)** — เปิดผ่าน URL เดียวกัน
บนมือถือ (ต้องเสิร์ฟผ่าน HTTPS หรือ IP ในเครือข่ายเดียวกัน) แล้วเลือก:

- **Android (Chrome):** เมนู ⋮ → "Add to Home screen"
- **iPhone (Safari):** ปุ่มแชร์ → "Add to Home Screen"

จะได้ไอคอนแอปบนหน้าจอโฮม เปิดแบบเต็มจอไม่มีแถบ URL และมีฟังก์ชันครบเหมือนหน้าเว็บ
(สถานะรีเลย์ + ปุ่มเปิด/ปิด/toggle + ตั้งเวลา) เพราะใช้ MQTT connection เดียวกัน

## MQTT Topic Reference

Topic prefix: `pkru/19_04_10_28`

| Topic | ทิศทาง | ค่า |
|---|---|---|
| `pkru/19_04_10_28/status` | บอร์ด → | `online` / `offline` (LWT) |
| `pkru/19_04_10_28/cmd/relay/N/toggle` | → บอร์ด | payload อะไรก็ได้ — สลับสถานะ |
| `pkru/19_04_10_28/cmd/relay/N/on` | → บอร์ด | payload อะไรก็ได้ — เปิด |
| `pkru/19_04_10_28/cmd/relay/N/off` | → บอร์ด | payload อะไรก็ได้ — ปิด |
| `pkru/19_04_10_28/cmd/relay/N/on_time` | → บอร์ด | นาทีนับจากเที่ยงคืน (0-1439, -1 = ปิด) |
| `pkru/19_04_10_28/cmd/relay/N/off_time` | → บอร์ด | เช่นเดียวกับข้างบน |
| `pkru/19_04_10_28/cmd/relay/N/sched_enable` | → บอร์ด | `ON`/`OFF` เปิด-ปิดการใช้ตารางเวลาของช่อง N |
| `pkru/19_04_10_28/state/relay/N/status` | บอร์ด → | `ON` / `OFF` |
| `pkru/19_04_10_28/state/relay/N/on_time` | บอร์ด → | นาทีนับจากเที่ยงคืน |
| `pkru/19_04_10_28/state/relay/N/off_time` | บอร์ด → | นาทีนับจากเที่ยงคืน |
| `pkru/19_04_10_28/state/relay/N/sched_enable` | บอร์ด → | `ON` / `OFF` |

`N` = 1 ถึง 6 — ลองทดสอบส่ง/ดู message ได้ด้วย MQTT Explorer (ต่อไปที่ `test.mosquitto.org`
พอร์ต 1883) ตามที่ตั้งไว้ในโปรแกรมของคุณ

## ทางเลือก: รัน broker เอง (แนะนำเมื่อใช้งานจริง)

`test.mosquitto.org` สะดวกสำหรับทดสอบ แต่เป็น broker สาธารณะไม่มีการยืนยันตัวตน ใครก็ตาม
ที่รู้ topic prefix สามารถสั่งเปิด/ปิดรีเลย์ได้ ถ้าจะใช้งานจริง (ควบคุมอุปกรณ์ในบ้าน) แนะนำ
ให้เปลี่ยนมาใช้ broker ของตัวเอง:

```powershell
cd C:\ESPHome
docker compose up -d
docker exec -it kc868-mosquitto mosquitto_passwd -c /mosquitto/config/passwd kc868
docker compose restart
```

แล้วแก้ `secrets.yaml`: เปลี่ยน `mqtt_broker` เป็น IP เครื่องที่รัน Docker และเพิ่ม
`mqtt_username` / `mqtt_password` กลับเข้าไปใน `kc868-a6.yaml` ส่วน `mqtt:` (มีตัวอย่าง
comment ไว้ในไฟล์แล้ว) ส่วนแดชบอร์ดก็เปลี่ยน host/port เป็น IP เครื่องนั้น + พอร์ต `9001`

## หมายเหตุ

- บอร์ดมีเว็บ UI ในตัวอยู่แล้วที่ `http://kc868-a6.local` (จาก `web_server:` ใน YAML)
  ไว้ใช้ debug เร็ว ๆ แยกจากแดชบอร์ดหลักใน `dashboard/`
- ตารางเวลาเช็คทุก 30 วินาทีเทียบกับเวลาปัจจุบัน (sync ผ่าน NTP, ตั้ง timezone เป็น
  `Asia/Bangkok` ใน YAML แล้ว — แก้ได้ตามต้องการ)
- ถ้าเปลี่ยนไปใช้ broker ส่วนตัว ให้ตั้ง `allow_anonymous false` และมี username/password
  เสมอเพื่อความปลอดภัย (ตั้งไว้ให้แล้วใน mosquitto.conf)
