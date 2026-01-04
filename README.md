README.txt
คู่มือพื้นฐาน: Amlogic update bulkcmd + UART/TTL (U‑Boot/Android)

⚠️ คำเตือนสำคัญ
- ทำเฉพาะกับอุปกรณ์ที่ “เป็นของคุณ/ได้รับอนุญาต” เท่านั้น
- คำสั่งบางอย่าง (ล้างพาร์ทิชัน/แฟลชบูตโหลดเดอร์) มีโอกาสทำให้เครื่อง “บริค” ได้
- UART ของ TV Box ส่วนมากเป็น 3.3V (ห้ามใช้ 5V) และ “ห้ามต่อ VCC” จาก USB‑TTL ไปที่บอร์ดถ้าไม่มั่นใจ

────────────────────────────────────────────────────────────
1) โหมด/คำศัพท์ที่เจอบ่อย (Amlogic)
- Burn/WorldCup mode: โหมดที่เครื่องเข้าสู่การแฟลชผ่าน USB (USB Burning Tool / update tool)
- U‑Boot console: เชลล์บูตโหลดเดอร์ที่คุยผ่าน UART/TTL
- Fastboot mode: โหมดแฟลช/สั่งงานบูตโหลดเดอร์ด้วย fastboot (Android platform tools)

────────────────────────────────────────────────────────────
2) เครื่องมือที่ใช้ (เลือกตามงาน)
A) แฟลชเฟิร์มแวร์ (GUI)
- Amlogic USB Burning Tool (Windows) ใช้แฟลชไฟล์ .img เพื่อกู้/อัปเกรดเฟิร์มแวร์

B) สั่งคำสั่งผ่าน USB แบบบรรทัดคำสั่ง
- Amlogic Update Tool / update.exe (Windows) หรือไบนารีบน Linux (บางแพ็กเรียก “update”)
  ใช้ส่งคำสั่งไปยัง U‑Boot ผ่าน USB (WorldCup/libusb driver) เช่น bulkcmd, partition ฯลฯ

C) ทางเลือกโอเพ่นซอร์ส/ครอสแพลตฟอร์ม (ขึ้นกับรุ่น/ชิป/ไดรเวอร์)
- aml-flash-tool (เช่นของ OSMC / Khadas utils)
- bulkcmd.py (บางงานวิจัย/โปรเจกต์ reverse-engineering ใช้แทน update bulkcmd)

D) UART/TTL
- USB‑to‑TTL adapter แบบ 3.3V (เช่น CP2102 / FT232 / CH340)
- โปรแกรม Terminal: PuTTY (Windows), minicom/screen (Linux), TeraTerm ฯลฯ

────────────────────────────────────────────────────────────
3) การต่อสาย UART/TTL (พื้นฐาน)
อุปกรณ์: USB‑TTL 3.3V + สาย Dupont
หลักการต่อ (ไขว้สัญญาณ):
- GND (USB‑TTL)  → GND (บอร์ด)
- TXD (USB‑TTL)  → RX (บอร์ด)
- RXD (USB‑TTL)  → TX (บอร์ด)
- (ปกติ) ไม่ต้องต่อ VCC

ค่าตั้งพอร์ตที่พบบ่อย:
- 115200 baud, 8 data bits, No parity, 1 stop bit (115200 8N1)
(บางบอร์ดอาจเป็น 1500000 หรือ 921600 ให้ลองไล่)

Tip:
- ถ้า “ตัวอักษรก๊องแก๊ง” ส่วนใหญ่คือ baud ไม่ตรง หรือสลับ TX/RX

────────────────────────────────────────────────────────────
4) คำสั่งพื้นฐานผ่าน UART (U‑Boot console)
หมายเหตุ: คำสั่งที่มีจริง “ขึ้นกับ U‑Boot ของแต่ละเครื่อง” ให้เริ่มจาก help ก่อน

# ดูคำสั่งทั้งหมด
help

# ดูตัวแปรสภาพแวดล้อม (env)
printenv

# ตั้งค่า env (ตัวอย่าง)
setenv bootdelay 3
saveenv

# รีบูต/รีเซ็ต
reset

# ตรวจสื่อเก็บข้อมูล
mmc info
mmc list

# เริ่ม USB (บางรุ่นต้องสั่งก่อนใช้แฟลชผ่าน USB storage)
usb start
usb stop

# เข้าสู่ fastboot จาก U‑Boot (บางบอร์ดใช้รูปแบบนี้)
fastboot 0
# หรือบางเจ้าใช้คำสั่ง/สคริปต์: run fastboot / run update ฯลฯ
run update

────────────────────────────────────────────────────────────
5) update bulkcmd คืออะไร + ใช้ทำอะไรได้
แนวคิด:
- bulkcmd = การ “ส่งคำสั่ง U‑Boot” ผ่าน USB (WorldCup/Update Tool) เหมือนเราพิมพ์ใน UART นั่นแหละ
- ดังนั้นคำสั่งที่ใช้ได้ = คำสั่งที่ U‑Boot ของเครื่องนั้นรองรับ

รูปแบบคำสั่ง (ตัวอย่างแนวทาง)
Windows (update.exe):
  update.exe bulkcmd "help"
  update.exe bulkcmd "printenv"
  update.exe bulkcmd "reset"

Linux (บางแพ็กชื่อ update):
  sudo ./update bulkcmd "help"

ตัวอย่างที่พบบ่อย (ใช้เมื่อจำเป็น)
- เข้า fastboot จากโหมด Update/WorldCup:
  update.exe bulkcmd fastboot
  (บางรุ่นต้องใส่เป็นสตริง: update.exe bulkcmd "fastboot")

