# คู่มือการเชื่อมต่อ API การสร้าง QR Payment

## 1. บทนำ

เมื่อท่านสมัครการใช้งานจะได้รับข้อมูลการเข้าถึง API จากเรา

และท่านจะได้ข้อมูลโดยมีข้อมูลดังนี้

| ชื่อพารามิเตอร์ | ประเภท | คำอธิบาย                              |
| --------------- | ------ | ------------------------------------- |
| `SERVER_IP`     | string | Server endpoint ip                    |
| `SERVER_PORT`   | string | Server endpoint port                  |
| `CUST_CODE`     | string | รหัสลูกค้า                            |
| `API_KEY`       | string | API key เพื่อระบุตัวตนของผู้ใช้งาน    |
| `API_SECRET`    | string | API secret เพื่อระบุตัวตนของผู้ใช้งาน |

และสามารถเริ่มใช้งานได้ทันที

1. API สำหรับการสร้าง QR Payment ช่วยให้คุณสามารถสร้าง QR Code สำหรับการชำระเงินได้ง่าย ๆ ผ่าน HTTP Request โดยรองรับรูปแบบการชำระเงินแบบ PromptPay QR Code และการโอนผ่านเลขที่บัญชี
2. API สำหรับการตรวจสอบสถานะการชำระเงิน ช่วยให้คุณสามารถตรวจสอบสถานะการชำระเงินได้ง่าย ๆ ผ่าน HTTP Request โดยส่ง `payment_id` ที่ได้จากการสร้าง QR Payment

## 2. วิธีการเรียก API

### Enpoint

- URL: `https://{{SERVER_IP}}:{{SERVER_PORT}}/`

#### ทุก api จะต้องมี header ดังนี้

| ชื่อพารามิเตอร์ | ประเภท | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย                              |
| --------------- | ------ | ------ | ---------------- | ------------------------------------- |
| `Content-Type`  | string | ✓      | ✗                | ประเภทของไฟล์ (`application/json`)    |
| `x-cust-code`   | string | ✓      | ✗                | รหัสลูกค้า                            |
| `x-api-key`     | string | ✓      | ✗                | API key เพื่อระบุตัวตนของผู้ใช้งาน    |
| `x-api-secret`  | string | ✓      | ✗                | API secret เพื่อระบุตัวตนของผู้ใช้งาน |

### 2.1 API สำหรับการสร้าง QR Payment

#### \*QR จะมีอายุ 5 นาทีในการอัพเดทอัตโนมัติ หลังจากการขอ หาเกิน 5 นาทีมีคนแจ้งโอนเงินมา ให้แจ้งมายังทีม Support

Method: `POST`<br/>
URL: `/api/v1/qrcode-payments`

#### คำอธิบาย qrcode-payments body parameters json

| ชื่อพารามิเตอร์      | ประเภท | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย                                                                  |
| -------------------- | ------ | ------ | ---------------- | ------------------------------------------------------------------------- |
| `callback_url`       | string | ✓      | ✗                | url สำหรับตอบกลับเมื่อชำระเงินเสร็จ                                       |
| `ref_id`             | string | ✓      | ✗                | รหัสอ้างอิงจากทางลูกค้า                                                   |
| `payment_type`       | string | ✓      | ✗                | ประเภทของ QR Payment (`promptpay`)                                        |
| `amount`             | float  | ✓      | ✗                | จำนวนเงินที่ต้องการชำระ (ใส่ `1` บาทขึ้นไป)                               |
| `currency`           | string | ✓      | ✗                | สกุลเงิน (`THB`)                                                          |
| `payer_name`         | string | ✓      | ✗                | ชื่อผู้โอนเงิน (\*ต้องใช้เนื่องจากต่าง ธ. ไม่มีเลขที่บัญชี)               |
| `payer_mobile`       | string | ✓      | ✗                | เบอร์โทรศัพท์ผู้โอนเงิน (\*ต้องใช้เนื่องจากต่าง ธ. ไม่มีเลขที่บัญชี)      |
| `payer_bank_account` | string | ✓      | ✗                | เลขที่บัญชีผู้โอนเงิน (\*ต้องใช้ ให้ match แม่นขึ้นในบาง ธ ที่มีเลขมาให้) |
| `payer_bank_code`    | string | ✓      | ✗                | รหัสธนาคารผู้โอนเงิน                                                      |
| `timestamp`          | string | ✓      | ✗                | เวลาที่ทำรายการ (รูปแบบ ISO 8601)                                         |

#### คำอธิบาย qrcode-payments response

