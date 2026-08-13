
# Lab6-1-Wi-Fi-SoftAP
## 6. ตารางบันทึกผลการทดลอง (Experiment Results)

### 6.1 บันทึกข้อมูล Client ที่เชื่อมต่อเข้ากับ ESP32 SoftAP

| อุปกรณ์ที่ใช้ทดสอบ (เช่น iPhone/Android) | MAC Address ที่ดักจับได้ | Association ID (AID) | หมายเลข IP Address ที่ได้ (ถ้าทราบ) |
| :--------------------------------------- | :----------------------- | :------------------: | :---------------------------------: |
| **อุปกรณ์ที่ 1**                         | 9A:4B:45:78:F5:A6        |          1           |             192.168.4.2             |
| **อุปกรณ์ที่ 2**                         | 9A:4B:45:78:F5:A6        |          2           |             192.168.4.4             |


---

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)

1. เหตุใด IP Address เริ่มต้นของ ESP32 SoftAP จึงเป็น `192.168.4.1` และ DHCP Server บน ESP32 เริ่มแจกจ่าย IP ที่หมายเลขใด?
ตอบ IP Address เริ่มต้นถูกกำหนดไว้ตามมาตรฐานของ Espressif ESP-IDF (และระบบเครือข่ายส่วนตัว RFC 1918) ให้ใช้ `192.168.4.1` เป็น Gateway/IP หลักของตัว SoftAP และจาก Log (`DHCP server assigned IP to a client, IP is: 192.168.4.2`) แสดงให้เห็นว่า DHCP Server เริ่มแจกจ่าย IP ให้กับ Client ตัวแรกที่หมายเลข **`192.168.4.2`**

2. สมาชิกตัวแปร `mac` ในโครงสร้าง `wifi_event_ap_staconnected_t` สามารถนำไปประยุกต์ใช้ทำระบบความปลอดภัยขั้นสูง (เช่น MAC Filtering) ได้อย่างไร?
ตอบ สมาชิกตัวแปร `mac` จะเก็บค่า Physical Address (MAC Address ขนาด 6 ไบต์) ของอุปกรณ์ Client ที่กำลังพยายามเชื่อมต่อเข้ามาทันทีเมื่อเกิด Event การเชื่อมต่อ เราสามารถนำค่านั้นไปเปรียบเทียบกับรายการรายชื่อที่อนุญาต (Whitelist) หรือรายชื่อที่บล็อก (Blacklist) ในโค้ด C ได้ หาก MAC Address ไม่อยู่ใน Whitelist ระบบสามารถสั่งปฏิเสธหรือเตะ (Deauthentication) Client ออกจากเครือข่ายทันทีเพื่อป้องกันผู้บุกรุก

3. หากมี Client พยายามเชื่อมต่อเป็นเครื่องที่ 5 (เกินค่า `max_connection = 4`) จะเกิดเหตุการณ์ใดขึ้นในระดับสัญญาณวิทยุ?
**ตอบ:** ในระดับสัญญาณวิทยุ (Wi-Fi Layer) เมื่อ SoftAP มีจำนวนสถานี (Stations) ครบตามโควตาสูงสุดที่กำหนดไว้แล้ว (เช่น 4 อุปกรณ์) เมื่ออุปกรณ์เครื่องที่ 5 พยายามส่งคำขอเชื่อมต่อเข้ามา (Association Request) ตัว ESP32 SoftAP จะปฏิเสธคำขอโดยการส่ง Response Frame กลับไปว่าปฏิเสธการเชื่อมต่อ (มักจะใช้ Reason Code เช่น _AP is full_ หรือ _Declined_) และอาจมีการส่งกรอบข้อมูลยกเลิกการเชื่อมต่อ (Deauthentication) ออกไปในอากาศ เพื่อไม่ให้อุปกรณ์เครื่องที่ 5 เข้ามาแย่งทรัพยากรช่องสัญญาณครับ

## Log
```
I (2454) wifi:new:<1,0>, old:<1,1>, ap:<1,0>, sta:<255,255>, prof:1, snd_ch_cfg:0x0
I (2454) wifi:station: e6:b7:bd:05:ec:b0 join, AID=1, bgn, 20
I (2494) LAB_SOFTAP: =======================================================
I (2494) LAB_SOFTAP: [FORENSIC EVENT]: Client Connected to ESP32 SoftAP!
I (2494) LAB_SOFTAP:   -> Client MAC Address : E6:B7:BD:05:EC:B0
I (2504) LAB_SOFTAP:   -> Assigned AID       : 1
I (2504) LAB_SOFTAP: =======================================================
I (2504) wifi:station: 9a:4b:45:78:f5:a6 join, AID=2, bgn, 20
I (2554) esp_netif_lwip: DHCP server assigned IP to a client, IP is: 192.168.4.2
I (2554) LAB_SOFTAP: =======================================================
I (2554) LAB_SOFTAP: [FORENSIC EVENT]: Client Connected to ESP32 SoftAP!
I (2564) LAB_SOFTAP:   -> Client MAC Address : 9A:4B:45:78:F5:A6
I (2564) LAB_SOFTAP:   -> Assigned AID       : 2
I (2574) LAB_SOFTAP: =======================================================
I (2894) wifi:<ba-add>idx:2 (ifx:1, e6:b7:bd:05:ec:b0), tid:0, ssn:173, winSize:64
I (3024) wifi:<ba-add>idx:3 (ifx:1, 9a:4b:45:78:f5:a6), tid:0, ssn:43, winSize:64
I (3614) esp_netif_lwip: DHCP server assigned IP to a client, IP is: 192.168.4.4
I (387414) wifi:<ba-add>idx:4 (ifx:1, 9a:4b:45:78:f5:a6), tid:1, ssn:0, winSize:64
```

# Lab6-2-RSSI-Speed-Profiler
## ตารางบันทึกผลการทดลอง (Experiment Results)

ให้นักศึกษาบันทึกค่าที่ได้จากการทดสอบในระดับ Tx Power ต่างๆ:

| การทดลองที่ | ค่า Tx Power ที่ตั้ง (dBm) | ค่า RSSI ที่อ่านได้จริง (dBm) | เวลาที่ใช้ (Seconds) | ความเร็วที่วัดได้ Throughput (Kbps) |
| :---------: | :------------------------: | :---------------------------: | :------------------: | :---------------------------------: |
|    1    |        20 dBm (Max)        |             -55.6             |        0.124         |               3453.20               |
|    2    |           15 dBm           |             -46.7             |        0.117         |               3630.46               |
|    3    |           10 dBm           |             -62.2             |        0.129         |               3322.71               |
|    4    |           5 dBm            |             -62.6             |        0.113         |               3687.95               |
|    5    |        2 dBm (Min)         |             -57.4             |        0.126         |               3363.37               |

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)

1. เมื่อลดระดับ Tx Power ลงจาก 20 dBm เหลือ 2 dBm ค่า RSSI ลดลงกี่ dBm และส่งผลต่อความเร็ว Throughput อย่างไร?
ตอบ - **ค่า RSSI:** ลดลงตามระดับกำลังส่งที่ลดลง (โดยทั่วไปจะลดลงประมาณ 15 ถึง 30 dBm)
    
- **ผลต่อ Throughput:** ความเร็วในการรับส่งข้อมูลจะลดลงเนื่องจากสัญญาณมีความเข้มลดลงและมีโอกาสเกิดข้อผิดพลาดในการรับส่งข้อมูลสูงขึ้น
- 
 2. เหตุใดในระดับ RSSI ที่อ่อนกว่า `-80 dBm` ความเร็ว Throughput ถึงตกลงอย่างกะทันหันในโปรโตคอล TCP?
ตอบ เนื่องจากสัญญาณที่อ่อนมากทำให้เกิด Packet Loss สูง โปรโตคอล TCP ซึ่งเน้นความถูกต้องของข้อมูลจะมีกลไกการส่งข้อมูลซ้ำ (Retransmission) และลดขนาดหน้าต่างการส่ง (Congestion Window) ลง ทำให้ความเร็วตกลงอย่างรวดเร็ว

3. สมการ Regression ที่ได้จากการทดลองสามารถนำไปประยุกต์ใช้ทำนายคุณภาพการเชื่อมต่อในแอปพลิเคชัน IoT ได้อย่างไร?
ตอบ นำไปใช้ในอุปกรณ์ IoT เพื่อประเมินคุณภาพสัญญาณและความเร็วที่คาดว่าจะได้รับจากค่า RSSI แบบเรียลไทม์ ช่วยให้อุปกรณ์ปรับตัว เช่น ปรับความถี่ในการส่งข้อมูลหรือจัดการพลังงานได้อย่างเหมาะสม


