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

### 2.1 API สำหรับการสร้างรายการ Payments

Method: `POST`<br/>
URL: `/api/v1/payment`

#### คำอธิบาย payment body parameters json

| ชื่อพารามิเตอร์      | ประเภท | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย                                                                  |
| -------------------- | ------ | ------ | ---------------- | ------------------------------------------------------------------------- |
| `callback_url`       | string | ✓      | ✗                | url สำหรับตอบกลับเมื่อชำระเงินเสร็จ                                       |
| `ref_id`             | string | ✓      | ✗                | รหัสอ้างอิงจากทางลูกค้า                                                   |
| `payment_type`       | string | ✓      | ✗                | ประเภทของ payment (`bank_account`)                                        |
| `amount`             | float  | ✓      | ✗                | จำนวนเงินที่ต้องการชำระ (ใส่ `1` บาทขึ้นไป)                               |
| `currency`           | string | ✓      | ✗                | สกุลเงิน (`THB`)                                                          |
| `payer_name`         | string | ✓      | ✗                | ชื่อผู้โอนเงิน (\*ต้องใช้เนื่องจากต่าง ธ. ไม่มีเลขที่บัญชี)               |
| `payer_mobile`       | string | ✓      | ✗                | เบอร์โทรศัพท์ผู้โอนเงิน (\*ต้องใช้เนื่องจากต่าง ธ. ไม่มีเลขที่บัญชี)      |
| `payer_bank_account` | string | ✓      | ✗                | เลขที่บัญชีผู้โอนเงิน (\*ต้องใช้ ให้ match แม่นขึ้นในบาง ธ ที่มีเลขมาให้) |
| `payer_bank_code`    | string | ✓      | ✗                | รหัสธนาคารผู้โอนเงิน                                                      |

### ตัด `timestamp` ออก

### คำอธิบาย payments response

