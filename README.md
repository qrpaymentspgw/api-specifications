# คู่มือการเชื่อมต่อ API การสร้าง QR Payment v1.0.0

## 1. บทนำ

เมื่อท่านสมัครการใช้งานจะได้รับข้อมูลการเข้าถึง API จากเรา และท่านต้องทำการตั้งค่า Hook api callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน

และท่านจะได้ข้อมูลโดยมีข้อมูลดังนี้

| ชื่อพารามิเตอร์ | ประเภท | คำอธิบาย |
|---------------|------|---------|
| `SERVER_IP` | string | Server endpoint ip |
| `SERVER_PORT` | string | Server endpoint port |
| `API-KEY` | string | API key เพื่อระบุตัวตนของผู้ใช้งาน |
| `API-SECRET` | string | API secret เพื่อระบุตัวตนของผู้ใช้งาน |

และสามารถเริ่มใช้งานได้ทันที

1. API สำหรับการสร้าง QR Payment ช่วยให้คุณสามารถสร้าง QR Code สำหรับการชำระเงินได้ง่าย ๆ ผ่าน HTTP Request โดยรองรับรูปแบบการชำระเงินแบบ PromptPay QR Code และการโอนผ่านเลขที่บัญชี
2. API สำหรับการตรวจสอบสถานะการชำระเงิน ช่วยให้คุณสามารถตรวจสอบสถานะการชำระเงินได้ง่าย ๆ ผ่าน HTTP Request โดยส่ง `payment_id` ที่ได้จากการสร้าง QR Payment

## 2. วิธีการเรียก API

### Enpoint

- URL: `https://{{SERVER_IP}}:{{SERVER_PORT}}/`

### 2.1 API สำหรับการสร้าง QR Payment

Method: `POST`<br/>
URL: `/api/qrcode-payments`

#### Headers

```json
{
  "Content-Type": "application/json",
  "x-api-key": "{{YOUR_API_KEY}}",
  "x-api-secret": "{{YOUR_API_SECRET}}"
}
```

#### Request Body

```json
{
  "payment_type": "promptpay", 
  "amount": 100,
  "format": "base64"
}
```

#### Body parameters

| ชื่อพารามิเตอร์ | ประเภท | จำเป็น | คำอธิบาย |
|---------------|------|--------|---------|
| `payment_type` | string | ✓ | ประเภทของ QR Payment (`promptpay`) |
| `amount` | float | ✓ | จำนวนเงินที่ต้องการชำระ (ใส่ `1` บาทขึ้นไป) |
| `format` | string | ✓ | รูปแบบของไฟล์ (`base64`) |

#### ตัวอย่างการใช้งาน cURL Example

```bash
curl -X POST https://{{SERVER_IP}}:{{SERVER_PORT}}/api/qrcode-payments \
     -H "Content-Type: application/json" \
     -H "x-api-key: {{YOUR_API_KEY}}" \
     -H "x-api-secret: {{YOUR_API_SECRET}}" \
     -d '{
       "payment_type": "promptpay",
       "amount": 100,
       "format": "base64"
     }' 
```

#### ตัวอย่างการตอบกลับ

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
  "message": "คำขอไม่ถูกต้อง (ตรวจสอบค่าพารามิเตอร์)"
}
```

---

### 2.2 API สำหรับการตรวจสอบสถานะการชำระเงิน

Method: `GET`<br/>
URL: `/api/qrcode-payments/{{payment_id}}`

#### พารามิเตอร์ที่ส่งมาในการเรียกใช้งาน

| พารามิเตอร์ | ประเภทข้อมูล | จำเป็นต้องระบุ | คำอธิบาย |
|-----------|------------|--------------|----------|
| payment_id | string | ✓ | รหัสอ้างอิงการชำระเงินที่ต้องการตรวจสอบ |

#### พารามิเตอร์ที่ส่งกลับ

| พารามิเตอร์ | ประเภทข้อมูล | คำอธิบาย |
|-----------|------------|----------|
| status | string | สถานะการตรวจสอบ <br/>(`success` = สำเร็จ, `pending` = รอดำเนินการ, `failed` = ล้มเหลว) |
| payment_id | string | รหัสอ้างอิงการชำระเงิน |
| amount | number | จำนวนเงินที่ชำระ |
| description | string | รายละเอียดการชำระเงิน |
| payment_date | string | วันเวลาที่ชำระเงิน (รูปแบบ ISO 8601) |

#### ตัวอย่างการใช้งาน cURL Example

```bash
curl -X GET https://{{SERVER_IP}}:{{SERVER_PORT}}/api/qrcode-payments/1234567890 \
     -H "x-api-key: {{YOUR_API_KEY}}" \
     -H "x-api-secret: {{YOUR_API_SECRET}}"
```

#### ตัวอย่างการตอบกลับ

```json
{
  "status": "success",
  "payment_id": "1234567890",
  "amount": 100.25,
  "description": "PromptPay x7878 นาย ทดสอบ ระบบ",
  "payment_date": "2025-01-20T17:34:00+07:00",
}
```

### 3. Hook api callback สำหรับการแจ้งตรวจสอบสถานะการชำระเงิน

เมื่อมีการชำระเงินจะมีการส่งข้อมูลกลับมาที่ URL ที่ท่านที่กำหนดไว้ ตามรายละเอียดด้านล่าง

Method: `POST`<br/>
URL: `{{HOOK_CALLBACK_URL}}`

#### ตัวอย่างการตอบกลับ

```json
{
  "status": "success",
  "payment_id": "1234567890",
  "amount": 100.25,
  "description": "PromptPay x7878 นาย ทดสอบ ระบบ",
  "payment_date": "2025-01-20T17:34:00+07:00",
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
  "message": "คำขอไม่ถูกต้อง (ตรวจสอบค่าพารามิเตอร์)"
}
```

---

## 4. ติดต่อฝ่ายสนับสนุน

หากพบปัญหาในการใช้งาน API กรุณาติดต่อทีมสนับสนุนที่อีเมล qrpayments.pgw@gmail.com