| ชื่อพารามิเตอร์                 | ประเภท  | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย                                                                                                                                           |
| ------------------------------- | ------- | ------ | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `status_code`                   | integer | ✓      | ✗                | รหัสสถานะการตอบกลับ<sup style="color: red;">\*1</sup>                                                                                              |
| `message`                       | string  | ✓      | ✗                | ข้อความที่อธิบายสถานะการตอบกลับ                                                                                                                    |
| `data`                          | object  | ✓      | ✗                | ข้อมูลการชำระเงิน                                                                                                                                  |
| `data.transaction_id`           | string  | ✓      | ✗                | รหัสธุรกรรมจากระบบ                                                                                                                                 |
| `data.ref_id`                   | string  | ✓      | ✗                | รหัสอ้างอิงจากทางลูกค้า                                                                                                                            |
| `data.qr_code`                  | string  | ✓      | ✗                | ข้อมูล QR Code ในรูปแบบ Base64                                                                                                                     |
| `data.amount`                   | float   | ✓      | ✗                | จำนวนเงินที่ชำระ                                                                                                                                   |
| `data.currency`                 | string  | ✓      | ✗                | สกุลเงิน                                                                                                                                           |
| `data.payer_name`               | string  | ✓      | ✗                | ชื่อผู้ที่ต้องการชำระเงิน                                                                                                                          |
| `data.payer_mobile`             | string  | ✓      | ✗                | เบอร์โทรศัพท์ผู้ที่ต้องการชำระเงิน                                                                                                                 |
| `data.payer_bank_account`       | string  | ✓      | ✗                | เลขที่บัญชีผู้ที่ต้องการชำระเงิน                                                                                                                   |
| `data.payer_bank_code`          | string  | ✓      | ✗                | รหัสธนาคารผู้ที่ต้องการชำระเงิน                                                                                                                    |
| `data.beneficiary_bank_account` | string  | ✓      | ✗                | เลขที่บัญชีผู้รับเงิน                                                                                                                              |
| `data.beneficiary_bank_code`    | string  | ✓      | ✗                | รหัสธนาคารผู้รับเงิน                                                                                                                               |
| `data.beneficiary_name`         | string  | ✓      | ✗                | ชื่อผู้รับเงิน                                                                                                                                     |
| `data.request_timestamp`        | string  | ✓      | ✗                | เวลาที่ทำรายการ (รูปแบบ ISO 8601)                                                                                                                  |
| `data.payment_status`           | string  | ✓      | ✗                | สถานะการชำระเงิน​ <br/>(`PENDING` = รอดำเนินการ, `DONE` = สำเร็จ, `RECIEVED_BUT_NOT_MATCH` = ได้รับเงินแต่ไม่ตรงกับรายละเอียด, `FAILED` = ล้มเหลว) |

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

#### ตัวอย่าง

```json
{
	"status_code": 400,
	"message": "Bad Request (ตรวจสอบค่าพารามิเตอร์)",
	"data": null
}
```

---

### 2.2 API สำหรับการตรวจสอบสถานะการชำระเงิน

Method: `GET`<br/>
URL: `/api/v1/transactions/{{transaction_id}}`

#### คำอธิบาย check qrcode-payments query parameters

| พารามิเตอร์    | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                       |
| -------------- | ------------ | -------------- | ---------------- | ------------------------------ |
| transaction_id | string       | ✓              | ✗                | รหัสอ้างอิงจากการสร้าง QR Code |

#### คำอธิบาย check qrcode-payments response

