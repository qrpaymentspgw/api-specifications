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

- URL: `http://{{SERVER_IP}}:{{SERVER_PORT}}/`

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
curl -X POST http://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/qrcode-payments \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}" \
     -d '{
       "callback_url": "http://domain.callback.net/result/pgw",
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
curl -X GET http://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/transactions/:transaction_id \
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

### 3. Hook api callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน

หากมีการทำรายการ Payout สำเร็จ จะมีการส่งข้อมูลกลับมาที่ URL ที่ระบุไว้ในพารามิเตอร์ `callback_url`

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
| `data.id`                             | string       | ✓              | ✗                | รหัสลูกค้า                                          |
| `data.cust_code`                      | string       | ✓              | ✗                | รหัสลูกค้า                                          |
| `data.api_key`                        | string       | ✓              | ✗                | คีย์สำหรับเรียกใช้ API                              |
| `data.api_secret`                     | string       | ✓              | ✗                | รหัสลับสำหรับเรียกใช้ API                           |
| `data.customer_name`                  | string       | ✓              | ✗                | ชื่อลูกค้า                                          |
| `data.accounts`                       | array        | ✓              | ✗                | บัญชีลูกค้า                                         |
| `data.accounts.bank_code`             | string       | ✓              | ✗                | รหัสธนาคาร                                          |
| `data.accounts.account_no`            | string       | ✓              | ✗                | เลขที่บัญชี                                         |
| `data.accounts.account_name`          | string       | ✓              | ✗                | ชื่อบัญชี                                           |
| `data.accounts.account_type`          | string       | ✓              | ✗                | ประเภทบัญชี                                         |
| `data.accounts.status`                | boolean      | ✓              | ✗                | สถานะบัญชี                                          |
| `data.is_active`                      | boolean      | ✓              | ✗                | สถานะลูกค้า                                         |
| `data.match_type`                     | string       | ✓              | ✗                | ประเภทการตรวจสอบสถานะการชำระเงิน                    |
| `data.hook_callback`                  | string       | ✓              | ✗                | ลิงค์ callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน |
| `data.hook_failed_url`                | string       | ✓              | ✗                | ลิงค์ callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน |
| `data.owner_id`                       | string       | ✓              | ✗                | รหัสผู้ดูแลลูกค้า                                   |
| `data.fee_rate`                       | number       | ✓              | ✗                | อัตราค่าธรรมเนียม                                   |
| `data.balance`                        | number       | ✓              | ✗                | ยอดเงินคงเหลือที่ใช้ได้                                   |

#### ตัวอย่างการเรียก API สำหรับการดึงข้อมูลลูกค้า

```bash
curl -X GET http://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/customer \
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
		"hook_callback": "http://payment.api.com/hook/callback",
		"hook_failed_url": "http://payment.api.com/hook/callback-failed",
		"owner_id": "60e0af0e-4b40-4c32-8988-ee0d7e75afb6",
		"fee_rate": 1.0,
		"balance": 1000
	}
}
```

## 5. API สำหรับ Payout

### ธนาคารที่รองรับ

| bank_code | bank_name_th                                        | bank_name_en                                              |
| --------- | -------------------------------------------------- | -------------------------------------------------------- |
| 014       | ธนาคารไทยพาณิชย์ จำกัด (มหาชน)                      | THE SIAM COMMERCIAL BANK PUBLIC COMPANY LIMITED           |
| 004       | ธนาคารกสิกรไทย จำกัด (มหาชน)                        | KASIKORNBANK PUBLIC COMPANY LIMITED                       |
| 006       | ธนาคารกรุงไทย จำกัด (มหาชน)                         | KRUNG THAI BANK PUBLIC COMPANY LIMITED                    |
| 002       | ธนาคารกรุงเทพ จำกัด (มหาชน)                         | BANGKOK BANK PUBLIC COMPANY LIMITED                       |
| 011       | ธนาคารทหารไทยธนชาต จำกัด (มหาชน)                    | TMBTHANACHART BANK PUBLIC COMPANY LIMITED                 |
| 030       | ธนาคารออมสิน                                        | GOVERNMENT SAVINGS BANK                                   |
| 025       | ธนาคารกรุงศรีอยุธยา จำกัด (มหาชน)                   | BANK OF AYUDHAYA PUBLIC COMPANY LIMITED                   |
| 034       | ธนาคารเพื่อการเกษตรและสหกรณ์การเกษตร                | BANK FOR AGRICULTURE AND AGRICULTURAL COOPERATIVES        |
| 024       | ธนาคารยูโอบี จำกัด (มหาชน)                          | UNITED OVERSEAS BANK (THAI) PUBLIC COMPANY LIMITED        |
| 033       | ธนาคารอาคารสงเคราะห์                                | GOVERNMENT HOUSTING BANK                                  |
| 022       | ธนาคารซีไอเอ็มบี ไทย จำกัด (มหาชน)                  | CIMB THAI BANK PUBLIC COMPANY LIMITED                     |