## output log ฝั่ง AP
```
I (27) boot: ESP-IDF v6.0.2 2nd stage bootloader I (27) boot: compile time Aug 11 2026 11:14:17 I (27) boot: Multicore bootloader I (29) boot: chip revision: v3.1 I (32) boot.esp32: SPI Speed : 40MHz I (35) boot.esp32: SPI Mode : DIO I (39) boot.esp32: SPI Flash Size : 2MB I (42) boot: Enabling RNG early entropy source... I (47) boot: Partition Table: I (49) boot: ## Label Usage Type ST Offset Length I (56) boot: 0 nvs WiFi data 01 02 00009000 00006000 I (62) boot: 1 phy_init RF data 01 01 0000f000 00001000 I (69) boot: 2 factory factory app 00 00 00010000 00100000 I (75) boot: End of partition table I (79) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=19e00h (105984) map I (124) esp_image: segment 1: paddr=00029e28 vaddr=3ffb0000 size=04528h ( 17704) load I (131) esp_image: segment 2: paddr=0002e358 vaddr=40080000 size=01cc0h ( 7360) load I (134) esp_image: segment 3: paddr=00030020 vaddr=400d0020 size=8794ch (555340) map I (333) esp_image: segment 4: paddr=000b7974 vaddr=40081cc0 size=13948h ( 80200) load I (366) esp_image: segment 5: paddr=000cb2c4 vaddr=50000000 size=00028h ( 40) load I (377) boot: Loaded app from partition at offset 0x10000 I (377) boot: Disabling RNG early entropy source... I (388) cpu_start: Multicore app I (396) cpu_start: GPIO 3 and 1 are used as console UART I/O pins I (396) cpu_start: Pro cpu start user code I (396) cpu_start: cpu freq: 160000000 Hz I (398) app_init: Application information: I (402) app_init: Project name: rssi_speed_profiler I (407) app_init: App version: 029da6d-dirty I (412) app_init: Compile time: Aug 11 2026 11:13:09 I (417) app_init: ELF file SHA256: d7bd9fdf9... I (421) app_init: ESP-IDF: v6.0.2 I (425) efuse_init: Min chip rev: v0.0 I (429) efuse_init: Max chip rev: v3.99 I (433) efuse_init: Chip rev: v3.1 I (437) heap_init: Initializing. RAM available for dynamic allocation: I (443) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM I (448) heap_init: At 3FFB8A20 len 000275E0 (157 KiB): DRAM I (453) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM I (458) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM I (464) heap_init: At 40095608 len 0000A9F8 (42 KiB): IRAM W (470) spi_flash: Detected boya flash chip but using generic driver. For optimal functionality, enable `SPI_FLASH_SUPPORT_BOYA_CHIP` in menuconfig I (482) spi_flash: detected chip: generic I (486) spi_flash: flash io: dio W (489) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header. I (503) main_task: Started on CPU0 I (503) main_task: Calling app_main() I (563) wifi:wifi driver task: 3ffc04fc, prio:23, stack:6656, core=0 I (583) wifi:wifi firmware version: 00ad238 I (583) wifi:wifi certification version: v7.0 I (583) wifi:config NVS flash: enabled I (583) wifi:config nano formatting: disabled I (583) wifi:Init data frame dynamic rx buffer num: 32 I (593) wifi:Init static rx mgmt buffer num: 5 I (593) wifi:Init management short buffer num: 32 I (603) wifi:Init dynamic tx buffer num: 32 I (603) wifi:Init static rx buffer size: 1600 I (603) wifi:Init static rx buffer num: 10 I (613) wifi:Init dynamic rx buffer num: 32 I (613) wifi_init: rx ba win: 6 I (623) wifi_init: accept mbox: 6 I (623) wifi_init: tcpip mbox: 32 I (623) wifi_init: udp mbox: 6 I (623) wifi_init: tcp mbox: 6 I (633) wifi_init: tcp tx win: 5760 I (633) wifi_init: tcp rx win: 5760 I (633) wifi_init: tcp mss: 1440 I (643) wifi_init: WiFi IRAM OP enabled I (643) wifi_init: WiFi RX IRAM OP enabled I (653) phy_init: phy_version 4863,a3a4459,Oct 28 2025,14:30:06 I (743) wifi:mode : softAP (14:08:08:a4:b6:61) I (753) wifi:Total power save buffer number: 16 I (753) wifi:Init max length of beacon: 752/752 I (753) wifi:Init max length of beacon: 752/752 I (753) AP_SERVER: ESP32 SoftAP Running! SSID: "ESP32_AP_134", Channel: 1 I (753) esp_netif_lwip: DHCP server started on interface WIFI_AP_DEF with IP: 192.168.4.1 I (763) AP_SERVER: [TCP SERVER]: Listening on port 8080 I (773) main_task: Returned from app_main() I (39723) wifi:station: 14:33:5c:0e:94:44 join, AID=1, bgn, 40U I (39753) AP_SERVER: ======================================================= I (39753) AP_SERVER: [FORENSIC EVENT]: Client Connected to ESP32 SoftAP! I (39763) AP_SERVER: -> Client MAC Address : 14:33:5c:0e:94:44 I (39763) AP_SERVER: -> Assigned AID : 1 I (39773) AP_SERVER: ======================================================= I (39793) esp_netif_lwip: DHCP server assigned IP to a client, IP is: 192.168.4.2 I (40823) wifi:<ba-add>idx:2 (ifx:1, 14:33:5c:0e:94:44), tid:0, ssn:0, winSize:64 I (41013) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client I (43203) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client I (45393) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client I (47543) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client I (49753) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client I (51893) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client I (54063) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client I (56213) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client I (58343) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client I (60483) AP_SERVER: [TCP SERVER]: Received total 51200 bytes from client
```

## output log ฝั่ง AP
```
I (29) boot: ESP-IDF v5.5.1 2nd stage bootloader I (29) boot: compile time Aug 11 2026 10:36:15 I (29) boot: Multicore bootloader I (31) boot: chip revision: v3.1 I (33) boot.esp32: SPI Speed : 40MHz I (37) boot.esp32: SPI Mode : DIO I (41) boot.esp32: SPI Flash Size : 2MB I (44) boot: Enabling RNG early entropy source... I (49) boot: Partition Table: I (51) boot: ## Label Usage Type ST Offset Length I (58) boot: 0 nvs WiFi data 01 02 00009000 00006000 I (64) boot: 1 phy_init RF data 01 01 0000f000 00001000 I (71) boot: 2 factory factory app 00 00 00010000 00100000 I (77) boot: End of partition table I (81) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=19a00h (104960) map I (124) esp_image: segment 1: paddr=00029a28 vaddr=3ffb0000 size=03eech ( 16108) load I (130) esp_image: segment 2: paddr=0002d91c vaddr=40080000 size=026fch ( 9980) load I (134) esp_image: segment 3: paddr=00030020 vaddr=400d0020 size=870e8h (553192) map I (324) esp_image: segment 4: paddr=000b7110 vaddr=400826fc size=15710h ( 87824) load I (359) esp_image: segment 5: paddr=000cc828 vaddr=50000000 size=00020h ( 32) load I (371) boot: Loaded app from partition at offset 0x10000 I (371) boot: Disabling RNG early entropy source... I (381) cpu_start: Multicore app I (389) cpu_start: Pro cpu start user code I (390) cpu_start: cpu freq: 160000000 Hz I (390) app_init: Application information: I (390) app_init: Project name: rssi_speed_profiler I (395) app_init: App version: 029da6d-dirty I (399) app_init: Compile time: Aug 11 2026 10:35:01 I (404) app_init: ELF file SHA256: ca6a37d40... I (408) app_init: ESP-IDF: v5.5.1 I (412) efuse_init: Min chip rev: v0.0 I (416) efuse_init: Max chip rev: v3.99 I (420) efuse_init: Chip rev: v3.1 I (424) heap_init: Initializing. RAM available for dynamic allocation: I (430) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM I (435) heap_init: At 3FFB7FD0 len 00028030 (160 KiB): DRAM I (440) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM I (446) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM I (451) heap_init: At 40097E0C len 000081F4 (32 KiB): IRAM I (458) spi_flash: detected chip: generic I (460) spi_flash: flash io: dio W (463) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header. I (476) main_task: Started on CPU0 I (486) main_task: Calling app_main() I (486) CLIENT_PROFILER: [FORENSIC]: Call nvs_flash_init() I (516) CLIENT_PROFILER: [FORENSIC]: Call esp_netif_init() I (516) CLIENT_PROFILER: [FORENSIC]: Call esp_event_loop_create_default() I (516) CLIENT_PROFILER: [FORENSIC]: Call esp_netif_create_default_wifi_sta() I (526) CLIENT_PROFILER: [FORENSIC]: Call esp_wifi_init(&config) I (536) wifi:wifi driver task: 3ffbffa8, prio:23, stack:6656, core=0 I (556) wifi:wifi firmware version: 14da9b7 I (556) wifi:wifi certification version: v7.0 I (556) wifi:config NVS flash: enabled I (556) wifi:config nano formatting: disabled I (566) wifi:Init data frame dynamic rx buffer num: 32 I (566) wifi:Init static rx mgmt buffer num: 5 I (566) wifi:Init management short buffer num: 32 I (576) wifi:Init dynamic tx buffer num: 32 I (576) wifi:Init static rx buffer size: 1600 I (586) wifi:Init static rx buffer num: 10 I (586) wifi:Init dynamic rx buffer num: 32 I (596) wifi_init: rx ba win: 6 I (596) wifi_init: accept mbox: 6 I (596) wifi_init: tcpip mbox: 32 I (596) wifi_init: udp mbox: 6 I (606) wifi_init: tcp mbox: 6 I (606) wifi_init: tcp tx win: 5760 I (606) wifi_init: tcp rx win: 5760 I (606) wifi_init: tcp mss: 1440 I (616) wifi_init: WiFi IRAM OP enabled I (616) wifi_init: WiFi RX IRAM OP enabled I (626) CLIENT_PROFILER: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA) I (636) CLIENT_PROFILER: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config) I (696) CLIENT_PROFILER: [FORENSIC]: Call esp_wifi_start() I (696) phy_init: phy_version 4861,b71b5ad,Aug 5 2025,11:16:06 I (776) wifi:mode : sta (14:33:5c:0e:94:44) I (776) wifi:enable tsf I (776) CLIENT_PROFILER: Client profiler ready: 50 KB x 10 rounds I (776) main_task: Returned from app_main() I (776) CLIENT_PROFILER: [FORENSIC EVENT]: Station started; connecting to ESP32_AP_134 I (1056) wifi:new:<1,1>, old:<1,0>, ap:<255,255>, sta:<1,1>, prof:1, snd_ch_cfg:0x0 I (1066) wifi:state: init -> auth (0xb0) I (1066) wifi:state: auth -> assoc (0x0) I (1086) wifi:state: assoc -> run (0x10) I (1106) wifi:connected with ESP32_AP_134, aid = 1, channel 1, 40U, bssid = 14:08:08:a4:b6:61 I (1106) wifi:security: WPA2-PSK, phy: bgn, rssi: -57 I (1126) wifi:pm start, type: 1 I (1126) wifi:dp: 1, bi: 102400, li: 3, scale listen interval from 307200 us to 307200 us I (1126) wifi:AP's beacon interval = 102400 us, DTIM period = 1 I (2146) esp_netif_handlers: sta ip: 192.168.4.2, mask: 255.255.255.0, gw: 192.168.4.1 I (2146) CLIENT_PROFILER: [FORENSIC EVENT]: Connected; IP=192.168.4.2 I (2146) CLIENT_PROFILER: [ROUND 1/10]: Connecting to 192.168.4.1:8080 I (2156) wifi:<ba-add>idx:0 (ifx:0, 14:08:08:a4:b6:61), tid:0, ssn:0, winSize:64 I (2346) CLIENT_PROFILER: ======================================================= I (2346) CLIENT_PROFILER: [BENCHMARK RESULT 1/10] I (2346) CLIENT_PROFILER: -> Current RSSI : -53 dBm I (2346) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (2356) CLIENT_PROFILER: -> Time Elapsed : 0.169 Seconds I (2366) CLIENT_PROFILER: -> Measured Speed : 2420.07 Kbps I (2366) CLIENT_PROFILER: ======================================================= I (4376) CLIENT_PROFILER: [ROUND 2/10]: Connecting to 192.168.4.1:8080 I (4526) CLIENT_PROFILER: ======================================================= I (4526) CLIENT_PROFILER: [BENCHMARK RESULT 2/10] I (4526) CLIENT_PROFILER: -> Current RSSI : -55 dBm I (4526) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (4536) CLIENT_PROFILER: -> Time Elapsed : 0.140 Seconds I (4546) CLIENT_PROFILER: -> Measured Speed : 2931.33 Kbps I (4546) CLIENT_PROFILER: ======================================================= I (6556) CLIENT_PROFILER: [ROUND 3/10]: Connecting to 192.168.4.1:8080 I (6736) CLIENT_PROFILER: ======================================================= I (6736) CLIENT_PROFILER: [BENCHMARK RESULT 3/10] I (6736) CLIENT_PROFILER: -> Current RSSI : -57 dBm I (6736) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (6746) CLIENT_PROFILER: -> Time Elapsed : 0.168 Seconds I (6746) CLIENT_PROFILER: -> Measured Speed : 2432.43 Kbps I (6756) CLIENT_PROFILER: ======================================================= I (8766) CLIENT_PROFILER: [ROUND 4/10]: Connecting to 192.168.4.1:8080 I (8886) CLIENT_PROFILER: ======================================================= I (8886) CLIENT_PROFILER: [BENCHMARK RESULT 4/10] I (8886) CLIENT_PROFILER: -> Current RSSI : -56 dBm I (8896) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (8896) CLIENT_PROFILER: -> Time Elapsed : 0.110 Seconds I (8906) CLIENT_PROFILER: -> Measured Speed : 3737.40 Kbps I (8916) CLIENT_PROFILER: ======================================================= I (10916) CLIENT_PROFILER: [ROUND 5/10]: Connecting to 192.168.4.1:8080 I (11096) CLIENT_PROFILER: ======================================================= I (11096) CLIENT_PROFILER: [BENCHMARK RESULT 5/10] I (11096) CLIENT_PROFILER: -> Current RSSI : -54 dBm I (11096) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (11106) CLIENT_PROFILER: -> Time Elapsed : 0.107 Seconds I (11106) CLIENT_PROFILER: -> Measured Speed : 3821.64 Kbps I (11116) CLIENT_PROFILER: ======================================================= I (13126) CLIENT_PROFILER: [ROUND 6/10]: Connecting to 192.168.4.1:8080 I (13236) CLIENT_PROFILER: ======================================================= I (13236) CLIENT_PROFILER: [BENCHMARK RESULT 6/10] I (13236) CLIENT_PROFILER: -> Current RSSI : -54 dBm I (13236) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (13246) CLIENT_PROFILER: -> Time Elapsed : 0.099 Seconds I (13246) CLIENT_PROFILER: -> Measured Speed : 4140.89 Kbps I (13256) CLIENT_PROFILER: ======================================================= I (15266) CLIENT_PROFILER: [ROUND 7/10]: Connecting to 192.168.4.1:8080 I (15396) CLIENT_PROFILER: ======================================================= I (15396) CLIENT_PROFILER: [BENCHMARK RESULT 7/10] I (15396) CLIENT_PROFILER: -> Current RSSI : -57 dBm I (15406) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (15406) CLIENT_PROFILER: -> Time Elapsed : 0.127 Seconds I (15416) CLIENT_PROFILER: -> Measured Speed : 3236.74 Kbps I (15416) CLIENT_PROFILER: ======================================================= I (17426) CLIENT_PROFILER: [ROUND 8/10]: Connecting to 192.168.4.1:8080 I (17546) CLIENT_PROFILER: ======================================================= I (17546) CLIENT_PROFILER: [BENCHMARK RESULT 8/10] I (17546) CLIENT_PROFILER: -> Current RSSI : -60 dBm I (17556) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (17556) CLIENT_PROFILER: -> Time Elapsed : 0.115 Seconds I (17566) CLIENT_PROFILER: -> Measured Speed : 3573.24 Kbps I (17566) CLIENT_PROFILER: ======================================================= I (19576) CLIENT_PROFILER: [ROUND 9/10]: Connecting to 192.168.4.1:8080 I (19696) CLIENT_PROFILER: ======================================================= I (19696) CLIENT_PROFILER: [BENCHMARK RESULT 9/10] I (19696) CLIENT_PROFILER: -> Current RSSI : -55 dBm I (19696) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (19706) CLIENT_PROFILER: -> Time Elapsed : 0.108 Seconds I (19706) CLIENT_PROFILER: -> Measured Speed : 3781.81 Kbps I (19716) CLIENT_PROFILER: ======================================================= I (21726) CLIENT_PROFILER: [ROUND 10/10]: Connecting to 192.168.4.1:8080 I (21826) CLIENT_PROFILER: ======================================================= I (21826) CLIENT_PROFILER: [BENCHMARK RESULT 10/10] I (21826) CLIENT_PROFILER: -> Current RSSI : -55 dBm I (21836) CLIENT_PROFILER: -> Total Transferred : 51200 Bytes I (21836) CLIENT_PROFILER: -> Time Elapsed : 0.092 Seconds I (21846) CLIENT_PROFILER: -> Measured Speed : 4456.44 Kbps I (21846) CLIENT_PROFILER: ======================================================= I (23856) CLIENT_PROFILER: All benchmark rounds completed
```

