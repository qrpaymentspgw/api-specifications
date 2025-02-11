# คู่มือการเชื่อมต่อ API การสร้าง QR Payment

## 1. บทนำ

เมื่อท่านสมัครการใช้งานจะได้รับข้อมูลการเข้าถึง API จากเรา

และท่านจะได้ข้อมูลโดยมีข้อมูลดังนี้

| ชื่อพารามิเตอร์ | ประเภท | คำอธิบาย |
|---------------|------|---------|
| `SERVER_IP` | string | Server endpoint ip |
| `SERVER_PORT` | string | Server endpoint port |
| `CUST_CODE` | string | รหัสลูกค้า |
| `API_KEY` | string | API key เพื่อระบุตัวตนของผู้ใช้งาน |
| `API_SECRET` | string | API secret เพื่อระบุตัวตนของผู้ใช้งาน |

และสามารถเริ่มใช้งานได้ทันที

1. API สำหรับการสร้าง QR Payment ช่วยให้คุณสามารถสร้าง QR Code สำหรับการชำระเงินได้ง่าย ๆ ผ่าน HTTP Request โดยรองรับรูปแบบการชำระเงินแบบ PromptPay QR Code และการโอนผ่านเลขที่บัญชี
2. API สำหรับการตรวจสอบสถานะการชำระเงิน ช่วยให้คุณสามารถตรวจสอบสถานะการชำระเงินได้ง่าย ๆ ผ่าน HTTP Request โดยส่ง `payment_id` ที่ได้จากการสร้าง QR Payment

## 2. วิธีการเรียก API

### Enpoint

- URL: `https://{{SERVER_IP}}:{{SERVER_PORT}}/`

#### ทุก api จะต้องมี header ดังนี้

| ชื่อพารามิเตอร์ | ประเภท | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย |
|---------------|------|--------|-----------------|---------|
| `Content-Type` | string | ✓ | ✗ | ประเภทของไฟล์ (`application/json`) |
| `x-cust-code` | string | ✓ | ✗ | รหัสลูกค้า |
| `x-api-key` | string | ✓ | ✗ | API key เพื่อระบุตัวตนของผู้ใช้งาน |
| `x-api-secret` | string | ✓ | ✗ | API secret เพื่อระบุตัวตนของผู้ใช้งาน |

### 2.1 API สำหรับการสร้าง QR Payment

#### *QR จะมีอายุ 5 นาทีในการอัพเดทอัตโนมัติ หลังจากการขอ หาเกิน 5 นาทีมีคนแจ้งโอนเงินมา ให้แจ้งมายังทีม Support

Method: `POST`<br/>
URL: `/api/v1/qrcode-payments`

#### คำอธิบาย qrcode-payments body parameters json

| ชื่อพารามิเตอร์ | ประเภท | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย |
|---------------|------|--------|-----------------|---------|
| `callback_url` | string | ✓ | ✗ | url สำหรับตอบกลับเมื่อชำระเงินเสร็จ |
| `ref_id` | string | ✓ | ✗ | รหัสอ้างอิงจากทางลูกค้า |
| `payment_type` | string | ✓ | ✗ | ประเภทของ QR Payment (`promptpay`) |
| `amount` | float | ✓ | ✗ | จำนวนเงินที่ต้องการชำระ (ใส่ `1` บาทขึ้นไป) |
| `currency` | string | ✓ | ✗ | สกุลเงิน (`THB`) |
| `payer_name` | string | ✓ | ✗ | ชื่อผู้โอนเงิน (*ต้องใช้เนื่องจากต่าง ธ. ไม่มีเลขที่บัญชี) |
| `payer_mobile` | string | ✓ | ✗ | เบอร์โทรศัพท์ผู้โอนเงิน (*ต้องใช้เนื่องจากต่าง ธ. ไม่มีเลขที่บัญชี) |
| `payer_bank_account` | string | ✓ | ✗ | เลขที่บัญชีผู้โอนเงิน (*ต้องใช้ ให้ match แม่นขึ้นในบาง ธ ที่มีเลขมาให้) |
| `payer_bank_code` | string | ✓| ✗ | รหัสธนาคารผู้โอนเงิน |
| `timestamp` | string | ✓ | ✗ | เวลาที่ทำรายการ (รูปแบบ ISO 8601) |