| ชื่อพารามิเตอร์                 | ประเภท  | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย                                                                                                                                           |
| ------------------------------- | ------- | ------ | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `status_code`                   | integer | ✓      | ✗                | รหัสสถานะการตอบกลับ<sup style="color: red;">\*1</sup>                                                                                              |
| `message`                       | string  | ✓      | ✗                | ข้อความที่อธิบายสถานะการตอบกลับ                                                                                                                    |
| `data`                          | object  | ✓      | ✗                | ข้อมูลการชำระเงิน                                                                                                                                  |
| `data.transaction_id`           | string  | ✓      | ✗                | รหัสธุรกรรมจากระบบ                                                                                                                                 |
| `data.ref_id`                   | string  | ✓      | ✗                | รหัสอ้างอิงจากทางลูกค้า                                                                                                                            |
| `data.qr_code`                  | string  | ✓      | ✓                | ข้อมูล QR Code ในรูปแบบ Base64                                                                                                                     |
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
| `data.payment_status`           | string  | ✓      | ✗                | สถานะการชำระเงิน​ <br/>(`PENDING` = รอดำเนินการ, `SUCCESS` = สำเร็จ, `RECIEVED_BUT_NOT_MATCH` = ได้รับเงินแต่ไม่ตรงกับรายละเอียด, `FAILED` = ล้มเหลว) |

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
	"message": "invalid amount",
	"error": "amount must be greater than 0"
}
```

---

### 2.2 API สำหรับการตรวจสอบสถานะการชำระเงิน

Method: `GET`<br/>
URL: `/api/v1/payment/{{transaction_id}}`

#### คำอธิบาย check qrcode-payments query parameters

| พารามิเตอร์    | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                       |
| -------------- | ------------ | -------------- | ---------------- | ------------------------------ |
| transaction_id | string       | ✓              | ✗                | รหัสอ้างอิงจากการสร้าง QR Code |

#### คำอธิบาย check qrcode-payments response

| พารามิเตอร์                                 | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                                          |
| ------------------------------------------- | ------------ | -------------- | ---------------- | ------------------------------------------------- |
| status_code                                 | number       | ✓              | ✗                | รหัสสถานะ HTTP                                    |
| message                                     | string       | ✓              | ✗                | ข้อความแสดงผลการทำงาน                             |
| data                                        | object       | ✓              | ✓                | ข้อมูลการชำระเงิน                                 |
| data.transaction_id                         | string       | ✓              | ✗                | รหัสธุรกรรม                                       |
| data.merchant_id                            | string       | ✓              | ✗                | รหัสร้านค้า                                       |
| data.merchant_ref_id                        | string       | ✓              | ✗                | รหัสอ้างอิงจากร้านค้า                             |
| data.for_staff_id                           | string       | ✓              | ✓                | รหัสพนักงาน (payout fee)                          |
| data.type                                   | string       | ✓              | ✗                | ประเภทธุรกรรม (payin, payout)                     |
| data.callback_url                           | string       | ✓              | ✗                | URL สำหรับรับการแจ้งเตือน                         |
| data.request_detail                         | object       | ✓              | ✗                | รายละเอียดคำขอ                                    |
| data.request_detail.amount                  | number       | ✓              | ✗                | จำนวนเงินที่ร้องขอ                                |
| data.request_detail.callbackurl             | string       | ✓              | ✗                | URL สำหรับรับการแจ้งเตือน                         |
| data.request_detail.currency                | string       | ✓              | ✗                | สกุลเงิน                                          |
| data.request_detail.merchantrefid           | string       | ✓              | ✗                | รหัสอ้างอิงจากร้านค้า                             |
| data.request_detail.payerbankaccount        | string       | ✓              | ✓                | เลขที่บัญชีผู้ชำระเงิน                            |
| data.request_detail.payerbankcode           | string       | ✓              | ✓                | รหัสธนาคารผู้ชำระเงิน                             |
| data.request_detail.payermobile             | string       | ✓              | ✓                | เบอร์โทรศัพท์ผู้ชำระเงิน                          |
| data.request_detail.payername               | string       | ✓              | ✓                | ชื่อผู้ชำระเงิน                                   |
| data.request_detail.paymenttype             | string       | ✓              | ✗                | ประเภทการชำระเงิน                                 |
| data.response_detail                        | object       | ✓              | ✗                | รายละเอียดการตอบกลับ                              |
| data.response_detail.amount                 | number       | ✓              | ✗                | จำนวนเงิน                                         |
| data.response_detail.beneficiarybankaccount | string       | ✓              | ✓                | เลขที่บัญชีผู้รับเงิน                             |
| data.response_detail.beneficiarybankcode    | string       | ✓              | ✓                | รหัสธนาคารผู้รับเงิน                              |
| data.response_detail.beneficiaryname        | string       | ✓              | ✓                | ชื่อผู้รับเงิน                                    |
| data.response_detail.currency               | string       | ✓              | ✗                | สกุลเงิน                                          |
| data.response_detail.merchantrefid          | string       | ✓              | ✗                | รหัสอ้างอิงจากร้านค้า                             |
| data.response_detail.payerbankaccount       | string       | ✓              | ✓                | เลขที่บัญชีผู้ชำระเงิน                            |
| data.response_detail.payerbankcode          | string       | ✓              | ✓                | รหัสธนาคารผู้ชำระเงิน                             |
| data.response_detail.payermobile            | string       | ✓              | ✓                | เบอร์โทรศัพท์ผู้ชำระเงิน                          |
| data.response_detail.payername              | string       | ✓              | ✓                | ชื่อผู้ชำระเงิน                                   |
| data.response_detail.paymentstatus          | string       | ✓              | ✗                | สถานะการชำระเงิน                                  |
| data.response_detail.qrcode                 | string       | ✓              | ✓                | รหัส QR Code (base64)                             |
| data.response_detail.requesttimestamp       | string       | ✓              | ✗                | เวลาที่ร้องขอ (รูปแบบ ISO 8601)                   |
| data.response_detail.transactionid          | string       | ✓              | ✗                | รหัสธุรกรรม                                       |
| data.amount_sign                            | string       | ✓              | ✗                | เครื่องหมายจำนวนเงิน (+ รับเงิน, - จ่ายเงิน)      |
| data.amount                                 | number       | ✓              | ✗                | จำนวนเงิน                                         |
| data.currency                               | string       | ✓              | ✗                | สกุลเงิน                                          |
| data.status                                 | string       | ✓              | ✗                | สถานะการชำระเงิน (`PENDING`, `SUCCESS`)           |
| data.remark                                 | string       | ✓              | ✓                | หมายเหตุ                                          |
| data.bank_trans_id                          | string       | ✓              | ✓                | รหัสอ้างอิงธุรกรรมจากธนาคาร (จะมีก็เมื่อ success) |
| data.created_at                             | string       | ✓              | ✗                | เวลาที่สร้างรายการ (รูปแบบ ISO 8601)              |
| data.updated_at                             | string       | ✓              | ✗                | เวลาที่อัปเดตล่าสุด (รูปแบบ ISO 8601)             |

#### ตัวอย่างการใช้งาน API สำหรับการตรวจสอบสถานะการชำระเงิน

```bash
curl -X GET http://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/payment/{{transaction_id}} \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}"
```

#### ตัวอย่างการตอบกลับ check qrcode-payments

```json
{
	"status_code": 200,
	"message": "transaction retrieved successfully",
	"data": {
		"transaction_id": "67d40c59e2fa6060c648d0e3",
		"merchant_id": "67d2cb07a0b8290c3b123722",
		"merchant_ref_id": "MERCHANT123456",
		"for_staff_id": "",
		"type": "payin",
		"callback_url": "https://merchant.example.com/callback",
		"request_detail": {
			"amount": 10,
			"callbackurl": "https://merchant.example.com/callback",
			"currency": "THB",
			"merchantrefid": "MERCHANT123456",
			"payerbankaccount": "1234567890",
			"payerbankcode": "SCB",
			"payermobile": "0812345678",
			"payername": "John Doe",
			"paymenttype": "bank_account"
		},
		"response_detail": {
			"amount": 10,
			"beneficiarybankaccount": "2388868920",
			"beneficiarybankcode": "014",
			"beneficiaryname": "เอ็มเพย์ จ่ายดี",
			"currency": "THB",
			"merchantrefid": "MERCHANT123456",
			"payerbankaccount": "1234567890",
			"payerbankcode": "SCB",
			"payermobile": "0812345678",
			"payername": "John Doe",
			"paymentstatus": "PENDING",
			"qrcode": "",
			"requesttimestamp": "2025-03-14T18:00:41+07:00",
			"transactionid": "67d40c59e2fa6060c648d0e3"
		},
		"amount_sign": "+",
		"amount": 10,
		"currency": "THB",
		"status": "PENDING",
		"remark": "",
		"bank_trans_id": "",
		"created_at": "2025-03-14T11:00:41.969Z",
		"updated_at": "2025-03-14T11:00:41.974Z"
	}
}
```

### 3. Hook api callback สำหรับการแจ้งสถานะการชำระเงิน

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
| `data.payment_status`           | string       | ✓              | ✗                | สถานะการชำระเงิน (`SUCCESS` = สำเร็จ)   |
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
		"payment_status": "SUCCESS",
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
URL: `{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/merchants`

#### คำอธิบายพารามิเตอร์การตอบกลับจาก API สำหรับการดึงข้อมูลลูกค้า

| พารามิเตอร์             | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                     |
| ----------------------- | ------------ | -------------- | ---------------- | ---------------------------- |
| `status_code`           | number       | ✓              | ✗                | รหัสสถานะ HTTP               |
| `message`               | string       | ✓              | ✗                | ข้อความแสดงผลการทำงาน        |
| `data`                  | object       | ✓              | ✓                | ข้อมูลลูกค้า                 |
| `data.code`             | string       | ✓              | ✗                | รหัสลูกค้า                   |
| `data.api_key`          | string       | ✓              | ✗                | คีย์สำหรับเรียกใช้ API       |
| `data.api_secret`       | string       | ✓              | ✗                | รหัสลับสำหรับเรียกใช้ API    |
| `data.display_name`     | string       | ✓              | ✗                | ชื่อลูกค้า                   |
| `data.summary_balance`  | number       | ✓              | ✗                | ยอดเงินคงเหลือที่ใช้ได้      |
| `data.callback`         | array        | ✓              | ✗                | รายการ callback              |
| `data.callback.type`    | string       | ✓              | ✗                | ประเภทของ callback           |
| `data.callback.url`     | string       | ✓              | ✗                | URL สำหรับ callback          |
| `data.callback.method`  | string       | ✓              | ✗                | HTTP method สำหรับ callback  |
| `data.callback.headers` | string       | ✓              | ✓                | HTTP headers สำหรับ callback |
| `data.callback.body`    | string       | ✓              | ✓                | HTTP body สำหรับ callback    |

#### ตัวอย่างการเรียก API สำหรับการดึงข้อมูลลูกค้า

```bash
curl -X GET http://{{SERVER_IP}}:{{SERVER_PORT}}/api/v1/merchants \
     -H "Content-Type: application/json" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-api-secret: {{API_SECRET}}"
```

#### ตัวอย่างการตอบกลับ API สำหรับการดึงข้อมูลลูกค้า

```json
{
	"status_code": 200,
	"message": "merchant balance retrieved successfully",
	"data": {
		"code": "MPAY01",
		"api_key": "api_key_123123123",
		"api_secret": "api_secret_987123456",
		"display_name": "MPAY Merchant",
		"summary_balance": 198800.93,
		"callback": [
			{
				"type": "payin_failed",
				"url": "http://localhost:1880/api/payin_failed",
				"method": "POST",
				"headers": "",
				"body": ""
			},
			{
				"type": "payout_failed",
				"url": "http://localhost:1880/api/payout_failed",
				"method": "POST",
				"headers": "",
				"body": ""
			}
		]
	}
}
```

## 5. API สำหรับ Payout

### ธนาคารที่รองรับ

| bank_code | bank_name_th                         | bank_name_en                                       |
| --------- | ------------------------------------ | -------------------------------------------------- |
| 014       | ธนาคารไทยพาณิชย์ จำกัด (มหาชน)       | THE SIAM COMMERCIAL BANK PUBLIC COMPANY LIMITED    |
| 004       | ธนาคารกสิกรไทย จำกัด (มหาชน)         | KASIKORNBANK PUBLIC COMPANY LIMITED                |
| 006       | ธนาคารกรุงไทย จำกัด (มหาชน)          | KRUNG THAI BANK PUBLIC COMPANY LIMITED             |
| 002       | ธนาคารกรุงเทพ จำกัด (มหาชน)          | BANGKOK BANK PUBLIC COMPANY LIMITED                |
| 011       | ธนาคารทหารไทยธนชาต จำกัด (มหาชน)     | TMBTHANACHART BANK PUBLIC COMPANY LIMITED          |
| 030       | ธนาคารออมสิน                         | GOVERNMENT SAVINGS BANK                            |
| 025       | ธนาคารกรุงศรีอยุธยา จำกัด (มหาชน)    | BANK OF AYUDHAYA PUBLIC COMPANY LIMITED            |
| 034       | ธนาคารเพื่อการเกษตรและสหกรณ์การเกษตร | BANK FOR AGRICULTURE AND AGRICULTURAL COOPERATIVES |
| 024       | ธนาคารยูโอบี จำกัด (มหาชน)           | UNITED OVERSEAS BANK (THAI) PUBLIC COMPANY LIMITED |
| 033       | ธนาคารอาคารสงเคราะห์                 | GOVERNMENT HOUSTING BANK                           |
| 022       | ธนาคารซีไอเอ็มบี ไทย จำกัด (มหาชน)   | CIMB THAI BANK PUBLIC COMPANY LIMITED              |

หากมีอัพเดท จะแจ้งให้ทราบผ่านช่องทางติดต่อกัน

### 5.1 สร้าง Payout

#### คำอธิบาย API สำหรับ Payout body parameters json

| พารามิเตอร์                | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                                                   |
| -------------------------- | ------------ | -------------- | ---------------- | ---------------------------------------------------------- |
| `ref_id`                   | string       | ✓              | ✗                | รหัสอ้างอิงจากลูกค้า                                       |
| `amount`                   | number       | ✓              | ✗                | จำนวนเงินที่ต้องการโอน                                     |
| `currency`                 | string       | ✓              | ✗                | สกุลเงิน                                                   |
| `callback_url`             | string       | ✓              | ✗                | URL สำหรับรับการแจ้งเตือนผลการทำรายการ                     |
| `beneficiary_name`         | string       | ✓              | ✗                | ชื่อผู้รับเงิน                                             |
| `beneficiary_mobile`       | string       | ✓              | ✗                | เบอร์โทรศัพท์ผู้รับเงิน                                    |
| `beneficiary_bank_account` | string       | ✓              | ✗                | เลขที่บัญชีปลายทาง                                         |
| `beneficiary_bank_code`    | string       | ✓              | ✗                | รหัสธนาคารปลายทาง Code จาก [ตารางรหัสธนาคาร](#8-bank-code) |

### ตัด `timestamp` ออก

#### คำอธิบายพารามิเตอร์การตอบกลับจาก API สำหรับการสร้าง Payout

| พารามิเตอร์                     | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                               |
| ------------------------------- | ------------ | -------------- | ---------------- | -------------------------------------- |
| `status_code`                   | number       | ✓              | ✗                | รหัสสถานะ HTTP                         |
| `message`                       | string       | ✓              | ✗                | ข้อความแสดงผลการทำงาน                  |
| `data`                          | object       | ✓              | ✗                | ข้อมูลการทำรายการ                      |
| `data.transaction_id`           | string       | ✓              | ✗                | รหัสอ้างอิงการทำรายการ Payout          |
| `data.ref_id`                   | string       | ✓              | ✗                | รหัสอ้างอิงจากลูกค้า                   |
| `data.for_staff_id`             | string       | ✓              | ✓                | รหัสพนักงานที่ทำรายการ (ถ้ามี)         |
| `data.amount`                   | number       | ✓              | ✗                | จำนวนเงินที่ต้องการโอน                 |
| `data.currency`                 | string       | ✓              | ✗                | สกุลเงิน                               |
| `data.beneficiary_name`         | string       | ✓              | ✗                | ชื่อผู้รับเงิน                         |
| `data.beneficiary_bank_account` | string       | ✓              | ✗                | เลขที่บัญชีปลายทาง                     |
| `data.beneficiary_bank_code`    | string       | ✓              | ✗                | รหัสธนาคารปลายทาง                      |
| `data.request_timestamp`        | string       | ✓              | ✗                | เวลาที่ทำรายการ (RFC3339)              |
| `data.payment_status`           | string       | ✓              | ✗                | สถานะการทำรายการ (QUEUE, SUCCESS, FAILED) |

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
		"beneficiary_name": "ทดสอบ การโอนเงิน",
		"beneficiary_mobile": "0891234567",
		"beneficiary_bank_account": "5557774445",
		"beneficiary_bank_code": "025",
		"timestamp": "2025-02-28T16:23:00Z"
	}
'
```

#### ตัวอย่างการตอบกลับ API สำหรับการสร้าง Payout

```json
{
	"status_code": 201,
	"message": "payout transaction created successfully",
	"data": {
		"transaction_id": "67dab886c803f64e2db3b7a4",
		"ref_id": "MERCH123456789",
		"for_staff_id": "",
		"amount": 10,
		"currency": "THB",
		"beneficiary_name": "John Doe",
		"beneficiary_bank_account": "1234567890",
		"beneficiary_bank_code": "004",
		"request_timestamp": "2025-03-19T19:28:54+07:00",
		"payment_status": "QUEUE"
	}
}
```

### 5.2 Callback ผลการทำรายการ Payout

หากมีการทำรายการ Payout สำเร็จ จะมีการส่งข้อมูลกลับมาที่ URL ที่ระบุไว้ในพารามิเตอร์ `callback_url`

Method: `POST`<br/>
URL: `{{HOOK_CALLBACK_PAYOUT_URL}}`

#### คำอธิบาย hook callback payout

| พารามิเตอร์                     | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย                       |
| ------------------------------- | ------------ | -------------- | ---------------- | ------------------------------ |
| `status_code`                   | number       | ✓              | ✗                | รหัสสถานะ HTTP                 |
| `message`                       | string       | ✓              | ✗                | ข้อความแสดงผลการทำงาน          |
| `data`                          | object       | ✓              | ✗                | ข้อมูลการทำรายการ              |
| `data.payment_status`           | string       | ✓              | ✗                | สถานะการทำรายการ               |
| `data.transaction_id`           | string       | ✓              | ✗                | รหัสธุรกรรม                    |
| `data.ref_id`                   | string       | ✓              | ✗                | รหัสอ้างอิงธุรกรรม             |
| `data.amount`                   | number       | ✓              | ✗                | จำนวนเงินที่ร้องขอ             |
| `data.currency`                 | string       | ✓              | ✗                | สกุลเงิน                       |
| `data.description`              | string       | ✓              | ✗                | รายละเอียดการทำรายการจากธนาคาร |
| `data.payment_date`             | string       | ✓              | ✗                | วันที่ทำรายการจากธนาคาร        |
| `data.payment_amount`           | number       | ✓              | ✗                | จำนวนเงินที่ทำรายการจากธนาคาร  |
| `data.beneficiary_name`         | string       | ✓              | ✗                | ชื่อบัญชีปลายทางจากคำขอ        |
| `data.beneficiary_mobile`       | string       | ✓              | ✗                | เบอร์โทรศัพท์ผู้รับเงินจากคำขอ |
| `data.beneficiary_bank_account` | string       | ✓              | ✗                | บัญชีปลายทางจากคำขอ            |
| `data.beneficiary_bank_code`    | string       | ✓              | ✗                | รหัสธนาคารปลายทางจากคำขอ       |
| `data.request_timestamp`        | string       | ✓              | ✗                | เวลาที่ทำรายการ (RFC3339)      |

#### ตัวอย่างการตอบกลับ Callback ผลการทำรายการ Payout

```json
{
	"status_code": 200,
	"message": "Success",
	"data": {
		"payment_status": "SUCCESS",
		"transaction_id": "67dab886c803f64e2db3b7a4",
		"ref_id": "MERCH123456789",
		"amount": 1000,
		"currency": "THB",
		"description": "โอนเงินให้ ทดสอบ การโอนเงิน",
		"payment_date": "2025-02-28T16:23:00Z",
		"payment_amount": 1000,
		"beneficiary_name": "ทดสอบ การโอนเงิน",
		"beneficiary_mobile": "0891234567",
		"beneficiary_bank_account": "5557774445",
		"beneficiary_bank_code": "025",
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