# Lab6-3-FreeRTOS-Sensor-Queue
## 6. ตารางบันทึกผลการทดลอง (Experiment Results)

### 6.1 บันทึกข้อมูล Forensic Stack High Water Mark

  
| ชื่อ FreeRTOS Task | ขนาด Stack ที่กำหนดใน xTaskCreate (Bytes) | ค่า High Water Mark ที่อ่านได้ (Words / Bytes) | สถานะความปลอดภัยสแตก |
| :--- | :---: | :---: | :---: |
| SensorCollectorTask | 3072 | 2028 words / 2028 bytes | ปลอดภัย |
| NetworkCommTask | 4096 | 3080 words / 3080 bytes | ปลอดภัย |

![[graph.jpg]]

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)


1. **เหตุใดการใช้ FreeRTOS Queue จึงมีความปลอดภัย (Thread-Safe) มากกว่าการใช้ตัวแปรแบบ Global ในการรับส่งข้อมูลระหว่างสอง Task?**
    
    - FreeRTOS Queue มีกลไกการจัดการ Critical Section และการล็อกข้อมูล (Mutex/Semaphore ภายใน) ในระดับฮาร์ดแวร์และเคอร์เนล ทำให้ป้องกันปัญหา Race Condition และ Data Corruption ที่อาจเกิดขึ้นจากการที่หลาย Task เข้าถึงและแก้ไขตัวแปร Global พร้อมกัน
        
2. **ค่า Stack High Water Mark มีประโยชน์อย่างไรในการตรวจวินิจฉัยปัญหาบั๊กในระบบเรียลไทม์ (RTOS)?**
    
    - ช่วยให้ทราบพื้นที่ว่างเหลือน้อยที่สุด (Minimum Free Stack Space) ที่ Task เคยใช้งานไปตลอดรันไทม์ ช่วยในการตรวจสอบและป้องกันปัญหา Stack Overflow ล่วงหน้า รวมถึงช่วยในการปรับลดหรือเพิ่มขนาด Stack ของแต่ละ Task ให้มีความเหมาะสม ไม่เปลืองหน่วยความจำ RAM
        
3. **หาก vSensorTask ส่งข้อมูลเร็วมาก (เช่น ทุก 10ms) แต่ vNetworkTask ส่งข้อมูลออก Wi-Fi ได้ช้า (เช่น ใช้เวลา 500ms) จะเกิดอะไรขึ้นกับ Queue และระบบจะรับมืออย่างไร?**
    
    - จะทำให้ Queue เต็มอย่างรวดเร็ว (Queue Full) หากใช้ฟังก์ชันส่งข้อมูลแบบกำหนดเวลาคอย (Timeout) Task ผู้ส่งจะต้องรอจนกว่าจะมีพื้นที่ว่างหรือเกิด Error ขึ้น (กรณีใช้ Block Time เป็น 0 จะทำให้ข้อมูลใหม่ถูกทิ้ง) วิธีรับมือคือการกำหนดขนาด Queue ให้เหมาะสม, เพิ่มกลไก Overwrite ข้อมูลเก่า, หรือปรับปรุงความเร็วในการประมวลผลฝั่งเครือข่าย