หากมีอัพเดท จะแจ้งให้ทราบผ่านช่องทางติดต่อกัน

### 5.1 สร้าง Payout

#### คำอธิบาย API สำหรับ Payout body parameters json

| พารามิเตอร์                           | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                                            |
| ------------------------------------- | ------------ | -------------- | ---------------- | --------------------------------------------------- |
| `callback_url`                        | string       | ✓              | ✗                | URL สำหรับรับการแจ้งเตือนผลการทำรายการ               |
| `ref_id`                              | string       | ✓              | ✗                | รหัสอ้างอิงจากลูกค้า                                  |
| `trigger`                             | string       | ✓              | ✗                | ช่องทางการทำรายการ                                   |
| `amount`                              | number       | ✓              | ✗                | จำนวนเงินที่ต้องการโอน                               |
| `currency`                            | string       | ✓              | ✗                | สกุลเงิน                                             |
| `beneficiary_bank_account_no`         | string       | ✓              | ✗                | เลขที่บัญชีปลายทาง                                   |
| `beneficiary_bank_account_name`       | string       | ✓              | ✗                | ชื่อบัญชีปลายทาง                                     |
| `beneficiary_bank_code`               | string       | ✓              | ✗                | รหัสธนาคารปลายทาง Code จาก [ตารางรหัสธนาคาร](#8-bank-code)           |
| `timestamp`                           | string       | ✓              | ✗                | เวลาที่ทำรายการ (RFC3339)                            |

#### คำอธิบายพารามิเตอร์การตอบกลับจาก API สำหรับการสร้าง Payout

| พารามิเตอร์                           | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                                            |
| ------------------------------------- | ------------ | -------------- | ---------------- | --------------------------------------------------- |
| `status_code`                         | number       | ✓              | ✗                | รหัสสถานะ HTTP                                      |
| `message`                             | string       | ✓              | ✗                | ข้อความแสดงผลการทำงาน                               |
| `data`                                | object       | ✓              | ✗                | ข้อมูลการทำรายการ                                    |
| `data.payout_id`                      | string       | ✓              | ✗                | รหัสอ้างอิงการทำรายการ Payout                        |
| `data.ref_id`                         | string       | ✓              | ✗                | รหัสอ้างอิงจากลูกค้า                                  |
| `data.callback_url`                   | string       | ✓              | ✗                | URL สำหรับรับการแจ้งเตือนผลการทำรายการ               |
| `data.trigger`                        | string       | ✓              | ✗                | ช่องทางการทำรายการ                                   |
| `data.amount`                         | number       | ✓              | ✗                | จำนวนเงินที่ต้องการโอน                               |
| `data.currency`                       | string       | ✓              | ✗                | สกุลเงิน                                             |
| `data.beneficiary_bank_account_no`    | string       | ✓              | ✗                | เลขที่บัญชีปลายทาง                                   |
| `data.beneficiary_bank_account_name`  | string       | ✓              | ✗                | ชื่อบัญชีปลายทาง                                     |
| `data.beneficiary_bank_code`          | string       | ✓              | ✗                | รหัสธนาคารปลายทาง                                    |
| `data.timestamp`                      | string       | ✓              | ✗                | เวลาที่ทำรายการ (RFC3339)                            |
| `data.status`                         | string       | ✓              | ✗                | สถานะการทำรายการ (QUEUED, SUCCESS, FAILED)           |

#### ตัวอย่างการเรียก API สำหรับการสร้าง Payout

```bash

curl -X POST http://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/payout \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}" \
     -d '{
		"callback_url": "https://payment.api.com/hook/callback",
		"ref_id": "TEST-1234567890",
		"trigger": "web manual",
		"amount": 1000,
		"currency": "THB",
		"beneficiary_bank_account_no": "5557774445",
		"beneficiary_bank_account_name": "ทดสอบ การโอนเงิน",
		"beneficiary_bank_code": "025", 
		"timestamp": "2025-02-28T16:23:00Z"
	}
'
```

#### ตัวอย่างการตอบกลับ API สำหรับการสร้าง Payout

```json
{
    "status_code": 201,
    "message": "Created - [payout created successfully]",
    "data": {
        "payout_id": "9032993c-4661-4ab9-9381-c258346a9d68",
        "ref_id": "TEST-1234567890", 
        "callback_url": "https://payment.api.com/hook/callback",
        "trigger": "web manual",
        "amount": 1000,
        "currency": "THB",
        "beneficiary_bank_account_no": "5557774445",
        "beneficiary_bank_account_name": "ทดสอบ การโอนเงิน",
        "beneficiary_bank_code": "025",
        "timestamp": "2025-02-28T16:23:00Z",
        "status": "QUEUED"
    }
}

```

### 5.2 Callback ผลการทำรายการ Payout

หากมีการทำรายการ Payout สำเร็จ จะมีการส่งข้อมูลกลับมาที่ URL ที่ระบุไว้ในพารามิเตอร์ `callback_url`

Method: `POST`<br/>
URL: `{{HOOK_CALLBACK_PAYOUT_URL}}`

#### คำอธิบาย hook callback payout

| พารามิเตอร์        | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                 |
| ------------------ | ------------ | -------------- | ---------------- | ------------------------ |
| payout_id          | string       | ✓              | ✗                | รหัสธุรกรรม              |
| ref_id             | string       | ✓              | ✗                | รหัสธุรกรรม              |
| currency           | string       | ✓              | ✗                | สกุลเงิน              |
| description        | string       | ✓              | ✗                | รายละเอียดการทำรายการ              |
| payout_date        | string       | ✓              | ✗                | วันที่ทำรายการ              |
| payout_amount      | number       | ✓              | ✗                | จำนวนเงินที่ทำรายการ              |
| beneficiary_bank_account | string       | ✓              | ✗                | บัญชีปลายทาง              |
| beneficiary_bank_code    | string       | ✓              | ✗                | รหัสธนาคารปลายทาง              |
| beneficiary_name         | string       | ✓              | ✗                | ชื่อบัญชีปลายทาง              |
| request_timestamp        | string       | ✓              | ✗                | เวลาที่ทำรายการ (RFC3339)              |

#### ตัวอย่างการตอบกลับ Callback ผลการทำรายการ Payout

```json
{
    "status_code": 200,
    "message": "Success",
    "data": {
        "payout_id": "46010361-73bc-40a2-8583-615f5e44e6f4",
        "ref_id": "TEST-1234567890",
        "currency": "THB",
        "description": "ทำรายการสำเร็จ",
        "payout_date": "2025-02-28T16:23:00Z",
        "payout_amount": 1000,
        "beneficiary_bank_account": "5557774445",
        "beneficiary_bank_code": "025",
        "beneficiary_name": "ทดสอบ การโอนเงิน",
        "request_timestamp": "2025-02-28T16:23:00Z"
    }
}
```

---

## 6. ติดต่อฝ่ายสนับสนุน

หากพบปัญหาในการใช้งาน API กรุณาติดต่อทีมสนับสนุนที่อีเมล <qrpayments.pgw@gmail.com>

## 7. HTTP Status Code

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

## 8. Bank code

### ดาวน์โหลดตาราง Bank Code

[⬇️ ดาวน์โหลดตาราง Bank Code (CSV)](http://raw.githubusercontent.com/qrpayments/docs/main/bank_codes.csv)


<!-- 
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
curl -X POST http://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/transactions/match \
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
curl -X POST http://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/transactions/match-callback \
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

### ตัวอย่างการตอบกลับจาก callback ให้ดูหัวข้อที่ 3 ได้เลยครับ 👇👇 -->
