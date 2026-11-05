# Week 05: ผลที่คาดว่าจะได้ (Lab 5.2 - 5.4)

> **สถานะ: ผลที่คาดว่าจะได้ ยังไม่ได้ทดสอบบนบอร์ดจริง**
> ทุกค่าในไฟล์นี้เป็นการคาดการณ์จากโค้ดและใบงาน ต้องแฟลชลงบอร์ดแล้วนำค่าจริงไปกรอกในตารางของใบงาน ห้ามใช้เป็นผลการทดลอง

Lab 5.1 มีผลวัดจริงอยู่ใน `07-Labsheet-05-1-Wi-Fi-Scan-Phase.md` แล้ว จึงไม่ได้อยู่ในไฟล์นี้

## ก่อนแฟลช

แก้ชื่อและรหัส Wi-Fi ที่จะใช้ทดสอบใน `main/main.c` แล้ว build ใหม่ ถ้าไม่แก้ เคสที่ควรเชื่อมต่อสำเร็จจะล้มเหลว

| Lab | ค่าที่ต้องแก้ |
| :--- | :--- |
| 5.2 | `EXAMPLE_ESP_WIFI_SSID`, `EXAMPLE_ESP_WIFI_PASS` |
| 5.3 | `TARGET_WIFI_SSID`, `TARGET_WIFI_PASS` |
| 5.4 | `TARGET_WIFI_SSID`, `TARGET_WIFI_PASS` (ค่าเดิมเป็น Hotspot ของอาจารย์) |

## Lab 5.2: Connection Phase

| ข้อ | สถานการณ์ | Event สุดท้าย | ผล | Reason Code |
| :---: | :--- | :--- | :---: | :--- |
| 5.2.1 | SSID และ Password ถูก | `IP_EVENT_STA_GOT_IP` | Passed | ไม่มี |
| 5.2.2 | SSID ผิด | `WIFI_EVENT_STA_DISCONNECTED` | Failed | 201 / 0xC9 (`NO_AP_FOUND`) |
| 5.2.3 | Password ผิด | `WIFI_EVENT_STA_DISCONNECTED` | Failed | 15 / 0x0F หรือ 204 / 0xCC (Handshake Timeout) |

ค่าใน 5.2.1 ขึ้นกับ AP ที่ใช้: Subnet Mask มักเป็น `255.255.255.0` ส่วน IP, Gateway, BSSID และ Channel ต้องอ่านจาก Log

**ข้อควรระวัง:** ข้อความ Reason ในโค้ดสลับเลขกันอยู่ `WIFI_REASON_4WAY_HANDSHAKE_TIMEOUT` มีค่าจริง 15 แต่โค้ดพิมพ์ว่า (204) ให้ยึดบรรทัด `Reason Code (Decimal)` เป็นหลัก

## Lab 5.3: Auth & Assoc Phase

| ข้อ | สถานการณ์ | Event | Link Layer | AID | Reason Code |
| :---: | :--- | :--- | :---: | :---: | :--- |
| 5.3.1 | AP มีจริง | `WIFI_EVENT_STA_CONNECTED` | สำเร็จ | เลขจำนวนเต็มบวก มักเป็น 1 ถ้ามีเครื่องเดียว | ไม่มี |
| 5.3.2 | AP ไม่มีจริง | `WIFI_EVENT_STA_DISCONNECTED` | ไม่สำเร็จ | ไม่มี | 201 |

`Auth Mode Enum` ควรเป็น 3 เมื่อ AP ใช้ `WIFI_AUTH_WPA2_PSK`

## Lab 5.4: Handshake & IP Phase

| ข้อ | สถานการณ์ | `STA_CONNECTED` | `GOT_IP` | ผล | Reason Code |
| :---: | :--- | :---: | :---: | :---: | :--- |
| 5.4.1 | Password ถูก | เกิด | เกิด | สำเร็จ | ไม่มี |
| 5.4.2 | Password ผิด | อาจเกิดสั้นๆ ตามที่ใบงานอธิบาย | ไม่เกิด | ล้มเหลว | 15 หรือ 204 |

ถ้า AP เป็น WPA3 ค่า Reason Code อาจต่างจากตาราง ให้บันทึกตามค่าจริง

## สิ่งที่แก้ในโค้ด

Lab 5.2 และ 5.3 build บน ESP-IDF v6.0.2 ไม่ผ่าน เพราะ enum ถูกเปลี่ยนชื่อ จึงแก้เฉพาะ `case` และคงข้อความเดิมไว้

| ชื่อเดิม | ชื่อใน v6 | ค่า |
| :--- | :--- | :---: |
| `WIFI_REASON_ASSOC_EXPIRE` | `WIFI_REASON_DISASSOC_DUE_TO_INACTIVITY` | 4 |
| `WIFI_REASON_NOT_AUTHED` | `WIFI_REASON_CLASS2_FRAME_FROM_NONAUTH_STA` | 6 |