## Log
```
I (27) boot: ESP-IDF v6.0.2 2nd stage bootloader
I (27) boot: compile time Aug 12 2026 16:00:41
I (28) boot: Multicore bootloader
I (29) boot: chip revision: v3.1
I (32) boot.esp32: SPI Speed      : 40MHz
I (35) boot.esp32: SPI Mode       : DIO
I (39) boot.esp32: SPI Flash Size : 2MB
I (42) boot: Enabling RNG early entropy source...
I (47) boot: Partition Table:
I (49) boot: ## Label            Usage          Type ST Offset   Length
I (56) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (62) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (69) boot:  2 factory          factory app      00 00 00010000 00100000
I (75) boot: End of partition table
I (79) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=08938h ( 35128) map
I (99) esp_image: segment 1: paddr=00018960 vaddr=3ffb0000 size=029fch ( 10748) load
I (103) esp_image: segment 2: paddr=0001b364 vaddr=40080000 size=04cb4h ( 19636) load
I (111) esp_image: segment 3: paddr=00020020 vaddr=400d0020 size=0c3b0h ( 50096) map
I (130) esp_image: segment 4: paddr=0002c3d8 vaddr=40084cb4 size=05fe4h ( 24548) load
I (140) esp_image: segment 5: paddr=000323c4 vaddr=50000000 size=00028h (    40) load
I (146) boot: Loaded app from partition at offset 0x10000
I (146) boot: Disabling RNG early entropy source...
I (158) cpu_start: Multicore app
I (166) cpu_start: GPIO 3 and 1 are used as console UART I/O pins
I (167) cpu_start: Pro cpu start user code
I (167) cpu_start: cpu freq: 160000000 Hz
I (168) app_init: Application information:
I (172) app_init: Project name:     freertos_sensor_queue
I (177) app_init: App version:      029da6d-dirty
I (182) app_init: Compile time:     Aug 12 2026 15:59:00
I (187) app_init: ELF file SHA256:  9578c46a5...
I (191) app_init: ESP-IDF:          v6.0.2
I (195) efuse_init: Min chip rev:     v0.0
I (199) efuse_init: Max chip rev:     v3.99 
I (203) efuse_init: Chip rev:         v3.1
I (207) heap_init: Initializing. RAM available for dynamic allocation:
I (213) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (218) heap_init: At 3FFB33A8 len 0002CC58 (179 KiB): DRAM
I (223) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (229) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (234) heap_init: At 4008AC98 len 00015368 (84 KiB): IRAM
W (241) spi_flash: Detected boya flash chip but using generic driver. For optimal functionality, enable `SPI_FLASH_SUPPORT_BOYA_CHIP` in menuconfig
I (253) spi_flash: detected chip: generic
I (256) spi_flash: flash io: dio
W (259) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (272) main_task: Started on CPU0
I (272) main_task: Calling app_main()
I (272) LAB_FREERTOS_QUEUE: ==================================================================
I (282) LAB_FREERTOS_QUEUE:   Lab 6.3: FreeRTOS Multi-Tasking & Sensor Data Queue Fusion
I (292) LAB_FREERTOS_QUEUE: ==================================================================
I (292) LAB_FREERTOS_QUEUE: [TASK CREATED]: Sensor Collector Task Started on Core 0
I (302) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 26.2 C, Hum: 54.9 %, Lux: 413
I (312) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (322) LAB_FREERTOS_QUEUE: [TASK CREATED]: Network Task Started on Core 0
I (322) LAB_FREERTOS_QUEUE: =======================================================
I (332) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (342) LAB_FREERTOS_QUEUE:   -> Timestamp   : 30 ms
I (342) LAB_FREERTOS_QUEUE:   -> Temperature : 26.20 degC
I (352) LAB_FREERTOS_QUEUE:   -> Humidity    : 54.90 %
I (352) LAB_FREERTOS_QUEUE:   -> Light Lux   : 413 lux
I (362) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (372) LAB_FREERTOS_QUEUE: =======================================================
I (372) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (382) main_task: Returned from app_main()
I (1822) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 28.4 C, Hum: 51.0 %, Lux: 543
I (1822) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (1822) LAB_FREERTOS_QUEUE: =======================================================
I (1832) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (1832) LAB_FREERTOS_QUEUE:   -> Timestamp   : 1550 ms
I (1842) LAB_FREERTOS_QUEUE:   -> Temperature : 28.40 degC
I (1842) LAB_FREERTOS_QUEUE:   -> Humidity    : 51.00 %
I (1852) LAB_FREERTOS_QUEUE:   -> Light Lux   : 543 lux
I (1852) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (1862) LAB_FREERTOS_QUEUE: =======================================================
I (1872) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (3322) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 34.9 C, Hum: 62.5 %, Lux: 695
I (3322) LAB_FREERTOS_QUEUE: =======================================================
I (3322) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (3322) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (3332) LAB_FREERTOS_QUEUE:   -> Timestamp   : 3050 ms
I (3342) LAB_FREERTOS_QUEUE:   -> Temperature : 34.90 degC
I (3342) LAB_FREERTOS_QUEUE:   -> Humidity    : 62.50 %
I (3352) LAB_FREERTOS_QUEUE:   -> Light Lux   : 695 lux
I (3352) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (3362) LAB_FREERTOS_QUEUE: =======================================================
I (3372) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (4832) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 26.0 C, Hum: 68.3 %, Lux: 665
I (4832) LAB_FREERTOS_QUEUE: =======================================================
I (4832) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (4832) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (4842) LAB_FREERTOS_QUEUE:   -> Timestamp   : 4560 ms
I (4852) LAB_FREERTOS_QUEUE:   -> Temperature : 26.00 degC
I (4852) LAB_FREERTOS_QUEUE:   -> Humidity    : 68.30 %
I (4862) LAB_FREERTOS_QUEUE:   -> Light Lux   : 665 lux
I (4862) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (4872) LAB_FREERTOS_QUEUE: =======================================================
I (4882) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (6342) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 31.5 C, Hum: 66.1 %, Lux: 280
I (6342) LAB_FREERTOS_QUEUE: =======================================================
I (6342) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (6342) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (6352) LAB_FREERTOS_QUEUE:   -> Timestamp   : 6070 ms
I (6362) LAB_FREERTOS_QUEUE:   -> Temperature : 31.50 degC
I (6362) LAB_FREERTOS_QUEUE:   -> Humidity    : 66.10 %
I (6372) LAB_FREERTOS_QUEUE:   -> Light Lux   : 280 lux
I (6372) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (6382) LAB_FREERTOS_QUEUE: =======================================================
I (6392) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (7852) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 26.8 C, Hum: 63.1 %, Lux: 531
I (7852) LAB_FREERTOS_QUEUE: =======================================================
I (7852) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (7852) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (7862) LAB_FREERTOS_QUEUE:   -> Timestamp   : 7580 ms
I (7872) LAB_FREERTOS_QUEUE:   -> Temperature : 26.80 degC
I (7872) LAB_FREERTOS_QUEUE:   -> Humidity    : 63.10 %
I (7882) LAB_FREERTOS_QUEUE:   -> Light Lux   : 531 lux
I (7882) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (7892) LAB_FREERTOS_QUEUE: =======================================================
I (7902) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (9362) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 25.6 C, Hum: 52.5 %, Lux: 331
I (9362) LAB_FREERTOS_QUEUE: =======================================================
I (9362) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (9362) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (9372) LAB_FREERTOS_QUEUE:   -> Timestamp   : 9090 ms
I (9382) LAB_FREERTOS_QUEUE:   -> Temperature : 25.60 degC
I (9382) LAB_FREERTOS_QUEUE:   -> Humidity    : 52.50 %
I (9392) LAB_FREERTOS_QUEUE:   -> Light Lux   : 331 lux
I (9392) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (9402) LAB_FREERTOS_QUEUE: =======================================================
I (9412) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (10872) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 30.2 C, Hum: 51.3 %, Lux: 251
I (10872) LAB_FREERTOS_QUEUE: =======================================================
I (10872) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (10872) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (10882) LAB_FREERTOS_QUEUE:   -> Timestamp   : 10600 ms
I (10892) LAB_FREERTOS_QUEUE:   -> Temperature : 30.20 degC
I (10892) LAB_FREERTOS_QUEUE:   -> Humidity    : 51.30 %
I (10902) LAB_FREERTOS_QUEUE:   -> Light Lux   : 251 lux
I (10902) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (10912) LAB_FREERTOS_QUEUE: =======================================================
I (10922) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (12382) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 33.3 C, Hum: 67.2 %, Lux: 345
I (12382) LAB_FREERTOS_QUEUE: =======================================================
I (12382) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (12382) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (12392) LAB_FREERTOS_QUEUE:   -> Timestamp   : 12110 ms
I (12402) LAB_FREERTOS_QUEUE:   -> Temperature : 33.30 degC
I (12402) LAB_FREERTOS_QUEUE:   -> Humidity    : 67.20 %
I (12412) LAB_FREERTOS_QUEUE:   -> Light Lux   : 345 lux
I (12412) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (12422) LAB_FREERTOS_QUEUE: =======================================================
I (12432) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (13892) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 30.7 C, Hum: 52.0 %, Lux: 228
I (13892) LAB_FREERTOS_QUEUE: =======================================================
I (13892) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (13892) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (13902) LAB_FREERTOS_QUEUE:   -> Timestamp   : 13620 ms
I (13912) LAB_FREERTOS_QUEUE:   -> Temperature : 30.70 degC
I (13912) LAB_FREERTOS_QUEUE:   -> Humidity    : 52.00 %
I (13922) LAB_FREERTOS_QUEUE:   -> Light Lux   : 228 lux
I (13922) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (13932) LAB_FREERTOS_QUEUE: =======================================================
I (13942) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (15402) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 33.9 C, Hum: 64.5 %, Lux: 461
I (15402) LAB_FREERTOS_QUEUE: =======================================================
I (15402) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (15402) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (15412) LAB_FREERTOS_QUEUE:   -> Timestamp   : 15130 ms
I (15422) LAB_FREERTOS_QUEUE:   -> Temperature : 33.90 degC
I (15422) LAB_FREERTOS_QUEUE:   -> Humidity    : 64.50 %
I (15432) LAB_FREERTOS_QUEUE:   -> Light Lux   : 461 lux
I (15432) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (15442) LAB_FREERTOS_QUEUE: =======================================================
I (15452) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (16912) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 29.6 C, Hum: 53.5 %, Lux: 676
I (16912) LAB_FREERTOS_QUEUE: =======================================================
I (16912) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (16912) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (16922) LAB_FREERTOS_QUEUE:   -> Timestamp   : 16640 ms
I (16932) LAB_FREERTOS_QUEUE:   -> Temperature : 29.60 degC
I (16932) LAB_FREERTOS_QUEUE:   -> Humidity    : 53.50 %
I (16942) LAB_FREERTOS_QUEUE:   -> Light Lux   : 676 lux
I (16942) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (16952) LAB_FREERTOS_QUEUE: =======================================================
I (16962) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (18422) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 31.4 C, Hum: 68.0 %, Lux: 607
I (18422) LAB_FREERTOS_QUEUE: =======================================================
I (18422) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (18422) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (18432) LAB_FREERTOS_QUEUE:   -> Timestamp   : 18150 ms
I (18442) LAB_FREERTOS_QUEUE:   -> Temperature : 31.40 degC
I (18442) LAB_FREERTOS_QUEUE:   -> Humidity    : 68.00 %
I (18452) LAB_FREERTOS_QUEUE:   -> Light Lux   : 607 lux
I (18452) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (18462) LAB_FREERTOS_QUEUE: =======================================================
I (18472) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (19932) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 27.9 C, Hum: 61.7 %, Lux: 635
I (19932) LAB_FREERTOS_QUEUE: =======================================================
I (19932) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (19932) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (19942) LAB_FREERTOS_QUEUE:   -> Timestamp   : 19660 ms
I (19952) LAB_FREERTOS_QUEUE:   -> Temperature : 27.90 degC
I (19952) LAB_FREERTOS_QUEUE:   -> Humidity    : 61.70 %
I (19962) LAB_FREERTOS_QUEUE:   -> Light Lux   : 635 lux
I (19962) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (19972) LAB_FREERTOS_QUEUE: =======================================================
I (19982) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (21442) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 34.5 C, Hum: 50.2 %, Lux: 549
I (21442) LAB_FREERTOS_QUEUE: =======================================================
I (21442) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (21442) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (21452) LAB_FREERTOS_QUEUE:   -> Timestamp   : 21170 ms
I (21462) LAB_FREERTOS_QUEUE:   -> Temperature : 34.50 degC
I (21462) LAB_FREERTOS_QUEUE:   -> Humidity    : 50.20 %
I (21472) LAB_FREERTOS_QUEUE:   -> Light Lux   : 549 lux
I (21472) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (21482) LAB_FREERTOS_QUEUE: =======================================================
I (21492) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (22952) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 31.6 C, Hum: 57.7 %, Lux: 210
I (22952) LAB_FREERTOS_QUEUE: =======================================================
I (22952) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (22952) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (22962) LAB_FREERTOS_QUEUE:   -> Timestamp   : 22680 ms
I (22972) LAB_FREERTOS_QUEUE:   -> Temperature : 31.60 degC
I (22972) LAB_FREERTOS_QUEUE:   -> Humidity    : 57.70 %
I (22982) LAB_FREERTOS_QUEUE:   -> Light Lux   : 210 lux
I (22982) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (22992) LAB_FREERTOS_QUEUE: =======================================================
I (23002) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (24462) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 33.7 C, Hum: 59.6 %, Lux: 431
I (24462) LAB_FREERTOS_QUEUE: =======================================================
I (24462) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (24462) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (24472) LAB_FREERTOS_QUEUE:   -> Timestamp   : 24190 ms
I (24482) LAB_FREERTOS_QUEUE:   -> Temperature : 33.70 degC
I (24482) LAB_FREERTOS_QUEUE:   -> Humidity    : 59.60 %
I (24492) LAB_FREERTOS_QUEUE:   -> Light Lux   : 431 lux
I (24492) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (24502) LAB_FREERTOS_QUEUE: =======================================================
I (24512) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (25972) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 26.3 C, Hum: 53.4 %, Lux: 379
I (25972) LAB_FREERTOS_QUEUE: =======================================================
I (25972) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (25972) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (25982) LAB_FREERTOS_QUEUE:   -> Timestamp   : 25700 ms
I (25992) LAB_FREERTOS_QUEUE:   -> Temperature : 26.30 degC
I (25992) LAB_FREERTOS_QUEUE:   -> Humidity    : 53.40 %
I (26002) LAB_FREERTOS_QUEUE:   -> Light Lux   : 379 lux
I (26002) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (26012) LAB_FREERTOS_QUEUE: =======================================================
I (26022) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3080 words (3080 bytes)
I (27482) LAB_FREERTOS_QUEUE: [SENSOR TASK]: Pushing Data -> Temp: 26.3 C, Hum: 56.6 %, Lux: 602
I (27482) LAB_FREERTOS_QUEUE: =======================================================
I (27482) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (27482) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Data Received from Queue!
I (27492) LAB_FREERTOS_QUEUE:   -> Timestamp   : 27210 ms
I (27502) LAB_FREERTOS_QUEUE:   -> Temperature : 26.30 degC
I (27502) LAB_FREERTOS_QUEUE:   -> Humidity    : 56.60 %
I (27512) LAB_FREERTOS_QUEUE:   -> Light Lux   : 602 lux
I (27512) LAB_FREERTOS_QUEUE: [NETWORK TASK]: Preparing JSON Packet for Wi-Fi Transmission...
I (27522) LAB_FREERTOS_QUEUE: =======================================================
```