| พารามิเตอร์                      | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                              |
| -------------------------------- | ------------ | -------------- | ---------------- | ------------------------------------- |
| status_code                      | number       | ✓              | ✗                | รหัสสถานะ HTTP                        |
| message                          | string       | ✓              | ✗                | ข้อความแสดงผลการทำงาน                 |
| data                             | object       | ✓              | ✓                | ข้อมูลการชำระเงิน                     |
| data.id                          | string       | ✓              | ✗                | รหัสธุรกรรม                           |
| data.ref_id                      | string       | ✓              | ✗                | รหัสอ้างอิงการชำระเงิน                |
| data.amount                      | number       | ✓              | ✗                | จำนวนเงินที่ต้องชำระ                  |
| data.currency                    | string       | ✓              | ✗                | สกุลเงิน                              |
| data.callback_url                | string       | ✓              | ✗                | URL สำหรับรับการแจ้งเตือน             |
| data.request_timestamp           | string       | ✓              | ✗                | เวลาที่ร้องขอ (รูปแบบ ISO 8601)       |
| data.payer_name                  | string       | ✓              | ✗                | ชื่อผู้ชำระเงิน                       |
| data.payer_mobile                | string       | ✓              | ✗                | เบอร์โทรศัพท์ผู้ชำระเงิน              |
| data.payer_bank_account          | string       | ✓              | ✗                | เลขที่บัญชีผู้ชำระเงิน                |
| data.payer_bank_code             | string       | ✓              | ✗                | รหัสธนาคารผู้ชำระเงิน                 |
| data.payment_status              | string       | ✓              | ✗                | สถานะการชำระเงิน                      |
| data.beneficiary_bank_account    | string       | ✓              | ✗                | เลขที่บัญชีผู้รับเงิน                 |
| data.beneficiary_bank_code       | string       | ✓              | ✗                | รหัสธนาคารผู้รับเงิน                  |
| data.beneficiary_bank_name       | string       | ✓              | ✗                | ชื่อธนาคารผู้รับเงิน                  |
| data.beneficiary_bank_name_short | string       | ✓              | ✗                | ชื่อย่อธนาคารผู้รับเงิน               |
| data.beneficiary_name            | string       | ✓              | ✗                | ชื่อผู้รับเงิน                        |
| data.ti_amount                   | number       | ✓              | ✗                | จำนวนเงินที่ชำระจริง                  |
| data.ti_customer                 | string       | ✓              | ✗                | ข้อมูลผู้ชำระเงิน                     |
| data.ti_at                       | string       | ✓              | ✗                | เวลาที่ชำระเงิน (รูปแบบ ISO 8601)     |
| data.ti_description              | string       | ✓              | ✗                | รายละเอียดการชำระเงิน                 |
| data.created_at                  | string       | ✓              | ✗                | เวลาที่สร้างรายการ (รูปแบบ ISO 8601)  |
| data.updated_at                  | string       | ✓              | ✗                | เวลาที่อัปเดตล่าสุด (รูปแบบ ISO 8601) |
| data.owner_id                    | string       | ✓              | ✗                | รหัสเจ้าของบัญชี                      |
| data.web_bank_name               | string       | ✓              | ✓                | ชื่อธนาคารบนเว็บ                      |

#### ตัวอย่างการใช้งาน API สำหรับการตรวจสอบสถานะการชำระเงิน

```bash
curl -X GET https://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/transactions/25021739391745511481672 \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}"
```

#### ตัวอย่างการตอบกลับ check qrcode-payments

```json
{
	"status_code": 200,
	"message": "OK (สำเร็จ) - [Success]",
	"data": {
		"id": "25021739391745511481672",
		"ref_id": "TEST123457",
		"amount": 1,
		"currency": "THB",
		"callback_url": "http://callback.com/api/hook",
		"request_timestamp": "2025-02-11T11:41:00Z",
		"payer_name": "ทดสอบ ระบบเกตเวย์",
		"payer_mobile": "0812345678",
		"payer_bank_account": "1234567890",
		"payer_bank_code": "014",
		"payment_status": "DONE",
		"beneficiary_bank_account": "1234567890",
		"beneficiary_bank_code": "014",
		"beneficiary_bank_name": "ธนาคารไทยพาณิชย์ จำกัด (มหาชน)",
		"beneficiary_bank_name_short": "SCB",
		"beneficiary_name": "บัญชี ระบบเกตเวย์",
		"ti_amount": 10000,
		"ti_customer": "ฝากถอนเงินโอนไม่ใช้สมุด",
		"ti_at": "2025-02-11T19:03:33+07:00",
		"ti_description": "กสิกรไทย (KBANK) /X987654",
		"created_at": "2025-02-12T20:22:25.512Z",
		"updated_at": "2025-02-12T21:12:37.989Z",
		"owner_id": "60e0af0e-4b40-4c32-8988-ee0d7e75afb6",
		"web_bank_name": ""
	}
}
```

### 2.3 API สำหรับการยืนยันการชำระเงิน

Method: `POST`<br/>
URL: `/api/v1/transactions/match`

#### คำอธิบาย match transaction

| พารามิเตอร์        | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                 |
| ------------------ | ------------ | -------------- | ---------------- | ------------------------ |
| transaction_id     | string       | ✓              | ✗                | รหัสธุรกรรม              |
| amount             | number       | ✓              | ✗                | จำนวนเงินที่ต้องชำระ     |
| payer_name         | string       | ✓              | ✗                | ชื่อผู้ชำระเงิน          |
| payer_mobile       | string       | ✓              | ✗                | เบอร์โทรศัพท์ผู้ชำระเงิน |
| payer_bank_account | string       | ✓              | ✗                | เลขที่บัญชีผู้ชำระเงิน   |
| payer_bank_code    | string       | ✓              | ✗                | รหัสธนาคารผู้ชำระเงิน    |

#### คำอธิบายตัวอย่างการตอบกลับ match transaction