#### คำอธิบาย qrcode-payments response

| ชื่อพารามิเตอร์ | ประเภท | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย |
|---------------|------|--------|-----------------|---------|
| `status_code` | integer | ✓ | ✗ | รหัสสถานะการตอบกลับ<sup style="color: red;">*1</sup> |
| `message` | string | ✓ | ✗ | ข้อความที่อธิบายสถานะการตอบกลับ |
| `data` | object | ✓ | ✗ | ข้อมูลการชำระเงิน |
| `data.transaction_id` | string | ✓ | ✗ | รหัสธุรกรรมจากระบบ |
| `data.ref_id` | string | ✓ | ✗ | รหัสอ้างอิงจากทางลูกค้า |
| `data.qr_code` | string | ✓ | ✗ | ข้อมูล QR Code ในรูปแบบ Base64 |
| `data.amount` | float | ✓ | ✗ | จำนวนเงินที่ชำระ |
| `data.currency` | string | ✓ | ✗ | สกุลเงิน |
| `data.payer_name` | string | ✓ | ✗ | ชื่อผู้ที่ต้องการชำระเงิน |
| `data.payer_mobile` | string | ✓ | ✗ | เบอร์โทรศัพท์ผู้ที่ต้องการชำระเงิน |
| `data.payer_bank_account` | string | ✓ | ✗ | เลขที่บัญชีผู้ที่ต้องการชำระเงิน |
| `data.payer_bank_code` | string | ✓ | ✗ | รหัสธนาคารผู้ที่ต้องการชำระเงิน |
| `data.beneficiary_bank_account` | string | ✓ | ✗ | เลขที่บัญชีผู้รับเงิน |
| `data.beneficiary_bank_code` | string | ✓ | ✗ | รหัสธนาคารผู้รับเงิน |
| `data.beneficiary_name` | string | ✓ | ✗ | ชื่อผู้รับเงิน |
| `data.request_timestamp` | string | ✓ | ✗ | เวลาที่ทำรายการ (รูปแบบ ISO 8601) |
| `data.payment_status` | string | ✓ | ✗ | สถานะการชำระเงิน​ <br/>(`PENDING` = รอดำเนินการ, `DONE` = สำเร็จ, `RECIEVED_BUT_NOT_MATCH` = ได้รับเงินแต่ไม่ตรงกับรายละเอียด, `FAILED` = ล้มเหลว) |

#### ตัวอย่างการใช้งาน API สำหรับการสร้าง QR Payment

```bash
curl -X POST https://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/qrcode-payments \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}" \
     -d '{
       "callback_url": "https://domain.callback.net/result/pgw",
       "ref_id": "1234567890",
       "payment_type": "promptpay",
       "amount": 10000,
       "currency": "THB",
       "payer_name": "ทดสอบ ระบบเกตเวย์",
       "payer_mobile": "0812345678",
       "payer_bank_account": "1234567890",
       "payer_bank_code": "014"
     }' 
```

#### ตัวอย่างการตอบกลับ qrcode-payments

```json
{
  "status_code": 201,
  "message": "Payment Pending",
  "data": {
    "transaction_id": "25021739275352571710294",
    "ref_id": "REF123457", 
    "qr_code": "iVBORw0KGgoAAAANSUh...",
    "amount": 10000,
    "currency": "THB",
    "payer_name": "ทดสอบ ระบบเกตเวย์",
    "payer_mobile": "0812345678",
    "payer_bank_account": "1234567890", 
    "payer_bank_code": "014",
    "beneficiary_bank_account": "1234567890",
    "beneficiary_bank_code": "014",
    "beneficiary_name": "บัญชี ระบบเกตเวย์",
    "request_timestamp": "2025-02-11T11:41:00Z",
    "payment_status": "PENDING"
  }
}
```

#### ข้อผิดพลาดที่พบบ่อย