# Lab6-4-IoT-Sensor-Dashboard
## 7. ตารางบันทึกผลการทดลอง (Experiment Results)

### 7.1 บันทึกข้อมูลจาก Dashboard

| ครั้งที่ | Temperature (°C) | Humidity (%) | Light Lux | Timestamp (ms) |
| :------: | :--------------: | :----------: | :-------: | :------------: |
|  **1**   |      29.50       |    67.50     |    616    |      3460      |
|  **2**   |      33.00       |    67.40     |    529    |      6480      |
|  **3**   |      33.60       |    63.40     |    295    |      7990      |

![[ESP32 Sensor.png]]
### 7.2 ทดสอบ JSON API (`/api/data`)

บันทึก Raw JSON Response จาก Browser:

```json
{"temperature":28.80,"humidity":62.40,"light_lux":256,"timestamp_ms":59380}
```

---

## 8. คำถามท้ายการทดลอง (Post-Lab Questions)

- **เหตุใดจึงต้องใช้ Mutex ในการป้องกันการเข้าถึงตัวแปร g_latest_data ร่วมกันระหว่าง vNetworkTask และ HTTP Handler? ถ้าไม่ใช้จะเกิดอะไรขึ้น?**
    
    - **เหตุผล:** เพื่อป้องกันปัญหา **Race Condition** เนื่องจากตัวแปร `g_latest_data` ถูกเขียนค่าโดย `vNetworkTask` และถูกอ่านค่าออกไปแสดงผลโดย `HTTP Handler` ซึ่งทำงานคนละเธรด/คอร์ หากไม่มีการล็อก (Mutex) อาจทำให้ข้อมูลระหว่างอ่านและเขียนทับซ้อนกันครึ่งๆ กลางๆ (Data Corruption)
        
    - **หากไม่ใช้:** ข้อมูล JSON ที่ส่งให้ผู้ใช้บนเว็บอาจแสดงค่าอุณหภูมิจากรอบเก่าผสมกับความชื้นจากรอบใหม่ที่ไม่ตรงกัน (Inconsistent Data)
        
- **esp_http_server รัน Handler บน Thread ใด — เป็น Thread เดียวกับ FreeRTOS Task ของเราหรือไม่?**
    
    - ระบบ `esp_http_server` จะสร้าง FreeRTOS Task แยกต่างหากสำหรับจัดการคำขอ HTTP (HTTP Server Daemon / Worker Threads) ซึ่งเป็นคนละ Task กับ `vSensorTask` และ `vNetworkTask` ของเรา
        
- **การที่ Dashboard ใช้ `<meta http-equiv="refresh" content="2">` แทนที่จะใช้ JavaScript `fetch()` มีข้อดีและข้อเสียอย่างไร?**
    
    - **ข้อดี:** โค้ดเขียนง่ายมากๆ ไม่ต้องใช้ JavaScript สลับซับซ้อน โหลดหน้าเว็บได้เสถียรแม้ในเบราว์เซอร์รุ่นเก่าหรืออุปกรณ์ IoT ที่มีทรัพยากรจำกัด
        
    - **ข้อเสีย:** หน้าเว็บจะทำการ Refresh (รีโหลดใหม่ทั้งหน้าจอ) ทุกๆ 2 วินาที ทำให้หน้าจอกระพริบ (Flicker) และเปลืองแบนด์วิดท์ในการดาวน์โหลดโครงสร้าง HTML ซ้ำๆ ทุกครั้ง ต่างจากการใช้ `fetch()` (AJAX) ที่จะอัปเดตเฉพาะตัวเลขข้อมูล (JSON) แบบเบื้องหลังโดยที่หน้าเว็บไม่กระพริบ
    