| พารามิเตอร์                                  | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| -------------------------------------------- | ------------ | -------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| status_code                                  | number       | ✓              | ✗                | รหัสสถานะ HTTP                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| message                                      | string       | ✓              | ✗                | ข้อความแสดงผลการทำงาน                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| data.annotation                              | string       | ✓              | ✗                | คำอธิบายสถานะการยืนยันการชำระเงิน<br/>สามารถมีค่าเป็น: <br/>- `NOT_FOUND`: ไม่พบข้อมูล <br/>- `SUCCESS`: การยืนยันสำเร็จ <br/>- `AMOUNT_BANK_STATEMENT_NOT_MATCH_ORIGINAL_TRANSACTION`: จำนวนเงินใน statement ไม่ตรงกับธุรกรรมเดิม <br/>- `ACCOUNT_NUMBER_NOT_MATCH`: เลขที่บัญชีไม่ตรงกัน <br/>- `NAME_NOT_MATCH`: ชื่อไม่ตรงกัน <br/>- `LAST_4_DIGITS_NOT_MATCH`: 4 หลักสุดท้ายของบัญชีไม่ตรงกัน <br/>- `NAME_AI_NOT_MATCH`: ชื่อไม่ตรงกันตามการตรวจสอบ AI <br/>- `FAILED`: การยืนยันล้มเหลว |
| data.input                                   | object       | ✓              | ✗                | ข้อมูลการยืนยันการชำระเงิน ที่ส่งมา                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| data.input.transaction_id                    | string       | ✓              | ✗                | รหัสธุรกรรม                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| data.input.amount                            | number       | ✓              | ✗                | จำนวนเงินที่ต้องชำระ                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| data.input.payer_name                        | string       | ✓              | ✗                | ชื่อผู้ชำระเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| data.input.payer_bank_account                | string       | ✓              | ✗                | เลขที่บัญชีผู้ชำระเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| data.input.payer_bank_code                   | string       | ✓              | ✗                | รหัสธนาคารผู้ชำระเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| data.bank_statement                          | object       | ✓              | ✗                | ข้อมูลการยืนยันการชำระเงิน จากธนาคาร                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| data.bank_statement.amount                   | number       | ✓              | ✗                | จำนวนเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| data.bank_statement.amount_sign              | string       | ✓              | ✗                | สัญลักษณ์จำนวนเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| data.bank_statement.balance                  | number       | ✓              | ✗                | ยอดคงเหลือ                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| data.bank_statement.bank_code                | string       | ✓              | ✗                | รหัสธนาคาร                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| data.bank_statement.description              | string       | ✓              | ✗                | รายละเอียด                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| data.bank_statement.record_id                | string       | ✓              | ✗                | รหัสบันทึก                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| data.bank_statement.status                   | string       | ✓              | ✗                | สถานะ                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| data.bank_statement.transaction_at           | string       | ✓              | ✗                | วันที่ทำธุรกรรม                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| data.bank_statement.transaction_code         | string       | ✓              | ✗                | รหัสธุรกรรม                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| data.bank_statement.transaction_code_desc    | string       | ✓              | ✗                | คำอธิบายรหัสธุรกรรม                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| data.transaction                             | object       | ✓              | ✗                | ข้อมูลการยืนยันการชำระเงิน จากระบบ                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| data.transaction.id                          | string       | ✓              | ✗                | รหัสธุรกรรม                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| data.transaction.ref_id                      | string       | ✓              | ✗                | รหัสอ้างอิง                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| data.transaction.amount                      | number       | ✓              | ✗                | จำนวนเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| data.transaction.currency                    | string       | ✓              | ✗                | สกุลเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| data.transaction.callback_url                | string       | ✓              | ✗                | URL สำหรับการตอบกลับ                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| data.transaction.request_timestamp           | string       | ✓              | ✗                | เวลาที่ขอ                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| data.transaction.payer_name                  | string       | ✓              | ✗                | ชื่อผู้ชำระเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| data.transaction.payer_mobile                | string       | ✓              | ✗                | เบอร์โทรศัพท์ผู้ชำระเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| data.transaction.payer_bank_account          | string       | ✓              | ✗                | เลขที่บัญชีผู้ชำระเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| data.transaction.payer_bank_code             | string       | ✓              | ✗                | รหัสธนาคารผู้ชำระเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| data.transaction.payment_status              | string       | ✓              | ✗                | สถานะการชำระเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| data.transaction.beneficiary_bank_account    | string       | ✓              | ✗                | เลขที่บัญชีผู้รับเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| data.transaction.beneficiary_bank_code       | string       | ✓              | ✗                | รหัสธนาคารผู้รับเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| data.transaction.beneficiary_bank_name       | string       | ✓              | ✗                | ชื่อธนาคารผู้รับเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| data.transaction.beneficiary_bank_name_short | string       | ✓              | ✗                | ชื่อย่อธนาคารผู้รับเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| data.transaction.beneficiary_name            | string       | ✓              | ✗                | ชื่อผู้รับเงิน                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| data.transaction.ti_amount                   | number       | ✓              | ✗                | จำนวนเงินที่ทำธุรกรรม                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| data.transaction.ti_customer                 | string       | ✓              | ✗                | ลูกค้าที่ทำธุรกรรม                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| data.transaction.ti_at                       | string       | ✓              | ✗                | วันที่ทำธุรกรรม                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| data.transaction.ti_description              | string       | ✓              | ✗                | รายละเอียดการทำธุรกรรม                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| data.transaction.created_at                  | string       | ✓              | ✗                | วันที่สร้าง                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| data.transaction.updated_at                  | string       | ✓              | ✗                | วันที่อัปเดต                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| data.transaction.owner_id                    | string       | ✓              | ✗                | รหัสเจ้าของ                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| data.transaction.web_bank_name               | string       | ✓              | ✗                | ชื่อธนาคารบนเว็บ                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

