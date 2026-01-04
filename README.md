คู่มือรวมคำสั่ง (เยอะๆ) : Amlogic update bulkcmd + UART/TTL + U‑Boot + Fastboot
เวอร์ชัน: 2.1 (เพิ่ม run update / update command / keyman อธิบายละเอียดแบบปลอดภัย)

⚠️ คำเตือนสำคัญ
- ทำเฉพาะกับอุปกรณ์ที่ “เป็นของคุณ/ได้รับอนุญาต” เท่านั้น
- คำสั่งประเภท write/erase/flash มีโอกาสทำให้เครื่อง “บริค” ได้ทันที
- UART ของ TV Box ส่วนมากเป็น 3.3V (ห้ามใช้ 5V) และ “ห้ามต่อ VCC” จาก USB‑TTL ไปที่บอร์ดถ้าไม่มั่นใจ
- อุปกรณ์บางรุ่นมี Secure Boot / AVB / OEM Lock / eFuse: จะปลดล็อกไม่ได้ถ้าไม่มีขั้นตอน/คีย์จากผู้ผลิต
- ไฟล์นี้ “จะไม่สอน” การเขียน/แก้คีย์ถาวร (DRM/OTP/eFuse) หรือวิธีบายพาสระบบความปลอดภัย

────────────────────────────────────────────────────────────
0) หลักสำคัญที่สุด: “คำสั่งจริงของเครื่องคุณ” ต้องดูจาก help
U‑Boot ของแต่ละกล่องไม่เหมือนกัน แถม vendor บางเจ้าเพิ่มคำสั่งเฉพาะ (store, keyman, usb_update ฯลฯ)
วิธีเอา “ครบที่สุด” คือดูจาก help บนเครื่องคุณเอง:

บน UART (U‑Boot):
  help
  help <ชื่อคำสั่ง>

บน update.exe (bulkcmd ส่งเข้า U‑Boot):
  update.exe bulkcmd "help"
  update.exe bulkcmd "help env"
  update.exe bulkcmd "help store"
  update.exe bulkcmd "help fastboot"

────────────────────────────────────────────────────────────
1) คำที่คนชอบสับสน:  update  vs  run update  vs  update.exe bulkcmd "update"
1.1  update (คำสั่ง U‑Boot)
- บางบอร์ด (โดยเฉพาะชุดอ้างอิงของ Amlogic/บาง vendor) มี “คำสั่งชื่อ update” ใน U‑Boot
- จุดประสงค์มักคือ “เข้า USB burning mode / WorldCup mode” เพื่อให้ PC แฟลชผ่าน USB ได้

ตัวอย่างการเรียก (ผ่าน UART):
  update
แล้วเครื่องจะเข้าโหมดที่ PC เห็นอุปกรณ์สำหรับแฟลช (USB Burning Tool / update.exe)

1.2  run update (คำสั่ง U‑Boot: run + env variable ชื่อ update)
- run = สั่งให้ U‑Boot รัน “สคริปต์ที่เก็บในตัวแปร env”
- update (ในกรณีนี้) คือ “ชื่อ env variable” ที่ภายในอาจเรียกคำสั่งหลายตัว เช่น init USB, ตั้งค่า GPIO แล้วค่อยเรียก update ฯลฯ

วิธีเช็คก่อนใช้:
  printenv update
หรือ (U‑Boot ใหม่):
  env print update

ถ้าพบว่ามีค่าอยู่ เช่น update=xxx;yyy;zzz
ให้รัน:
  run update

1.3  update.exe bulkcmd "update" (ฝั่ง PC ส่งคำว่า update เข้า U‑Boot)
- bulkcmd คือส่ง “คำสั่ง U‑Boot” ผ่าน USB (WorldCup/Update protocol)
- ดังนั้นถ้า U‑Boot มีคำสั่ง update จริง คุณสามารถสั่งจาก PC ได้ว่า:
  update.exe bulkcmd "update"
(ผลลัพธ์ขึ้นกับ U‑Boot: อาจเข้า burn mode / หรือเป็นคำสั่งคนละอย่าง)

สรุปสั้นๆ:
- “update” = อาจเป็นคำสั่ง U‑Boot
- “run update” = ใช้คำสั่ง run เพื่อรัน env variable ชื่อ update
- “update.exe bulkcmd …” = PC ส่งข้อความนั้นเข้า U‑Boot (เหมือนคุณพิมพ์ผ่าน UART)

────────────────────────────────────────────────────────────
2) update bulkcmd fastboot / update bulkcmd update (แนวทางใช้งานจริง)
2.1 เข้า fastboot “จาก WorldCup/Update tool”
เงื่อนไข: U‑Boot ต้องรองรับคำสั่ง fastboot (ดูได้จาก help fastboot)

บน PC:
  update.exe identify
  update.exe bulkcmd "help fastboot"
  update.exe bulkcmd "fastboot"          (บางรุ่น)
  update.exe bulkcmd "fastboot usb 0"    (บางรุ่นตาม U‑Boot docs)
จากนั้นใช้ fastboot บน PC:
  fastboot devices
  fastboot getvar all

2.2 เข้า USB burning mode “จาก U‑Boot”
เงื่อนไข: U‑Boot ต้องมีคำสั่ง update (หรือมี env script update)

บน UART:
  update
หรือ:
  run update