## Log
```
I (27) boot: ESP-IDF v6.0.2 2nd stage bootloader
I (27) boot: compile time Aug 12 2026 17:00:19
I (28) boot: Multicore bootloader
I (29) boot: chip revision: v3.1
I (32) boot.esp32: SPI Speed      : 40MHz
I (35) boot.esp32: SPI Mode       : DIO
I (39) boot.esp32: SPI Flash Size : 2MB
I (42) boot: Enabling RNG early entropy source...
I (47) boot: Partition Table:
I (49) boot: ## Label            Usage          Type ST Offset   Length
I (56) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (62) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (69) boot:  2 factory          factory app      00 00 00010000 00100000
I (75) boot: End of partition table
I (79) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=1d5a8h (120232) map
I (129) esp_image: segment 1: paddr=0002d5d0 vaddr=3ffb0000 size=02a48h ( 10824) load
I (134) esp_image: segment 2: paddr=00030020 vaddr=400d0020 size=8ef48h (585544) map
I (342) esp_image: segment 3: paddr=000bef70 vaddr=3ffb2a48 size=01ae0h (  6880) load
I (345) esp_image: segment 4: paddr=000c0a58 vaddr=40080000 size=1562ch ( 87596) load
I (382) esp_image: segment 5: paddr=000d608c vaddr=50000000 size=00028h (    40) load
I (393) boot: Loaded app from partition at offset 0x10000
I (393) boot: Disabling RNG early entropy source...
I (404) cpu_start: Multicore app
I (412) cpu_start: GPIO 3 and 1 are used as console UART I/O pins
I (412) cpu_start: Pro cpu start user code
I (412) cpu_start: cpu freq: 160000000 Hz
I (414) app_init: Application information:
I (418) app_init: Project name:     iot_sensor_dashboard
I (423) app_init: App version:      029da6d-dirty
I (428) app_init: Compile time:     Aug 12 2026 16:57:56
I (433) app_init: ELF file SHA256:  c2f1046a2...
I (437) app_init: ESP-IDF:          v6.0.2
I (441) efuse_init: Min chip rev:     v0.0
I (445) efuse_init: Max chip rev:     v3.99 
I (449) efuse_init: Chip rev:         v3.1
I (453) heap_init: Initializing. RAM available for dynamic allocation:
I (459) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (464) heap_init: At 3FFB8A38 len 000275C8 (157 KiB): DRAM
I (469) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (474) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (480) heap_init: At 4009562C len 0000A9D4 (42 KiB): IRAM
W (486) spi_flash: Detected boya flash chip but using generic driver. For optimal functionality, enable `SPI_FLASH_SUPPORT_BOYA_CHIP` in menuconfig
I (498) spi_flash: detected chip: generic
I (502) spi_flash: flash io: dio
W (505) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (519) main_task: Started on CPU0
I (519) main_task: Calling app_main()
I (519) MAIN: [FORENSIC]: Call nvs_flash_init()
I (569) MAIN: =======================================================
I (569) MAIN:   Lab 6-4: IoT Sensor Dashboard
I (569) MAIN:   SoftAP + FreeRTOS Queue + HTTP Server
I (569) MAIN: =======================================================
I (579) MAIN: [FORENSIC]: Call xSemaphoreCreateMutex()
I (579) MAIN: [FORENSIC]: Mutex created at 0x3ffbbefc
I (589) MAIN: [FORENSIC]: Call xQueueCreate(10, sizeof(sensor_data_t))
I (589) MAIN: [FORENSIC]: Queue created at 0x3ffbbf54
I (599) SOFTAP: [FORENSIC]: Call esp_netif_init()
I (599) SOFTAP: [FORENSIC]: Call esp_event_loop_create_default()
I (609) SOFTAP: [FORENSIC]: Call esp_netif_create_default_wifi_ap()
I (609) SOFTAP: [FORENSIC]: SoftAP netif created at 0x3ffbdee4 (IP: 192.168.4.1)
I (619) SOFTAP: [FORENSIC]: Call esp_wifi_init()
I (639) wifi:wifi driver task: 3ffc0668, prio:23, stack:6656, core=0
I (669) wifi:wifi firmware version: 00ad238
I (669) wifi:wifi certification version: v7.0
I (669) wifi:config NVS flash: enabled
I (669) wifi:config nano formatting: disabled
I (669) wifi:Init data frame dynamic rx buffer num: 32
I (679) wifi:Init static rx mgmt buffer num: 5
I (679) wifi:Init management short buffer num: 32
I (679) wifi:Init dynamic tx buffer num: 32
I (689) wifi:Init static rx buffer size: 1600
I (689) wifi:Init static rx buffer num: 10
I (699) wifi:Init dynamic rx buffer num: 32
I (699) wifi_init: rx ba win: 6
I (699) wifi_init: accept mbox: 6
I (709) wifi_init: tcpip mbox: 32
I (709) wifi_init: udp mbox: 6
I (709) wifi_init: tcp mbox: 6
I (709) wifi_init: tcp tx win: 5760
I (719) wifi_init: tcp rx win: 5760
I (719) wifi_init: tcp mss: 1440
I (719) wifi_init: WiFi IRAM OP enabled
I (729) wifi_init: WiFi RX IRAM OP enabled
I (729) SOFTAP: [FORENSIC]: Call esp_event_handler_instance_register(WIFI_EVENT)
I (739) SOFTAP: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_AP)
I (749) SOFTAP: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_AP)
I (759) SOFTAP: [FORENSIC]: Call esp_wifi_start()
I (759) phy_init: phy_version 4863,a3a4459,Oct 28 2025,14:30:06
I (839) phy_init: Saving new calibration data due to checksum failure or outdated calibration data, mode(0)
I (859) wifi:mode : softAP (14:08:08:a4:b6:61)
I (869) wifi:Total power save buffer number: 16
I (869) wifi:Init max length of beacon: 752/752
I (869) wifi:Init max length of beacon: 752/752
I (869) SOFTAP: =======================================================
I (869) esp_netif_lwip: DHCP server started on interface WIFI_AP_DEF with IP: 192.168.4.1
I (879) SOFTAP:   SoftAP Running! SSID: "MY_ESP32_SENSOR_AP", Channel: 1
I (889) SOFTAP:   -> Connect your phone to: MY_ESP32_SENSOR_AP
I (899) SOFTAP:   -> Then open browser:      http://192.168.4.1
I (899) SOFTAP: =======================================================
I (909) HTTP_SERVER: [FORENSIC]: Call httpd_start()
I (919) HTTP_SERVER: =======================================================
I (919) HTTP_SERVER: [HTTP SERVER]: Started successfully
I (919) HTTP_SERVER:   -> Dashboard : http://192.168.4.1/
I (929) HTTP_SERVER:   -> JSON API  : http://192.168.4.1/api/data
I (929) HTTP_SERVER: =======================================================
I (939) MAIN: [FORENSIC]: Call xTaskCreate(vSensorTask)  Stack=3072
I (949) SENSOR_TASK: [FORENSIC]: Sensor Collector Task started on Core 0
I (949) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 29.2 C, Hum: 51.3 %, Lux: 381
I (959) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (969) MAIN: [FORENSIC]: Call xTaskCreate(vNetworkTask) Stack=4096
I (969) NETWORK_TASK: [FORENSIC]: Network Task started on Core 0
W (979) wifi:I (979) NETWORK_TASK: =======================================================
no need to send deauth when softap is sending deauth
I (989) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (979) MAIN: =======================================================
I (999) NETWORK_TASK:   -> Timestamp   : 430 ms
I (1009) NETWORK_TASK:   -> Temperature : 29.20 degC
I (1009) NETWORK_TASK:   -> Humidity    : 51.30 %
I (1019) NETWORK_TASK:   -> Light Lux   : 381 lux
I (1019) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (1029) NETWORK_TASK: =======================================================
I (1039) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (1039) MAIN:   System Ready! Open browser at http://192.168.4.1
I (1049) MAIN: =======================================================
I (1049) main_task: Returned from app_main()
I (2469) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 33.5 C, Hum: 50.3 %, Lux: 604
I (2469) NETWORK_TASK: =======================================================
I (2469) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (2469) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (2479) NETWORK_TASK:   -> Timestamp   : 1950 ms
I (2489) NETWORK_TASK:   -> Temperature : 33.50 degC
I (2489) NETWORK_TASK:   -> Humidity    : 50.30 %
I (2499) NETWORK_TASK:   -> Light Lux   : 604 lux
I (2499) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (2509) NETWORK_TASK: =======================================================
I (2509) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
W (2889) wifi:no need to send deauth when softap is sending deauth
W (2899) wifi:no need to send deauth when softap is sending deauth
I (3029) wifi:new:<1,0>, old:<1,1>, ap:<1,0>, sta:<255,255>, prof:1, snd_ch_cfg:0x0
I (3029) wifi:station: a6:78:5c:8f:02:26 join, AID=1, bgn, 20
I (3069) SOFTAP: =======================================================
I (3069) SOFTAP: [FORENSIC EVENT]: Client Connected to SoftAP!
I (3069) SOFTAP:   -> Client MAC : A6:78:5C:8F:02:26
I (3079) SOFTAP:   -> AID        : 1
I (3079) SOFTAP: =======================================================
I (3079) wifi:<ba-add>idx:2 (ifx:1, a6:78:5c:8f:02:26), tid:0, ssn:602, winSize:64
I (3109) esp_netif_lwip: DHCP server assigned IP to a client, IP is: 192.168.4.2
I (3979) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 29.5 C, Hum: 67.5 %, Lux: 616
I (3979) NETWORK_TASK: =======================================================
I (3979) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (3979) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (3989) NETWORK_TASK:   -> Timestamp   : 3460 ms
I (3999) NETWORK_TASK:   -> Temperature : 29.50 degC
I (3999) NETWORK_TASK:   -> Humidity    : 67.50 %
I (4009) NETWORK_TASK:   -> Light Lux   : 616 lux
I (4009) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (4019) NETWORK_TASK: =======================================================
I (4019) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (4529) HTTP_SERVER: [FORENSIC]: GET / (Dashboard) requested
I (4539) HTTP_SERVER: [FORENSIC]: HTML Dashboard sent (2730 bytes)
I (4569) HTTP_SERVER: [FORENSIC]: GET /api/data requested
I (4569) HTTP_SERVER: [FORENSIC]: Mutex acquired & released for /api/data
I (4579) HTTP_SERVER: [FORENSIC]: Response sent: {"temperature":29.50,"humidity":67.50,"light_lux":616,"timestamp_ms":3460}
I (5089) HTTP_SERVER: [FORENSIC]: GET / (Dashboard) requested
I (5099) HTTP_SERVER: [FORENSIC]: HTML Dashboard sent (2730 bytes)
I (5109) HTTP_SERVER: [FORENSIC]: GET /api/data requested
I (5109) HTTP_SERVER: [FORENSIC]: Mutex acquired & released for /api/data
I (5119) HTTP_SERVER: [FORENSIC]: Response sent: {"temperature":29.50,"humidity":67.50,"light_lux":616,"timestamp_ms":3460}
I (5489) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 26.9 C, Hum: 61.4 %, Lux: 482
I (5489) NETWORK_TASK: =======================================================
I (5489) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (5489) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (5499) NETWORK_TASK:   -> Timestamp   : 4970 ms
I (5509) NETWORK_TASK:   -> Temperature : 26.90 degC
I (5509) NETWORK_TASK:   -> Humidity    : 61.40 %
I (5519) NETWORK_TASK:   -> Light Lux   : 482 lux
I (5519) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (5529) NETWORK_TASK: =======================================================
I (5529) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (6999) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 33.0 C, Hum: 67.4 %, Lux: 529
I (6999) NETWORK_TASK: =======================================================
I (6999) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (6999) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (7009) NETWORK_TASK:   -> Timestamp   : 6480 ms
I (7019) NETWORK_TASK:   -> Temperature : 33.00 degC
I (7019) NETWORK_TASK:   -> Humidity    : 67.40 %
I (7029) NETWORK_TASK:   -> Light Lux   : 529 lux
I (7029) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (7039) NETWORK_TASK: =======================================================
I (7039) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (7129) HTTP_SERVER: [FORENSIC]: GET /api/data requested
I (7129) HTTP_SERVER: [FORENSIC]: Mutex acquired & released for /api/data
I (7139) HTTP_SERVER: [FORENSIC]: Response sent: {"temperature":33.00,"humidity":67.40,"light_lux":529,"timestamp_ms":6480}
I (8509) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 33.6 C, Hum: 63.4 %, Lux: 295
I (8509) NETWORK_TASK: =======================================================
I (8509) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (8509) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (8519) NETWORK_TASK:   -> Timestamp   : 7990 ms
I (8529) NETWORK_TASK:   -> Temperature : 33.60 degC
I (8529) NETWORK_TASK:   -> Humidity    : 63.40 %
I (8539) NETWORK_TASK:   -> Light Lux   : 295 lux
I (8539) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (8549) NETWORK_TASK: =======================================================
I (8549) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (9119) HTTP_SERVER: [FORENSIC]: GET /api/data requested
I (9119) HTTP_SERVER: [FORENSIC]: Mutex acquired & released for /api/data
I (9129) HTTP_SERVER: [FORENSIC]: Response sent: {"temperature":33.60,"humidity":63.40,"light_lux":295,"timestamp_ms":7990}
I (10019) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 28.1 C, Hum: 56.7 %, Lux: 527
I (10019) NETWORK_TASK: =======================================================
I (10019) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (10019) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (10029) NETWORK_TASK:   -> Timestamp   : 9500 ms
I (10039) NETWORK_TASK:   -> Temperature : 28.10 degC
I (10039) NETWORK_TASK:   -> Humidity    : 56.70 %
I (10049) NETWORK_TASK:   -> Light Lux   : 527 lux
I (10049) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (10059) NETWORK_TASK: =======================================================
I (10059) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (11119) HTTP_SERVER: [FORENSIC]: GET /api/data requested
I (11119) HTTP_SERVER: [FORENSIC]: Mutex acquired & released for /api/data
I (11129) HTTP_SERVER: [FORENSIC]: Response sent: {"temperature":28.10,"humidity":56.70,"light_lux":527,"timestamp_ms":9500}
I (11529) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 31.1 C, Hum: 55.5 %, Lux: 610
I (11529) NETWORK_TASK: =======================================================
I (11529) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (11529) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (11539) NETWORK_TASK:   -> Timestamp   : 11010 ms
I (11549) NETWORK_TASK:   -> Temperature : 31.10 degC
I (11549) NETWORK_TASK:   -> Humidity    : 55.50 %
I (11559) NETWORK_TASK:   -> Light Lux   : 610 lux
I (11559) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (11569) NETWORK_TASK: =======================================================
I (11569) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (13039) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 31.5 C, Hum: 68.4 %, Lux: 694
I (13039) NETWORK_TASK: =======================================================
I (13039) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (13039) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (13049) NETWORK_TASK:   -> Timestamp   : 12520 ms
I (13059) NETWORK_TASK:   -> Temperature : 31.50 degC
I (13059) NETWORK_TASK:   -> Humidity    : 68.40 %
I (13069) NETWORK_TASK:   -> Light Lux   : 694 lux
I (13069) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (13079) NETWORK_TASK: =======================================================
I (13079) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (13119) HTTP_SERVER: [FORENSIC]: GET /api/data requested
I (13119) HTTP_SERVER: [FORENSIC]: Mutex acquired & released for /api/data
I (13129) HTTP_SERVER: [FORENSIC]: Response sent: {"temperature":31.50,"humidity":68.40,"light_lux":694,"timestamp_ms":12520}
I (14549) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 30.0 C, Hum: 54.3 %, Lux: 202
I (14549) NETWORK_TASK: =======================================================
I (14549) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (14549) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (14559) NETWORK_TASK:   -> Timestamp   : 14030 ms
I (14569) NETWORK_TASK:   -> Temperature : 30.00 degC
I (14569) NETWORK_TASK:   -> Humidity    : 54.30 %
I (14579) NETWORK_TASK:   -> Light Lux   : 202 lux
I (14579) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (14589) NETWORK_TASK: =======================================================
I (14589) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (15129) HTTP_SERVER: [FORENSIC]: GET /api/data requested
I (15129) HTTP_SERVER: [FORENSIC]: Mutex acquired & released for /api/data
I (15139) HTTP_SERVER: [FORENSIC]: Response sent: {"temperature":30.00,"humidity":54.30,"light_lux":202,"timestamp_ms":14030}
I (16059) SENSOR_TASK: [SENSOR TASK]: Pushing -> Temp: 25.5 C, Hum: 53.1 %, Lux: 347
I (16059) NETWORK_TASK: =======================================================
I (16059) FORENSIC_STACK:   -> SensorTask Stack Remaining: 2028 words (2028 bytes)
I (16059) NETWORK_TASK: [NETWORK TASK]: Data Received from Queue!
I (16069) NETWORK_TASK:   -> Timestamp   : 15540 ms
I (16079) NETWORK_TASK:   -> Temperature : 25.50 degC
I (16079) NETWORK_TASK:   -> Humidity    : 53.10 %
I (16089) NETWORK_TASK:   -> Light Lux   : 347 lux
I (16089) NETWORK_TASK: [NETWORK TASK]: g_latest_data updated (Mutex OK)
I (16099) NETWORK_TASK: =======================================================
I (16099) FORENSIC_STACK:   -> NetworkTask Stack Remaining: 3072 words (3072 bytes)
I (17149) HTTP_SERVER: [FORENSIC]: GET /api/data requested
I (17149) HTTP_SERVER: [FORENSIC]: Mutex acquired & released for /api/data
I (17159) HTTP_SERVER: [FORENSIC]: Response sent: {"temperature":25.50,"humidity":53.10,"light_lux":347,"timestamp_ms":15540}
```

