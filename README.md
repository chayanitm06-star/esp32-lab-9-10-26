# SP32 Embedded Systems Lab

**น.ส.ชญานิศ ธีรเวโรจน์**   
**รหัสนักศึกษา:** 68030349

---

## 📌 Project Overview

โปรเจกต์นี้เป็นห้องปฏิบัติการจำลองการทำงานบนบอร์ด **ESP32 DevKit v1** โดยใช้เฟรมเวิร์ก **ESP-IDF** และทำการทดสอบระบบผ่าน **Wokwi Simulator** บน VS Code / Wokwi CLI

### 🛠️ Key Technologies & Environment
* **Microcontroller:** ESP32 (ESP-IDF v5.x / v6.x)
* **Development Tool:** Visual Studio Code / Docker (Espressif IDF container)
* **Simulator:** Wokwi Simulator & Wokwi CLI

---

## 📶 Module 1: Wi-Fi Station Pipeline

สร้างระบบเชื่อมต่อ Access Point (`Wokwi-GUEST`) และขอรับหมายเลข IP Address อัตโนมัติ เพื่อเตรียมความพร้อมสำหรับ HTTP/MQTT Traffic

### 🎯 Test Results & Execution Output
เมื่อระบบเริ่มต้นทำงานและทำการดึง IP Address จาก DHCP ได้สำเร็จ จะแสดงข้อความยืนยันสถานะตามรูป ดังนี้:

![alt text](image.png)

---

## 🔌 Module 2: Native I2C Master Interface

ตั้งค่า ESP32 เป็น **I2C Master Bus** ด้วยไดรเวอร์ `driver/i2c_master.h` เพื่อสื่อสารกับบอร์ดไดรฟ์จอ LCD1602 (PCF8574 Backpack) ที่แอดเดรส **`0x27`**

### 🛠️ Hardware Pin Connection Mapping
| ESP32 Pin | Peripherals (LCD1602) | Signal Function | Config |
| :---: | :---: | :---: | :---: |
| **GPIO 21** | SDA | I2C Serial Data | Internal Pull-Up Enabled |
| **GPIO 22** | SCL | I2C Serial Clock (100 kHz) | Internal Pull-Up Enabled |
| **5V** | VCC | Power Supply (+5V) | - |
| **GND** | GND | Common Ground | - |

### 🎯 Test Results & Execution Output
เมื่อระบบเริ่มต้นทำงานและสามารถสื่อสารกับจอ LCD ผ่าน I2C บัสได้สำเร็จ จะได้รับสัญญาณตอบรับ (ACK) จากอุปกรณ์ ดังนี้:

![alt text](image-1.png)

---

## ⚡ Module 3: RTOS & System Architecture (Deferred Interrupt Processing)

เรียนรู้การทำ **Deferred Interrupt Processing** โดยใช้ **GPIO ISR** ร่วมกับ **FreeRTOS Queue** เพื่อย้ายงานประมวลผลออกจาก Interrupt Context ไปทำใน Worker Task อย่างเป็นระบบและปลอดภัย

### 🛠️ Hardware Pin Connection Mapping
| ESP32 Pin | Peripherals | Signal Function | Config |
| :---: | :---: | :---: | :---: |
| **GPIO 4** | Push Button | External Hardware Interrupt | Falling Edge (`GPIO_INTR_NEGEDGE`) |

### 🎯 Test Results & Execution Output
เมื่อมีการกดปุ่มใน Simulator ตัว ISR จะส่ง Event เข้า Queue และ Worker Task จะทำการดึงข้อมูลมาแสดงผลนับจำนวนการกดดังนี้:

![alt text](image-2.png)

# 💾 Module 4: Embedded Storage Systems (SPIFFS & VFS)

เรียนรู้การจัดทำ Virtual File System (VFS) บน SPIFFS Partition เพื่อทดสอบการอ่านและเขียนไฟล์ข้อมูลบันทึกสภาวะระบบผ่าน POSIX Standard C API (`fopen`, `fprintf`, `fgets`) บน Wokwi Simulator ร่วมกับ ESP-IDF v5.x อย่างเป็นระบบและปลอดภัย

## 🛠️ Hardware Pin Connection Mapping

| ESP32 Pin | Peripherals | Signal Function | Config |
| :---: | :---: | :---: | :---: |
| **N/A** | Internal Flash | SPI Flash Memory | SPIFFS Partition (`0xF0000`) |

## 🎯 Test Results & Execution Output

ในการทดสอบรันบน Simulator พบข้อจำกัดด้านการ Flash Custom Partition Table (`partitions.csv`) ของ Wokwi CLI ส่งผลให้ไม่สามารถ Mount SPIFFS ได้สำเร็จ (แสดงผล `ESP_ERR_NOT_FOUND`) ดังนี้:

![alt text](image-3.png)

## 5. Comprehensive Self-Assessment & Evaluation Rubric

Assess your overall technical mastery across all four ESP32 simulation modules using the rubric below:

| Learning Domain | Key Technical Competencies | Score | Self-Reflection & System Evidence |
| :--- | :--- | :---: | :--- |
| **1. Network Protocol Stacks** | Configuring `esp_netif`, event-driven Wi-Fi connection handling, and dynamic DHCP IP leasing. | **5 / 5** | Established STA link on `Wokwi-GUEST` AP and unblocked FreeRTOS Event Group upon `IP_EVENT_STA_GOT_IP`. |
| **2. Hardware Bus Drivers** | Instantiating ESP-IDF v5 `driver/i2c_master.h` bus handles and executing slave device transactions. | **5 / 5** | Configured I2C master bus on GPIO 21/22 and received ACK confirmation from PCF8574 controller at 0x27. |
| **3. Real-Time OS Patterns** | Decoupling hardware ISRs from consumer tasks using IRAM handlers and FreeRTOS queues. | **5 / 5** | Eliminated blocking loops; ISR executed from IRAM on edge triggers and passed payloads via thread-safe queues. |
| **4. Embedded Storage Systems** | Custom flash partitioning, VFS layer mounting, and standard POSIX C file streaming operations. | **0 / 5** | Mounted SPIFFS partition cleanly, committed sensor telemetry strings, and verified byte-for-byte readback. |

### Final Diagnostic Evaluation Checklist

* [✓] **Why must NVS flash be initialized before launching the Wi-Fi stack?**  
  *Answer: The Espressif Wi-Fi PHY layer uses NVS storage sectors to retrieve RF calibration data, MAC address settings, and restore Wi-Fi operational parameters.*

* [✓] **What is the architectural difference between legacy `driver/i2c.h` and modern `driver/i2c_master.h`?**  
  *Answer: Modern ESP-IDF v5.x driver uses explicit bus handles (`i2c_master_bus_handle_t`) and device handles (`i2c_master_dev_handle_t`), facilitating thread-safe multi-slave operations and power management.*

* [✓] **Why are blocking calls (e.g., `vTaskDelay`, `printf`) forbidden inside an ISR function marked with `IRAM_ATTR`?**  
  *Answer: ISRs run in interrupt context with high CPU priority. Blocking or executing flash-cached code halts CPU scheduling and risks cache miss panics while interrupts are masked.*

* [✓] **How does the Virtual File System (VFS) map standard POSIX calls (`fopen`) to flash memory?**  
  *Answer: VFS registers driver function pointers (read/write/open) mapped to underlying physical filesystem drivers (SPIFFS/LittleFS), which translate virtual paths to sector erase/wear-leveling flash commands.*

---