#### ตัวอย่างการเรียก API

```bash
curl -X POST https://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/transactions/match \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}"
     -d '{
        "transaction_id": "25021739391745511481672",
        "amount": 10000,
        "payer_name": "ทดสอบ ระบบเกตเวย์",
        "payer_mobile": "0812345678",
        "payer_bank_account": "1234567890",
        "payer_bank_code": "014"
     }'
```

#### ตัวอย่างการตอบกลับ match transaction

```json
{
	"status_code": 200,
	"message": "OK (สำเร็จ) - [Success]",
	"data": {
		"input": {
			"transaction_id": "25021739391745511481672",
			"amount": 10,
			"payer_name": "ทดสอบ ระบบเกตเวย์",
			"payer_bank_account": "1234567890",
			"payer_bank_code": "014"
		},
		"bank_statement": {
			"amount": 10,
			"amount_sign": "+",
			"balance": 0,
			"bank_code": "014",
			"description": "กสิกรไทย (KBANK) /X987654",
			"record_id": "218245648014659999421092067584  ",
			"status": "NEW",
			"transaction_at": "2025-02-11T19:03:33+07:00",
			"transaction_code": "X1",
			"transaction_code_desc": "ฝากถอนเงินโอนไม่ใช้สมุด"
		},
		"transaction": {
			"id": "25021739391745511481672",
			"ref_id": "TEST123457",
			"amount": 1,
			"currency": "THB",
			"callback_url": "http://callback.com/api/hook",
			"request_timestamp": "2025-02-11T11:41:00Z",
			"payer_name": "ทดสอบ ระบบเกตเวย์",
			"payer_mobile": "0812345678",
			"payer_bank_account": "1234567890",
			"payer_bank_code": "014",
			"payment_status": "MANUAL_MATCH",
			"beneficiary_bank_account": "1234567890",
			"beneficiary_bank_code": "014",
			"beneficiary_bank_name": "ธนาคารไทยพาณิชย์ จำกัด (มหาชน)",
			"beneficiary_bank_name_short": "SCB",
			"beneficiary_name": "บัญชี ระบบเกตเวย์",
			"ti_amount": 10,
			"ti_customer": "ฝากถอนเงินโอนไม่ใช้สมุด",
			"ti_at": "2025-02-11T19:03:33+07:00",
			"ti_description": "กสิกรไทย (KBANK) /X987654",
			"created_at": "2025-02-12T20:22:25.512Z",
			"updated_at": "2025-02-12T21:30:57.586487582Z",
			"owner_id": "60e0af0e-4b40-4c32-8988-ee0d7e75afb6",
			"web_bank_name": ""
		}
	}
}
```

### 2.4 API สำหรับการยืนยันการชำระเงินพร้อม callback

Method: `POST`<br/>
URL: `{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/transactions/match-callback`

### การทำงานเหมือนกับการเรียก 2.3 เพิ่มการยิง callback ไปปลายทางที่ท่านใส่เข้ามาในตอนสร้าง QR Code

#### ตัวอย่างการเรียก API Match callback

```bash
curl -X POST https://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/transactions/match-callback \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}"
     -d '{
        "transaction_id": "25021739391745511481672",
        "amount": 10000,
        "payer_name": "ทดสอบ ระบบเกตเวย์",
        "payer_mobile": "0812345678",
        "payer_bank_account": "1234567890",
        "payer_bank_code": "014"
     }'
```

### พร้อมกับยิง callback ไปปลายทางที่ท่านใส่เข้ามาในตอนสร้าง QR Code

### ตัวอย่างการตอบกลับจาก callback ให้ดูหัวข้อที่ 3 ได้เลยครับ 👇👇

### 3. Hook api callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน

Method: `POST`<br/>
URL: `{{HOOK_CALLBACK_URL}}`

