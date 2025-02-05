# คู่มือการเชื่อมต่อ API การสร้าง QR Payment

## 1. บทนำ

เมื่อท่านสมัครการใช้งานจะได้รับข้อมูลการเข้าถึง API จากเรา และท่านต้องทำการตั้งค่า Hook api callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน

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

Method: `POST`<br/>
URL: `/api/v1/qrcode-payments`

#### คำอธิบาย qrcode-payments body parameters json

| ชื่อพารามิเตอร์ | ประเภท | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย |
|---------------|------|--------|-----------------|---------|
| `payment_type` | string | ✓ | ✗ | ประเภทของ QR Payment (`promptpay`) |
| `amount` | integer | ✓ | ✗ | จำนวนเงินที่ต้องการชำระ (ใส่ `1` บาทขึ้นไป) |
| `format` | string | ✓ | ✗ | รูปแบบของไฟล์ (`base64`) |

#### คำอธิบาย qrcode-payments response

| ชื่อพารามิเตอร์ | ประเภท | จำเป็น | อนุญาตให้ว่างได้ | คำอธิบาย |
|---------------|------|--------|-----------------|---------|
| `status` | string | ✓ | ✗ | สถานะการตรวจสอบ <br/>(`success` = สำเร็จ, `failed` = ล้มเหลว) |
| `message` | string | ✓ | ✗ | ข้อความที่อธิบายสถานะการตรวจสอบ |
| `data` | object | ✓ | ✗ | ข้อมูลการชำระเงิน |
| `data.payment_id` | string | ✓ | ✗ | รหัสอ้างอิงการชำระเงิน |
| `data.amount` | float | ✓ | ✗ | จำนวนเงินที่ชำระ |
| `data.qrcode_base64` | string | ✓ | ✗ | ข้อมูล QR Code ในรูปแบบ Base64 |

#### ตัวอย่างการใช้งาน API สำหรับการสร้าง QR Payment

```bash
curl -X POST https://{{SERVER_IP}}:{{SERVER_PORT}}/api/qrcode-payments \
     -H "Content-Type: application/json" \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-cust-code: {{CUST_CODE}}" \
     -H "x-api-secret: {{API_SECRET}}" \
     -d '{
       "payment_type": "promptpay",
       "amount": 100,
       "format": "base64"
     }' 
```

#### ตัวอย่างการตอบกลับ qrcode-payments

```json
{
  "status": 200,
  "message": "QR Code created successfully",
  "data": {
    "payment_id": "1234567890",
    "amount": 100.25,
    "qrcode_base64": "iVBORw0KGgoAAAANSUhEUgAA..."
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
  "status": 400,
  "message": "คำขอไม่ถูกต้อง (ตรวจสอบค่าพารามิเตอร์)",
  "data": null
}
```

---

### 2.2 API สำหรับการตรวจสอบสถานะการชำระเงิน

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
curl -X GET https://{{SERVER_IP}}:{{SERVER_PORT}}/api/qrcode-payments/1234567890 \
     -H "x-api-key: {{API_KEY}}" \
     -H "x-cust-code: {{CUST_CODE}}" \
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
    "amount": 100.25,
    "description": "PromptPay x7878 นาย ทดสอบ ระบบ",
    "payment_date": "2025-01-20T17:34:00+07:00"
  }
}
```

### 3. Hook api callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน

เมื่อมีการชำระเงินจะมีการส่งข้อมูลกลับมาที่ URL ที่ท่านที่กำหนดไว้ ตามรายละเอียดด้านล่าง

#### คำอธิบาย

| พารามิเตอร์ | ประเภทข้อมูล | จำเป็นต้องระบุ | อนุญาตให้ว่างได้ | คำอธิบาย |
|-----------|------------|--------------|----------------|----------|
| status | string | ✓ | ✗ | สถานะการตรวจสอบ <br/>(`success` = สำเร็จ, `failed` = ล้มเหลว) |
| data | object | ✓ | ✓ | ข้อมูลการชำระเงิน |
| data.status | string | ✓ | ✗ | สถานะการตรวจสอบ <br/>(`DONE` = สำเร็จ, `PENDING` = รอดำเนินการ, `CANCEL` = ยกเลิก) |
| data.ref_id | string | ✓ | ✗ | รหัสอ้างอิงการชำระเงิน |
| data.amount | number | ✓ | ✗ | จำนวนเงินที่ชำระ |
| data.description | string | ✗ | ✓ | รายละเอียดการชำระเงิน |
| data.payment_date | string | ✗ | ✓ | วันเวลาที่ชำระเงิน (รูปแบบ ISO 8601) |

Method: `POST`<br/>
URL: `{{HOOK_CALLBACK_URL}}`

#### ตัวอย่างการตอบกลับ

```json
{
  "status": "success",
  "message": "QR Code received successfully",
  "data": {
    "status": "DONE",
    "ref_id": "1234567890",
    "amount": 100.25,
    "description": "PromptPay x7878 นาย ทดสอบ ระบบ",
    "payment_date": "2025-01-20T17:34:00+07:00"
  }
}
```

---

## 4. ติดต่อฝ่ายสนับสนุน

หากพบปัญหาในการใช้งาน API กรุณาติดต่อทีมสนับสนุนที่อีเมล <qrpayments.pgw@gmail.com>