- รีเซ็ต:
  update.exe bulkcmd "reset"

- แฟลชไฟล์แยกพาร์ทิชัน (ตัวอย่างจากเอกสาร/รีลีสโน้ตบางชุด):
  update.exe partition bootloader u-boot.bin
  update.exe partition boot dtb.img
  update.exe partition boot boot.img

⚠️ “erase / store erase / keyman / key write” เป็นคำสั่งที่อาจลบถาวรหรือกระทบคีย์/DRM
ให้ใช้เฉพาะเมื่อมีเอกสารของรุ่นนั้น ๆ และรู้ผลลัพธ์แน่ชัด

────────────────────────────────────────────────────────────
6) เข้าสู่ Fastboot แบบ “ปลอดภัย”
วิธีที่พบบ่อย:
A) จาก Android (ถ้าบูตเข้าได้)
- adb reboot bootloader

B) จาก U‑Boot (UART)
- fastboot 0   (ถ้ารองรับ)
- หรือ run fastboot / run update (แล้วแต่ env)

C) จาก update bulkcmd (โหมด WorldCup)
- update.exe bulkcmd fastboot

────────────────────────────────────────────────────────────
7) ปลดล็อก Bootloader (แนวทางที่ถูกต้องตาม Android)
- โดยทั่วไปการปลดล็อกทำผ่าน fastboot:
  fastboot flashing unlock
- การปลดล็อกจะมีหน้าจอยืนยันบนเครื่อง และ “ล้างข้อมูล (factory reset)” เพื่อป้องกันการเข้าถึงข้อมูลโดยไม่ได้รับอนุญาต
- การล็อกกลับ:
  fastboot flashing lock
- ผ่านตัว update หรือ bulkcmd
update bulkcmd "setenv -f EnableSelinux permissive"
update bulkcmd "env set lock 0"
update bulkcmd "env set avb2 0"
update bulkcmd "env set oem_lock unlock"
update bulkcmd "store rom_protect off"
update bulkcmd "saveenv"

หมายเหตุสำคัญ:
- “การพยายามปลดล็อก/บายพาสผ่าน UART ด้วยการแก้ค่า secure/efuse/คีย์” เป็นเรื่องเฉพาะรุ่นและมีความเสี่ยงสูง
  (และอาจเข้าข่ายการหลบเลี่ยงระบบความปลอดภัย) จึงไม่ควรทำหากไม่มีเอกสารผู้ผลิต/สิทธิ์ที่ถูกต้อง

────────────────────────────────────────────────────────────
8) เปลี่ยน Serial/USID แบบที่ “ไม่ไปยุ่งคีย์ถาวร” (แนวทางทั่วไป)
- หลายบอร์ดมี serial อยู่ใน U‑Boot env เช่น serial#, usid, androidboot.serialno ฯลฯ
แนวทาง:
1) ดูก่อนว่าเก็บชื่ออะไร
   printenv
   (แล้วมองหาบรรทัดที่เกี่ยวกับ serial/usid/androidboot.serialno)
   หรือถ้ารู้ชื่อตัวแปรแล้วให้ระบุชื่อตรง ๆ เช่น:
   printenv serial#
   printenv usid
2) ถ้าเจอเป็นตัวแปร env:
    update.exe bulkcmd "keyman init 0x1234"
    update.exe bulkcmd "keyman write usid str 1234567890"
    update.exe bulkcmd "saveenv"
3) รีบูต
   reset

หมายเหตุ:
- ถ้าซีเรียลถูกเก็บใน “พื้นที่คีย์/OTP/eFuse (เช่น keyman)” การเขียนผิดอาจแก้กลับไม่ได้
  แนะนำให้ใช้วิธีที่ผู้ผลิตระบุเท่านั้น

────────────────────────────────────────────────────────────
9) ลิงก์รูปประกอบ (เอาไว้ดูภาพประกอบ/การต่อสาย/หน้าตาเครื่องมือ)
(วางลิงก์ไว้ในโค้ดบล็อกตามข้อกำหนดของระบบ)

- Amlogic USB Burning Tool (คู่มือ+ภาพหน้าจอ):
  https://wiki.coreelec.org/coreelec:aml_usb_tool

- ตัวอย่างภาพ “UART pinout บนบอร์ด Amlogic” (กระทู้ LibreELEC):
  https://forum.libreelec.tv/thread/29167-unreadable-uart-output-on-s905-board/

- ภาพตัวอย่าง USB‑to‑TTL (CP2102):
  https://www.amazon.com/HiLetgo-CP2102-Converter-Adapter-Downloader/dp/B00LODGRV8

- ภาพไดอะแกรมหลักการต่อ UART (TX/RX/GND):
  https://www.slideshare.net/slideshow/uart-communication/250706087

────────────────────────────────────────────────────────────
10) แหล่งอ้างอิง/อ่านต่อ (แนวคิด/เอกสาร)
- Android AOSP: Lock/Unlock bootloader & fastboot flashing unlock/lock
  https://source.android.com/docs/core/architecture/bootloader/locking_unlocking
- CoreELEC Wiki: Amlogic USB Burning Tool
  https://wiki.coreelec.org/coreelec:aml_usb_tool
- เอกสาร/รีลีสโน้ตที่พูดถึง update.exe และตัวอย่างคำสั่ง partition/bulkcmd (Openlinux release notes):
  (อาจเป็น PDF กระจายหลายที่ ให้ค้นคำว่า “Amlogic Openlinux Release Notes update.exe bulkcmd”)

จบไฟล์