# Lab6-5-Proximity-Attendance
## 5. ตารางบันทึกผลการทดลอง (Experiment Results)

### 5.1 ตารางบันทึกการเช็กชื่อผ่าน RF Proximity

| ลำดับที่ | ชื่อสมาร์ตโฟน / MAC Address | ระดับ RSSI (dBm) | ระยะทางประเมิน (Near/Far) | ผลการลงชื่อ (Passed/Rejected) |
| :------: | :-------------------------- | :--------------: | :-----------------------: | :---------------------------: |
|  **1**   | `22:BF:91:F1:88:E0`         |     -45 dBm      |           NEAR            |            PASSED             |

![[Smart Classroom.png]]
## 6. คำถามท้ายการทดลอง (Post-Lab Questions)

- **การใช้ RF Signal Proximity (RSSI) ร่วมกับ HTTP Web Server บน ESP32 แก้ปัญหาการฝากเช็กชื่อแทนกันในห้องเรียนได้อย่างไร?**
    
    - **แนวคำตอบ:** ระบบจะตรวจสอบทั้งการเชื่อมต่อ Wi-Fi และบังคับให้ยืนยันตัวตนผ่านหน้าเว็บภายในวงแลนเฉพาะจุด (SoftAP) โดยอิงจากค่าความแรงของสัญญาณ (RSSI) ของอุปกรณ์จริงที่อยู่ในรัศมีใกล้เคียง ทำให้คนที่ไม่ได้อยู่จริงในห้อง (สัญญาณอ่อนเกินเกณฑ์) ไม่สามารถกดเช็กชื่อผ่านเว็บได้ ป้องกันการฝากเพื่อนกดหรือแชร์ลิงก์มาจากนอกห้องเรียน
        
- **เหตุใดระดับเกณฑ์ RSSI ที่ `-55 dBm` จึงเหมาะสมสำหรับการระบุตำแหน่งอุปกรณ์ให้อยู่ภายในรัศมีโต๊ะปฏิบัติการ?**
    
    - **แนวคำตอบ:** เนื่องจากสัญญาณวิทยุ Wi-Fi จะลดทอนลงตามระยะทางและสิ่งกีดขวาง ค่าประมาณ `-55 dBm` (หรือช่วงใกล้เคียง) เป็นค่าความแรงที่สะท้อนว่าอุปกรณ์อยู่ใกล้ตัวปล่อยสัญญาณ (Access Point บนโต๊ะ) ในระยะทางไม่กี่เมตร หากเดินออกห่างไปนอกห้องเรียนหรือไกลเกินพื้นที่ปฏิบัติการ ค่า RSSI จะตกลงไปมาก (เช่น ต่ำกว่า `-70 dBm`) ซึ่งใช้เป็นเกณฑ์คัดกรองระยะทางทางกายภาพได้อย่างแม่นยำ
        
- **หากต้องการต่อยอดมินิโปรเจกต์นี้ในอนาคต ให้สามารถบันทึกข้อมูลการเข้าเรียนลงระบบ Cloud (เช่น Google Sheets หรือ Firebase) จะต้องเพิ่มส่วนเชื่อมต่อใดบ้าง?**
    
    - **แนวคำตอบ:**
        
        1. เปลี่ยนโหมดการทำงานของ ESP32 จาก SoftAP (ที่สร้างวงแลนตัวเอง) เป็น **Station Mode (STA)** เพื่อให้ ESP32 สามารถเชื่อมต่ออินเทอร์เน็ต Wi-Fi ภายนอกได้
            
        2. เพิ่มโมดูลหรือฟังก์ชัน **HTTP Client / HTTPS Client** (เช่น ใช้ `esp_http_client`) เพื่อส่งข้อมูล JSON ของผู้เช็กชื่อ (MAC Address, Timestamp, Status) ยิง API ไปยัง Cloud Endpoint, Firebase Realtime Database หรือ Google Apps Script (Web App) ที่เตรียมไว้