| รหัสข้อผิดพลาด | คำอธิบาย |
|--------------|---------|
| `400` | คำขอไม่ถูกต้อง (ตรวจสอบค่าพารามิเตอร์) |
| `401` | ไม่ได้รับอนุญาตให้ใช้งาน API |
| `500` | ข้อผิดพลาดจากเซิร์ฟเวอร์ |

#### ตัวอย่างข้อผิดพลาดที่พบบ่อย

```json
{
  "status_code": 400,
  "message": "คำขอไม่ถูกต้อง (ตรวจสอบค่าพารามิเตอร์)",
  "data": null
}
```

---

### 2.2 API สำหรับการตรวจสอบสถานะการชำระเงิน (COMMING Soon...)
<!-- 
Method: `GET`<br/>
URL: `/api/v1/qrcode-payments/{{payment_id}}`

#### คำอธิบาย check qrcode-payments query parameters

| พารามิเตอร์ | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย |
|-----------|------------|--------------|----------------|----------|
| ref_id | string | ✓ | ✗ | รหัสอ้างอิงการชำระเงินที่ต้องการตรวจสอบ |

#### คำอธิบาย check qrcode-payments response

| พารามิเตอร์ | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย |
|-----------|------------|--------------|----------------|----------|
| status | string | ✓ | ✗ | สถานะการตรวจสอบ <br/>(`success` = สำเร็จ, `failed` = ล้มเหลว) |
| data | object | ✓ | ✓ | ข้อมูลการชำระเงิน |
| data.status | string | ✓ | ✗ | สถานะการตรวจสอบ <br/>(`DONE` = สำเร็จ, `PENDING` = รอดำเนินการ, `CANCEL` = ยกเลิก) |
| data.ref_id | string | ✓ | ✗ | รหัสอ้างอิงการชำระเงิน |
| data.amount | number | ✓ | ✗ | จำนวนเงินที่ชำระ |
| data.description | string | ✗ | ✓ | รายละเอียดการชำระเงิน |
| data.payment_date | string | ✗ | ✓ | วันเวลาที่ชำระเงิน (รูปแบบ ISO 8601) |

#### ตัวอย่างการใช้งาน API สำหรับการตรวจสอบสถานะการชำระเงิน

```bash
curl -X GET https://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/qrcode-payments/1234567890 \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}"
```

#### ตัวอย่างการตอบกลับ check qrcode-payments

```json
{
  "status": "success",
  "message": "QR Code received successfully",
  "data": {
    "status": "DONE",
    "ref_id": "1234567890",
    "payment_id": "25021739006403384456201",
    "amount": 100.25,
    "description": "PromptPay x7878 นาย ทดสอบ ระบบ",
    "payment_date": "2025-01-20T17:34:00+07:00"
  }
}
``` -->

### 3. Hook api callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน

เมื่อมีการชำระเงินจะมีการส่งข้อมูลกลับมาที่ URL ที่ท่านที่กำหนดไว้ ตามรายละเอียดด้านล่าง

#### คำอธิบาย

| พารามิเตอร์ | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย |
|-----------|------------|--------------|----------------|----------|
| `status_code` | number | ✓ | ✗ | รหัสสถานะ HTTP |
| `message` | string | ✓ | ✗ | ข้อความแสดงผลการทำงาน |
| `data` | object | ✓ | ✓ | ข้อมูลการชำระเงิน |
| `data.payment_status` | string | ✓ | ✗ | สถานะการชำระเงิน (`DONE` = สำเร็จ) |
| `data.ref_id` | string | ✓ | ✗ | รหัสอ้างอิงการชำระเงิน |
| `data.transaction_id` | string | ✓ | ✗ | รหัสธุรกรรม |
| `data.amount` | number | ✓ | ✗ | จำนวนเงินที่ต้องชำระ |
| `data.currency` | string | ✓ | ✗ | สกุลเงิน |
| `data.description` | string | ✓ | ✗ | รายละเอียดการชำระเงิน |
| `data.payment_date` | string | ✓ | ✗ | วันเวลาที่ชำระเงิน (รูปแบบ ISO 8601) |
| `data.payment_amount` | number | ✓ | ✗ | จำนวนเงินที่ชำระจริง |
| `data.payer_name` | string | ✓ | ✗ | ชื่อผู้ชำระเงิน |
| `data.payer_mobile` | string | ✓ | ✗ | เบอร์โทรศัพท์ผู้ชำระเงิน |
| `data.payer_bank_account` | string | ✓ | ✗ | เลขที่บัญชีผู้ชำระเงิน |
| `data.payer_bank_code` | string | ✓ | ✗ | รหัสธนาคารผู้ชำระเงิน |
| `data.beneficiary_bank_account` | string | ✓ | ✗ | เลขที่บัญชีผู้รับเงิน |
| `data.beneficiary_bank_code` | string | ✓ | ✗ | รหัสธนาคารผู้รับเงิน |
| `data.beneficiary_name` | string | ✓ | ✗ | ชื่อผู้รับเงิน |
| `data.request_timestamp` | string | ✓ | ✗ | วันเวลาที่ร้องขอ (รูปแบบ ISO 8601) |