2.3 “ฮาร์ดคอ” ที่เจอบ่อย: สคริปต์ env รวมหลายคำสั่งแล้วเรียก run
- vendor มักเก็บเป็น env เช่น: upgrade, update, storeboot, recovery_from_sd ฯลฯ
- หลักการคือดูด้วย printenv แล้วค่อย run ตามชื่อที่มีจริง

ตัวอย่างที่ปลอดภัย (อ่านอย่างเดียว):
  printenv
  printenv bootcmd
  printenv upgrade
แล้วค่อย:
  run upgrade   (ถ้ามี)

────────────────────────────────────────────────────────────
3) keyman คืออะไร (อธิบายแบบปลอดภัย) + ทำไมถึงเจอคำสั่งพวกนี้
keyman (ใน U‑Boot สาย Amlogic) มักเป็น “ระบบอ่าน/จัดการคีย์/ข้อมูลประจำเครื่อง” เช่น usid, mac ฯลฯ
หลาย vendor ใช้ keyman “อ่าน usid แล้วเอาไปใส่ใน bootargs” เพื่อให้ Android เห็น serialno

สิ่งที่ควรรู้ก่อนแตะ keyman:
- บางอุปกรณ์เก็บคีย์ในที่ “ถาวร/ย้อนกลับไม่ได้” (OTP/eFuse/secure storage)
- การ “เขียน” ค่าผิด อาจทำให้ DRM/Network/Provisioning พังถาวร
- ไฟล์นี้จึง “ไม่สอนการเขียน/เปลี่ยน usid/mac/key” ผ่าน keyman (เพื่อความปลอดภัย)

สิ่งที่ทำได้แบบปลอดภัยกว่า:
A) ใช้ keyman “อ่านเพื่อดีบัก” (ถ้าเครื่องคุณ/งานคุณจำเป็นจริง และคุณมีสิทธิ์)
- ขั้นตอน/รูปแบบคำสั่ง “แตกต่างตาม U‑Boot” ให้ดู `help keyman` ก่อนทุกครั้ง

B) ถ้าต้องการ “เปลี่ยน serial แบบชั่วคราว/ไม่ถาวร”
- ให้ลองดูว่ามีตัวแปร env อย่าง serial#, serialno, androidboot.serialno ไหม
  printenv | หา serial
- ถ้ามี ให้เปลี่ยนผ่าน env แล้ว saveenv (ความเสี่ยงต่ำกว่าการเขียนคีย์ถาวร)

────────────────────────────────────────────────────────────
4) U‑Boot: run / env / saveenv (พื้นฐานที่ต้องรู้)
คำสั่งพื้นฐาน:
  help
  printenv
  setenv <var> <value>
  saveenv
  run <var>
  reset

ตัวอย่างแนวคิด run (ไม่ต้องทำตามตรงๆ):
- bootcmd มักเป็นสคริปต์ที่เรียก run distro_bootcmd หรือ run storeboot
- คุณแก้ bootdelay/bootargs ได้ แล้ว saveenv เพื่อเก็บถาวร

────────────────────────────────────────────────────────────
5) Fastboot ฝั่ง PC (พื้นฐาน→โหด)
พื้นฐาน:
  fastboot devices
  fastboot getvar all
  fastboot reboot
แฟลช:
  fastboot flash <partition> <file.img>
ล้าง:
  fastboot erase <partition>
A/B:
  fastboot getvar current-slot
  fastboot set_active a|b

────────────────────────────────────────────────────────────
6) ลิงก์อ้างอิง + รูปประกอบ (อ่านต่อ/ภาพต่อสาย)
```text
U‑Boot: Environment variables / env / ตัวอย่าง bootcmd ที่ใช้ run …
https://docs.u-boot.org/en/latest/usage/environment.html
https://docs.u-boot.org/en/latest/usage/cmd/env.html

U‑Boot: Fastboot (คำสั่งฝั่ง U‑Boot เช่น fastboot usb 0)
https://docs.u-boot.org/en/latest/android/fastboot.html

Android AOSP: Lock/Unlock bootloader (แนวทางมาตรฐาน)
https://source.android.com/docs/core/architecture/bootloader/locking_unlocking

CoreELEC Wiki: Amlogic USB Burning Tool (มีภาพ/ขั้นตอน)
https://wiki.coreelec.org/coreelec:aml_usb_tool

SparkFun: CP2102 USB‑to‑Serial Hookup Guide (มีรูปต่อสาย TX/RX/GND)
https://learn.sparkfun.com/tutorials/cp2102-usb-to-serial-converter-hook-up-guide/all

Openlinux Release Notes (มีตัวอย่าง “ใน uboot ให้รัน update เพื่อเข้า usb burning mode”
และมีตัวอย่าง update.exe partition + update.exe bulkcmd "reset")
https://device.report/m/5b1e754da9e6a89d81016558396aaf6aa76d3f9ea272fff79c072df21720dda4.pdf
```

────────────────────────────────────────────────────────────
7) เช็กลิสต์เซฟๆ ก่อนลองคำสั่งหนักๆ
- เซฟ output ของ `printenv` เก็บไว้ก่อนแก้
- ถ้าจะเขียน/ลบ: หาคู่มือรุ่นนั้น ๆ หรือ dump ของเดิมก่อน
- เริ่มจาก “บูตชั่วคราว” (SD/USB/TFTP/fastboot boot) ก่อนเขียนถาวร
- อย่าพยายามแก้คีย์/OTP/eFuse โดยเดา (เสี่ยงถาวร)

จบไฟล์