# Log
```
I (27) boot: ESP-IDF v6.0.2 2nd stage bootloader
I (27) boot: compile time Aug 12 2026 17:41:07
I (28) boot: Multicore bootloader
I (29) boot: chip revision: v3.1
I (32) boot.esp32: SPI Speed      : 40MHz
I (35) boot.esp32: SPI Mode       : DIO
I (39) boot.esp32: SPI Flash Size : 2MB
I (42) boot: Enabling RNG early entropy source...
I (47) boot: Partition Table:
I (49) boot: ## Label            Usage          Type ST Offset   Length
I (56) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (62) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (69) boot:  2 factory          factory app      00 00 00010000 00100000
I (75) boot: End of partition table
I (79) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=1c0e0h (114912) map
I (127) esp_image: segment 1: paddr=0002c108 vaddr=3ffb0000 size=03f10h ( 16144) load
I (134) esp_image: segment 2: paddr=00030020 vaddr=400d0020 size=8e314h (582420) map
I (341) esp_image: segment 3: paddr=000be33c vaddr=3ffb3f10 size=00618h (  1560) load
I (342) esp_image: segment 4: paddr=000be95c vaddr=40080000 size=1562ch ( 87596) load
I (381) esp_image: segment 5: paddr=000d3f90 vaddr=50000000 size=00028h (    40) load
I (392) boot: Loaded app from partition at offset 0x10000
I (392) boot: Disabling RNG early entropy source...
I (403) cpu_start: Multicore app
I (411) cpu_start: GPIO 3 and 1 are used as console UART I/O pins
I (411) cpu_start: Pro cpu start user code
I (411) cpu_start: cpu freq: 160000000 Hz
I (413) app_init: Application information:
I (417) app_init: Project name:     proximity_attendance
I (422) app_init: App version:      029da6d-dirty
I (427) app_init: Compile time:     Aug 12 2026 17:36:55
I (432) app_init: ELF file SHA256:  2064eb734...
I (436) app_init: ESP-IDF:          v6.0.2
I (440) efuse_init: Min chip rev:     v0.0
I (444) efuse_init: Max chip rev:     v3.99 
I (448) efuse_init: Chip rev:         v3.1
I (452) heap_init: Initializing. RAM available for dynamic allocation:
I (458) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (463) heap_init: At 3FFB8AA0 len 00027560 (157 KiB): DRAM
I (468) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (473) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (479) heap_init: At 4009562C len 0000A9D4 (42 KiB): IRAM
W (485) spi_flash: Detected boya flash chip but using generic driver. For optimal functionality, enable `SPI_FLASH_SUPPORT_BOYA_CHIP` in menuconfig
I (497) spi_flash: detected chip: generic
I (501) spi_flash: flash io: dio
W (504) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (518) main_task: Started on CPU0
I (518) main_task: Calling app_main()
I (568) wifi:wifi driver task: 3ffc057c, prio:23, stack:6656, core=0
I (598) wifi:wifi firmware version: 00ad238
I (598) wifi:wifi certification version: v7.0
I (598) wifi:config NVS flash: enabled
I (598) wifi:config nano formatting: disabled
I (598) wifi:Init data frame dynamic rx buffer num: 32
I (608) wifi:Init static rx mgmt buffer num: 5
I (608) wifi:Init management short buffer num: 32
I (608) wifi:Init dynamic tx buffer num: 32
I (618) wifi:Init static rx buffer size: 1600
I (618) wifi:Init static rx buffer num: 10
I (628) wifi:Init dynamic rx buffer num: 32
I (628) wifi_init: rx ba win: 6
I (628) wifi_init: accept mbox: 6
I (638) wifi_init: tcpip mbox: 32
I (638) wifi_init: udp mbox: 6
I (638) wifi_init: tcp mbox: 6
I (638) wifi_init: tcp tx win: 5760
I (648) wifi_init: tcp rx win: 5760
I (648) wifi_init: tcp mss: 1440
I (648) wifi_init: WiFi IRAM OP enabled
I (658) wifi_init: WiFi RX IRAM OP enabled
I (668) phy_init: phy_version 4863,a3a4459,Oct 28 2025,14:30:06
I (748) wifi:mode : softAP (14:08:08:a4:b6:61)
I (758) wifi:Total power save buffer number: 16
I (758) wifi:Init max length of beacon: 752/752
I (758) wifi:Init max length of beacon: 752/752
W (758) wifi:no need to send deauth when softap is sending deauth
W (768) wifi:no need to send deauth when softap is sending deauth
W (768) wifi:no need to send deauth when softap is sending deauth
W (778) wifi:no need to send deauth when softap is sending deauth
W (788) wifi:no need to send deauth when softap is sending deauth
W (788) wifi:no need to send deauth when softap is sending deauth
W (798) wifi:no need to send deauth when softap is sending deauth
W (798) wifi:no need to send deauth when softap is sending deauth
W (808) wifi:no need to send deauth when softap is sending deauth
W (818) wifi:no need to send deauth when softap is sending deauth
W (818) wifi:no need to send deauth when softap is sending deauth
W (828) wifi:no need to send deauth when softap is sending deauth
W (828) wifi:no need to send deauth when softap is sending deauth
W (838) wifi:no need to send deauth when softap is sending deauth
W (848) wifi:no need to send deauth when softap is sending deauth
W (848) wifi:no need to send deauth when softap is sending deauth
W (858) wifi:no need to send deauth when softap is sending deauth
W (858) wifi:no need to send deauth when softap is sending deauth
W (868) wifi:no need to send deauth when softap is sending deauth
W (868) wifi:no need to send deauth when softap is sending deauth
W (878) wifi:no need to send deauth when softap is sending deauth
W (888) wifi:no need to send deauth when softap is sending deauth
W (888) wifi:no need to send deauth when softap is sending deauth
W (898) wifi:no need to send deauth when softap is sending deauth
W (898) wifi:no need to send deauth when softap is sending deauth
W (908) wifi:no need to send deauth when softap is sending deauth
W (908) wifi:no need to send deauth when softap is sending deauth
W (918) wifi:no need to send deauth when softap is sending deauth
W (928) wifi:no need to send deauth when softap is sending deauth
W (928) wifi:no need to send deauth when softap is sending deauth
W (938) wifi:no need to send deauth when softap is sending deauth
W (938) wifi:no need to send deauth when softap is sending deauth
W (948) wifi:no need to send deauth when softap is sending deauth
I (958) esp_netif_lwip: DHCP server started on interface WIFI_AP_DEF with IP: 192.168.4.1
I (968) SMART_ATTENDANCE: Attendance Web Server Started at http://192.168.4.1
I (968) main_task: Returned from app_main()
W (968) wifi:no need to send deauth when softap is sending deauth
W (978) wifi:no need to send deauth when softap is sending deauth
W (988) wifi:no need to send deauth when softap is sending deauth
W (988) wifi:no need to send deauth when softap is sending deauth
W (998) wifi:no need to send deauth when softap is sending deauth
W (998) wifi:no need to send deauth when softap is sending deauth
W (1008) wifi:no need to send deauth when softap is sending deauth
W (1008) wifi:no need to send deauth when softap is sending deauth
W (1018) wifi:no need to send deauth when softap is sending deauth
W (1028) wifi:no need to send deauth when softap is sending deauth
W (1028) wifi:no need to send deauth when softap is sending deauth
W (1038) wifi:no need to send deauth when softap is sending deauth
W (1038) wifi:no need to send deauth when softap is sending deauth
W (1048) wifi:no need to send deauth when softap is sending deauth
W (1058) wifi:no need to send deauth when softap is sending deauth
W (1058) wifi:no need to send deauth when softap is sending deauth
W (1068) wifi:no need to send deauth when softap is sending deauth
W (1068) wifi:no need to send deauth when softap is sending deauth
W (1078) wifi:no need to send deauth when softap is sending deauth
W (1088) wifi:no need to send deauth when softap is sending deauth
W (1088) wifi:no need to send deauth when softap is sending deauth
W (1098) wifi:no need to send deauth when softap is sending deauth
W (1098) wifi:no need to send deauth when softap is sending deauth
W (1108) wifi:no need to send deauth when softap is sending deauth
W (1108) wifi:no need to send deauth when softap is sending deauth
W (1118) wifi:no need to send deauth when softap is sending deauth
W (1128) wifi:no need to send deauth when softap is sending deauth
W (1128) wifi:no need to send deauth when softap is sending deauth
W (1138) wifi:no need to send deauth when softap is sending deauth
W (1138) wifi:no need to send deauth when softap is sending deauth
W (1148) wifi:no need to send deauth when softap is sending deauth
W (1158) wifi:no need to send deauth when softap is sending deauth
W (1158) wifi:no need to send deauth when softap is sending deauth
W (1168) wifi:no need to send deauth when softap is sending deauth
W (1168) wifi:no need to send deauth when softap is sending deauth
W (1178) wifi:no need to send deauth when softap is sending deauth
W (1188) wifi:no need to send deauth when softap is sending deauth
W (1188) wifi:no need to send deauth when softap is sending deauth
W (1198) wifi:no need to send deauth when softap is sending deauth
W (1198) wifi:no need to send deauth when softap is sending deauth
W (1208) wifi:no need to send deauth when softap is sending deauth
W (1218) wifi:no need to send deauth when softap is sending deauth
W (1218) wifi:no need to send deauth when softap is sending deauth
W (1228) wifi:no need to send deauth when softap is sending deauth
W (1228) wifi:no need to send deauth when softap is sending deauth
W (1238) wifi:no need to send deauth when softap is sending deauth
W (1248) wifi:no need to send deauth when softap is sending deauth
W (1248) wifi:no need to send deauth when softap is sending deauth
W (1258) wifi:no need to send deauth when softap is sending deauth
W (1258) wifi:no need to send deauth when softap is sending deauth
W (1268) wifi:no need to send deauth when softap is sending deauth
W (1268) wifi:no need to send deauth when softap is sending deauth
W (1278) wifi:no need to send deauth when softap is sending deauth
W (1288) wifi:no need to send deauth when softap is sending deauth
W (1288) wifi:no need to send deauth when softap is sending deauth
W (1298) wifi:no need to send deauth when softap is sending deauth
W (1298) wifi:no need to send deauth when softap is sending deauth
W (1308) wifi:no need to send deauth when softap is sending deauth
W (1318) wifi:no need to send deauth when softap is sending deauth
W (1318) wifi:no need to send deauth when softap is sending deauth
I (2108) wifi:new:<1,0>, old:<1,1>, ap:<1,0>, sta:<255,255>, prof:1, snd_ch_cfg:0x0
I (2108) wifi:station: 22:bf:91:f1:88:e0 join, AID=1, bgn, 20
I (2148) SMART_ATTENDANCE: [PROXIMITY DETECTED]: New student device connected!
I (2158) wifi:<ba-add>idx:2 (ifx:1, 22:bf:91:f1:88:e0), tid:1, ssn:50, winSize:64
I (2158) wifi:<ba-add>idx:3 (ifx:1, 22:bf:91:f1:88:e0), tid:0, ssn:60, winSize:64
I (2188) esp_netif_lwip: DHCP server assigned IP to a client, IP is: 192.168.4.2
W (10998) httpd_uri: httpd_uri: URI '/api/data' not found
W (10998) httpd_txrx: httpd_resp_send_err: 404 Not Found - Nothing matches the given URI
W (39988) httpd_txrx: httpd_sock_err: error in recv : 104
W (68628) httpd_uri: httpd_uri: URI '/checkin' not found
W (68628) httpd_txrx: httpd_resp_send_err: 404 Not Found - Nothing matches the given URI
W (76118) httpd_uri: httpd_uri: URI '/checkin' not found
W (76118) httpd_txrx: httpd_resp_send_err: 404 Not Found - Nothing matches the given URI
W (78808) httpd_uri: httpd_uri: URI '/checkin' not found
W (78808) httpd_txrx: httpd_resp_send_err: 404 Not Found - Nothing matches the given URI
W (83188) httpd_uri: httpd_uri: URI '/checkin' not found
W (83188) httpd_txrx: httpd_resp_send_err: 404 Not Found - Nothing matches the given URI
I (110018) wifi:station: 22:bf:91:f1:88:e0 leave, AID = 1, reason = 8, bss_flags is 691299, bss:0x3ffb9e54
I (110018) wifi:<ba-del>idx:3, tid:0
I (110018) wifi:<ba-del>idx:2, tid:1
I (110038) wifi:new:<1,1>, old:<1,0>, ap:<1,1>, sta:<255,255>, prof:1, snd_ch_cfg:0x0
I (112508) wifi:new:<1,0>, old:<1,1>, ap:<1,0>, sta:<255,255>, prof:1, snd_ch_cfg:0x0
I (112508) wifi:station: 22:bf:91:f1:88:e0 join, AID=1, bgn, 20
I (112538) SMART_ATTENDANCE: [PROXIMITY DETECTED]: New student device connected!
I (112688) esp_netif_lwip: DHCP server assigned IP to a client, IP is: 192.168.4.2
I (113028) wifi:<ba-add>idx:2 (ifx:1, 22:bf:91:f1:88:e0), tid:0, ssn:2, winSize:64
I (116508) wifi:<ba-add>idx:3 (ifx:1, 22:bf:91:f1:88:e0), tid:1, ssn:0, winSize:64
I (117508) wifi:station: 22:bf:91:f1:88:e0 leave, AID = 1, reason = 8, bss_flags is 691315, bss:0x3ffb9e54
I (117508) wifi:<ba-del>idx:2, tid:0
I (117508) wifi:<ba-del>idx:3, tid:1
I (117518) wifi:new:<1,1>, old:<1,0>, ap:<1,1>, sta:<255,255>, prof:1, snd_ch_cfg:0x0
I (118758) wifi:new:<1,0>, old:<1,1>, ap:<1,0>, sta:<255,255>, prof:1, snd_ch_cfg:0x0
I (118758) wifi:station: 22:bf:91:f1:88:e0 join, AID=1, bgn, 20
I (118788) SMART_ATTENDANCE: [PROXIMITY DETECTED]: New student device connected!
I (119008) esp_netif_lwip: DHCP server assigned IP to a client, IP is: 192.168.4.2
I (119148) wifi:<ba-add>idx:2 (ifx:1, 22:bf:91:f1:88:e0), tid:0, ssn:2, winSize:64
I (119148) wifi:<ba-add>idx:3 (ifx:1, 22:bf:91:f1:88:e0), tid:1, ssn:0, winSize:64
I (123808) wifi:station: 22:bf:91:f1:88:e0 leave, AID = 1, reason = 8, bss_flags is 691315, bss:0x3ffb9e54
I (123808) wifi:<ba-del>idx:2, tid:0
I (123808) wifi:<ba-del>idx:3, tid:1
I (123818) wifi:new:<1,1>, old:<1,0>, ap:<1,1>, sta:<255,255>, prof:1, snd_ch_cfg:0x0
I (124788) wifi:new:<1,0>, old:<1,1>, ap:<1,0>, sta:<255,255>, prof:1, snd_ch_cfg:0x0
I (124788) wifi:station: 22:bf:91:f1:88:e0 join, AID=1, bgn, 20
I (124818) SMART_ATTENDANCE: [PROXIMITY DETECTED]: New student device connected!
I (124928) esp_netif_lwip: DHCP server assigned IP to a client, IP is: 192.168.4.2
I (125268) wifi:<ba-add>idx:2 (ifx:1, 22:bf:91:f1:88:e0), tid:0, ssn:2, winSize:64
I (125928) wifi:<ba-add>idx:3 (ifx:1, 22:bf:91:f1:88:e0), tid:1, ssn:0, winSize:64
I (131478) wifi:station: 22:bf:91:f1:88:e0 leave, AID = 1, reason = 8, bss_flags is 691299, bss:0x3ffb9e54
I (131478) wifi:<ba-del>idx:2, tid:0
I (131478) wifi:<ba-del>idx:3, tid:1
```