Method: `POST`<br/>
URL: `{{HOOK_CALLBACK_URL}}`

#### ตัวอย่างการตอบกลับ

```json
{
  "status": "success",
  "message": "QR Code received successfully",
  "data": {
    "payment_status": "DONE",
    "ref_id": "REF123457", 
    "transaction_id": "25021739274517180554000",
    "amount": 10000,
    "currency": "THB",
    "description": "ฝากถอนเงินโอนไม่ใช้สมุด",
    "payment_date": "2025-02-11T18:49:37+07:00",
    "payment_amount": 10000,
    "payer_name": "ทดสอบ ระบบเกตเวย์",
    "payer_mobile": "0812345678",
    "payer_bank_account": "1234567890",
    "payer_bank_code": "014",
    "beneficiary_bank_account": "1234567890", 
    "beneficiary_bank_code": "014",
    "beneficiary_name": "บัญชี ระบบเกตเวย์",
    "request_timestamp": "2025-02-11T18:49:37+07:00"
  }
}
```

---

## 4. ติดต่อฝ่ายสนับสนุน

หากพบปัญหาในการใช้งาน API กรุณาติดต่อทีมสนับสนุนที่อีเมล <qrpayments.pgw@gmail.com>

## 5. Bank code

### ดาวน์โหลดตาราง Bank Code