เมื่อมีการชำระเงินจะมีการส่งข้อมูลกลับมาที่ URL ที่ท่านที่กำหนดไว้ ตามรายละเอียดด้านล่าง

#### คำอธิบาย hook callback

| พารามิเตอร์                     | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                             |
| ------------------------------- | ------------ | -------------- | ---------------- | ------------------------------------ |
| `status_code`                   | number       | ✓              | ✗                | รหัสสถานะ HTTP                       |
| `message`                       | string       | ✓              | ✗                | ข้อความแสดงผลการทำงาน                |
| `data`                          | object       | ✓              | ✓                | ข้อมูลการชำระเงิน                    |
| `data.payment_status`           | string       | ✓              | ✗                | สถานะการชำระเงิน (`DONE` = สำเร็จ)   |
| `data.ref_id`                   | string       | ✓              | ✗                | รหัสอ้างอิงการชำระเงิน               |
| `data.transaction_id`           | string       | ✓              | ✗                | รหัสธุรกรรม                          |
| `data.amount`                   | number       | ✓              | ✗                | จำนวนเงินที่ต้องชำระ                 |
| `data.currency`                 | string       | ✓              | ✗                | สกุลเงิน                             |
| `data.description`              | string       | ✓              | ✗                | รายละเอียดการชำระเงิน                |
| `data.payment_date`             | string       | ✓              | ✗                | วันเวลาที่ชำระเงิน (รูปแบบ ISO 8601) |
| `data.payment_amount`           | number       | ✓              | ✗                | จำนวนเงินที่ชำระจริง                 |
| `data.payer_name`               | string       | ✓              | ✗                | ชื่อผู้ชำระเงิน                      |
| `data.payer_mobile`             | string       | ✓              | ✗                | เบอร์โทรศัพท์ผู้ชำระเงิน             |
| `data.payer_bank_account`       | string       | ✓              | ✗                | เลขที่บัญชีผู้ชำระเงิน               |
| `data.payer_bank_code`          | string       | ✓              | ✗                | รหัสธนาคารผู้ชำระเงิน                |
| `data.beneficiary_bank_account` | string       | ✓              | ✗                | เลขที่บัญชีผู้รับเงิน                |
| `data.beneficiary_bank_code`    | string       | ✓              | ✗                | รหัสธนาคารผู้รับเงิน                 |
| `data.beneficiary_name`         | string       | ✓              | ✗                | ชื่อผู้รับเงิน                       |
| `data.request_timestamp`        | string       | ✓              | ✗                | วันเวลาที่ร้องขอ (รูปแบบ ISO 8601)   |

#### ตัวอย่างการตอบกลับ