[⬇️ ดาวน์โหลดตาราง Bank Code (CSV)](https://raw.githubusercontent.com/qrpayments/docs/main/bank_codes.csv)

| bank_code | bank_name_th | bank_name_en |
|-----------|-------------|--------------|
| 002 | ธนาคารกรุงเทพ จำกัด (มหาชน) | BANGKOK BANK PUBLIC COMPANY LIMITED |
| 004 | ธนาคารกสิกรไทย จำกัด (มหาชน) | KASIKORNBANK PUBLIC COMPANY LIMITED |
| 006 | ธนาคารกรุงไทย จำกัด (มหาชน) | KRUNG THAI BANK PUBLIC COMPANY LIMITED |
| 008 | ธนาคารเจพีมอร์แกน เชส | JPMORGAN CHASE BANK, N.A. |
| 009 | ธนาคารโอเวอร์ซี-ไชนีสแบงกิ้งคอร์ปอเรชั่น จำกัด | OVERSEA-CHINESE BANKING CORPORATION LIMITED |
| 011 | ธนาคารทหารไทยธนชาต จำกัด (มหาชน) | TMBTHANACHART BANK PUBLIC COMPANY LIMITED |
| 014 | ธนาคารไทยพาณิชย์ จำกัด (มหาชน) | THE SIAM COMMERCIAL BANK PUBLIC COMPANY LIMITED |
| 017 | ธนาคารซิตี้แบงก์ เอ็น.เอ. | CITIBANK, N.A. |
| 018 | ธนาคารซูมิโตโม มิตซุย แบงกิ้ง คอร์ปอเรชั่น | SUMITOMO MITSUI BANKING CORPORATION |
| 020 | ธนาคารสแตนดาร์ดชาร์เตอร์ด (ไทย) จำกัด (มหาชน) | STANDARD CHARTERED BANK (THAI) PUBLIC COMPANY LIMITED |
| 022 | ธนาคารซีไอเอ็มบี ไทย จำกัด (มหาชน) | CIMB THAI BANK PUBLIC COMPANY LIMITED |
| 023 | ธนาคารอาร์ เอช บี จำกัด | RHB BANK BERHAD, THAILAND |
| 024 | ธนาคารยูโอบี จำกัด (มหาชน) | UNITED OVERSEAS BANK (THAI) PUBLIC COMPANY LIMITED |
| 025 | ธนาคารกรุงศรีอยุธยา จำกัด (มหาชน) | BANK OF AYUDHAYA PUBLIC COMPANY LIMITED |
| 026 | ธนาคารเมกะ สากลพาณิชย์ จำกัด (มหาชน) | MEGA INTERNATIONAL COMMERCIAL BANK PUBLIC COMPANY LIMITED |
| 027 | ธนาคารแห่งอเมริกาเนชั่นแนลแอสโซซิเอชั่น | BANK OF AMERICA NATIONAL ASSOCIATION |
| 029 | ธนาคารอินเดียนโอเวอร์ซีส์ | INDIAN OVERSEAS BANK |
| 030 | ธนาคารออมสิน | GOVERNMENT SAVINGS BANK |
| 031 | ธนาคารฮ่องกงและเซี่ยงไฮ้แบงกิ้งคอร์ปอเรชั่น จำกัด | THE HONGKONG AND SHANGHAI BANKING CORPORATION LIMITED |
| 032 | ธนาคารดอยซ์แบงก์ | DEUTSCHE BANK AG. |
| 033 | ธนาคารอาคารสงเคราะห์ | GOVERNMENT HOUSTING BANK |
| 034 | ธนาคารเพื่อการเกษตรและสหกรณ์การเกษตร | BANK FOR AGRICULTURE AND AGRICULTURAL COOPERATIVES |
| 035 | ธนาคารเพื่อการส่งออกและนำเข้าแห่งประเทศไทย | EXPORT IMPORT BANK OF THAILAND |
| 039 | ธนาคารมิซูโฮ จำกัด สาขากรุงเทพฯ | MIZUHO BANK, LTD. BANGKOK BRANCH |
| 045 | ธนาคารบีเอ็นพี พารีบาส์ | BNP PARIBAS BANGKOK BRANCH |
| 052 | ธนาคารแห่งประเทศจีน (ไทย) จำกัด (มหาชน) | BANK OF CHINA (THAI) PUBLIC COMPANY LIMITED |
| 066 | ธนาคารอิสลามแห่งประเทศไทย | ISLAMIC BANK OF THAILAND |
| 067 | ธนาคารทิสโก้ จำกัด (มหาชน) | TISCO BANK PUBLIC COMPANY LIMITED |
| 069 | ธนาคารเกียรตินาคินภัทร จำกัด (มหาชน) | KIATNAKIN PHATRA BANK PUBLIC COMPANY LIMITED |
| 070 | ธนาคารไอซีบีซี (ไทย) จำกัด (มหาชน) | INDUSTRIAL AND COMMERCIAL BANK OF CHINA (THAI) PCL. |
| 071 | ธนาคารไทยเครดิต จํากัด (มหาชน) | THAI CREDIT BANK PUBLIC COMPANY LIMITED |
| 073 | ธนาคารแลนด์ แอนด์ เฮ้าส์ จำกัด (มหาชน) | LAND AND HOUSES BANK PUBLIC COMPANY LIMITED |
| 080 | ธนาคารซูมิโตโม มิตซุย ทรัสต์ (ไทย) จำกัด (มหาชน) | SUMITOMO MITSUI TRUST BANK (THAI) PUBLIC COMPANY LIMITED |
| 098 | ธนาคารพัฒนาวิสาหกิจขนาดกลางและขนาดย่อมแห่งประเทศไทย | SMALL AND MEDIUM ENTERPRISE DEVELOPMENT BANK OF THAILAND |