```json
{
	"status_code": 200,
	"message": "OK (สำเร็จ) - [Success]",
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

## 4. API สำหรับการดึงข้อมูลลูกค้า

Method: `GET`<br/>
URL: `{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/customer`

#### คำอธิบายพารามิเตอร์การตอบกลับจาก API สำหรับการดึงข้อมูลลูกค้า

| พารามิเตอร์                           | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                                            |
| ------------------------------------- | ------------ | -------------- | ---------------- | --------------------------------------------------- |
| `status_code`                         | number       | ✓              | ✗                | รหัสสถานะ HTTP                                      |
| `message`                             | string       | ✓              | ✗                | ข้อความแสดงผลการทำงาน                               |
| `data`                                | object       | ✓              | ✓                | ข้อมูลลูกค้า                                        |
| `data.settled_balance`                | number       | ✓              | ✗                | ยอดเงินที่รับเงินเรียบร้อยแล้ว                      |
| `data.payout_balance`                 | number       | ✓              | ✗                | ยอดเงินที่โอนออกไปยัง บช พักเรียบร้อยแล้ว           |
| `data.customer`                       | object       | ✓              | ✗                | ข้อมูลลูกค้า                                        |
| `data.customer.id`                    | string       | ✓              | ✗                | รหัสลูกค้า                                          |
| `data.customer.cust_code`             | string       | ✓              | ✗                | รหัสลูกค้า                                          |
| `data.customer.customer_name`         | string       | ✓              | ✗                | ชื่อลูกค้า                                          |
| `data.customer.accounts`              | array        | ✓              | ✗                | บัญชีลูกค้า                                         |
| `data.customer.accounts.bank_code`    | string       | ✓              | ✗                | รหัสธนาคาร                                          |
| `data.customer.accounts.account_no`   | string       | ✓              | ✗                | เลขที่บัญชี                                         |
| `data.customer.accounts.account_name` | string       | ✓              | ✗                | ชื่อบัญชี                                           |
| `data.customer.accounts.account_type` | string       | ✓              | ✗                | ประเภทบัญชี                                         |
| `data.customer.accounts.status`       | boolean      | ✓              | ✗                | สถานะบัญชี                                          |
| `data.customer.is_active`             | boolean      | ✓              | ✗                | สถานะลูกค้า                                         |
| `data.customer.match_type`            | string       | ✓              | ✗                | ประเภทการตรวจสอบสถานะการชำระเงิน                    |
| `data.customer.hook_callback`         | string       | ✓              | ✗                | ลิงค์ callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน |
| `data.customer.hook_failed_url`       | string       | ✓              | ✗                | ลิงค์ callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน |
| `data.customer.owner_id`              | string       | ✓              | ✗                | รหัสผู้ดูแลลูกค้า                                   |

#### ตัวอย่างการเรียก API สำหรับการดึงข้อมูลลูกค้า

```bash
curl -X GET https://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/customer \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}"
```

#### ตัวอย่างการตอบกลับ API สำหรับการดึงข้อมูลลูกค้า

```json
{
	"status_code": 200,
	"message": "Customers fetched successfully",
	"data": {
		"settled_balance": 3295,
		"payout_balance": 0,
		"customer": {
			"id": "67a4afe39be1807f84017e91",
			"cust_code": "CUSTCODE01",
			"api_key": "C01-9999998889890",
			"api_secret": "qwerty987654321qwerty",
			"customer_name": "ชื่อลูกค้า",
			"accounts": [
				{
					"bank_code": "014",
					"account_no": "1234567890",
					"account_name": "ชื่อบัญชี",
					"account_type": "savings",
					"status": true
				}
			],
			"is_active": true,
			"match_type": "account",
			"hook_callback": "https://payment.api.com/hook/callback",
			"hook_failed_url": "https://payment.api.com/hook/callback-failed",
			"owner_id": "60e0af0e-4b40-4c32-8988-ee0d7e75afb6"
		}
	}
}
```

---

## 5. ติดต่อฝ่ายสนับสนุน

หากพบปัญหาในการใช้งาน API กรุณาติดต่อทีมสนับสนุนที่อีเมล <qrpayments.pgw@gmail.com>

## 6. HTTP Status Code

#### status code

| รหัส  | คำอธิบาย                                             |
| ----- | ---------------------------------------------------- |
| `200` | OK                                                   |
| `201` | Created - [message from pgw]                         |
| `202` | Accepted - [message from pgw]                        |
| `203` | Non-Authoritative Information - [message from pgw]   |
| `204` | No Content - [message from pgw]                      |
| `400` | Bad Request - [message from pgw]                     |
| `401` | Unauthorized - [message from pgw]                    |
| `402` | Payment Required - [message from pgw]                |
| `403` | Forbidden - [message from pgw]                       |
| `404` | Not Found - [message from pgw]                       |
| `406` | Not Acceptable - [message from pgw]                  |
| `408` | Request Timeout - [message from pgw]                 |
| `409` | Conflict - [message from pgw]                        |
| `421` | Misdirected Request - [message from pgw]             |
| `430` | Request Header Fields Too Large - [message from pgw] |
| `440` | Login Time-out - [message from pgw]                  |
| `500` | Internal Server Error - [message from pgw]           |
| `502` | Bad Gateway - [message from pgw]                     |
| `503` | Service Unavailable - [message from pgw]             |
| `504` | Gateway Timeout - [message from pgw]                 |

## 7. Bank code

### ดาวน์โหลดตาราง Bank Code

[⬇️ ดาวน์โหลดตาราง Bank Code (CSV)](https://raw.githubusercontent.com/qrpayments/docs/main/bank_codes.csv)

| bank_code | bank_name_th                                        | bank_name_en                                              |
| --------- | --------------------------------------------------- | --------------------------------------------------------- |
| 002       | ธนาคารกรุงเทพ จำกัด (มหาชน)                         | BANGKOK BANK PUBLIC COMPANY LIMITED                       |
| 004       | ธนาคารกสิกรไทย จำกัด (มหาชน)                        | KASIKORNBANK PUBLIC COMPANY LIMITED                       |
| 006       | ธนาคารกรุงไทย จำกัด (มหาชน)                         | KRUNG THAI BANK PUBLIC COMPANY LIMITED                    |
| 008       | ธนาคารเจพีมอร์แกน เชส                               | JPMORGAN CHASE BANK, N.A.                                 |
| 009       | ธนาคารโอเวอร์ซี-ไชนีสแบงกิ้งคอร์ปอเรชั่น จำกัด      | OVERSEA-CHINESE BANKING CORPORATION LIMITED               |
| 011       | ธนาคารทหารไทยธนชาต จำกัด (มหาชน)                    | TMBTHANACHART BANK PUBLIC COMPANY LIMITED                 |
| 014       | ธนาคารไทยพาณิชย์ จำกัด (มหาชน)                      | THE SIAM COMMERCIAL BANK PUBLIC COMPANY LIMITED           |
| 017       | ธนาคารซิตี้แบงก์ เอ็น.เอ.                           | CITIBANK, N.A.                                            |
| 018       | ธนาคารซูมิโตโม มิตซุย แบงกิ้ง คอร์ปอเรชั่น          | SUMITOMO MITSUI BANKING CORPORATION                       |
| 020       | ธนาคารสแตนดาร์ดชาร์เตอร์ด (ไทย) จำกัด (มหาชน)       | STANDARD CHARTERED BANK (THAI) PUBLIC COMPANY LIMITED     |
| 022       | ธนาคารซีไอเอ็มบี ไทย จำกัด (มหาชน)                  | CIMB THAI BANK PUBLIC COMPANY LIMITED                     |
| 023       | ธนาคารอาร์ เอช บี จำกัด                             | RHB BANK BERHAD, THAILAND                                 |
| 024       | ธนาคารยูโอบี จำกัด (มหาชน)                          | UNITED OVERSEAS BANK (THAI) PUBLIC COMPANY LIMITED        |
| 025       | ธนาคารกรุงศรีอยุธยา จำกัด (มหาชน)                   | BANK OF AYUDHAYA PUBLIC COMPANY LIMITED                   |
| 026       | ธนาคารเมกะ สากลพาณิชย์ จำกัด (มหาชน)                | MEGA INTERNATIONAL COMMERCIAL BANK PUBLIC COMPANY LIMITED |
| 027       | ธนาคารแห่งอเมริกาเนชั่นแนลแอสโซซิเอชั่น             | BANK OF AMERICA NATIONAL ASSOCIATION                      |
| 029       | ธนาคารอินเดียนโอเวอร์ซีส์                           | INDIAN OVERSEAS BANK                                      |
| 030       | ธนาคารออมสิน                                        | GOVERNMENT SAVINGS BANK                                   |
| 031       | ธนาคารฮ่องกงและเซี่ยงไฮ้แบงกิ้งคอร์ปอเรชั่น จำกัด   | THE HONGKONG AND SHANGHAI BANKING CORPORATION LIMITED     |
| 032       | ธนาคารดอยซ์แบงก์                                    | DEUTSCHE BANK AG.                                         |
| 033       | ธนาคารอาคารสงเคราะห์                                | GOVERNMENT HOUSTING BANK                                  |
| 034       | ธนาคารเพื่อการเกษตรและสหกรณ์การเกษตร                | BANK FOR AGRICULTURE AND AGRICULTURAL COOPERATIVES        |
| 035       | ธนาคารเพื่อการส่งออกและนำเข้าแห่งประเทศไทย          | EXPORT IMPORT BANK OF THAILAND                            |
| 039       | ธนาคารมิซูโฮ จำกัด สาขากรุงเทพฯ                     | MIZUHO BANK, LTD. BANGKOK BRANCH                          |
| 045       | ธนาคารบีเอ็นพี พารีบาส์                             | BNP PARIBAS BANGKOK BRANCH                                |
| 052       | ธนาคารแห่งประเทศจีน (ไทย) จำกัด (มหาชน)             | BANK OF CHINA (THAI) PUBLIC COMPANY LIMITED               |
| 066       | ธนาคารอิสลามแห่งประเทศไทย                           | ISLAMIC BANK OF THAILAND                                  |
| 067       | ธนาคารทิสโก้ จำกัด (มหาชน)                          | TISCO BANK PUBLIC COMPANY LIMITED                         |
| 069       | ธนาคารเกียรตินาคินภัทร จำกัด (มหาชน)                | KIATNAKIN PHATRA BANK PUBLIC COMPANY LIMITED              |
| 070       | ธนาคารไอซีบีซี (ไทย) จำกัด (มหาชน)                  | INDUSTRIAL AND COMMERCIAL BANK OF CHINA (THAI) PCL.       |
| 071       | ธนาคารไทยเครดิต จํากัด (มหาชน)                      | THAI CREDIT BANK PUBLIC COMPANY LIMITED                   |
| 073       | ธนาคารแลนด์ แอนด์ เฮ้าส์ จำกัด (มหาชน)              | LAND AND HOUSES BANK PUBLIC COMPANY LIMITED               |
| 080       | ธนาคารซูมิโตโม มิตซุย ทรัสต์ (ไทย) จำกัด (มหาชน)    | SUMITOMO MITSUI TRUST BANK (THAI) PUBLIC COMPANY LIMITED  |
| 098       | ธนาคารพัฒนาวิสาหกิจขนาดกลางและขนาดย่อมแห่งประเทศไทย | SMALL AND MEDIUM ENTERPRISE DEVELOPMENT BANK OF THAILAND  |
