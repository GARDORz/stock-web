# 📦 Stockify — Pro Inventory Management

ระบบจัดการสต็อกสินค้าออนไลน์ (Web-based Inventory System) สำหรับธุรกิจ SME รองรับการจัดการสินค้า, สต็อก, ออเดอร์ และประวัติการทำรายการ พร้อมระบบล็อกอินผ่าน AWS Cognito

---

## Features

- **Dashboard** — สรุปภาพรวม: จำนวนสินค้า, สินค้าใกล้หมด, มูลค่าสต็อกรวม พร้อมกราฟหมวดหมู่และ Top 5 มูลค่าสูงสุด
- **Products** — เพิ่ม / แก้ไข / ลบสินค้า (รองรับรูปภาพ Base64)
- **Stock** — รับเข้า (IN) และเบิกออก (OUT) พร้อมแจ้งเตือนเมื่อสต็อกต่ำกว่าจุดสั่งซื้อ
- **Orders** — บริหารออเดอร์ประเภท PO (Purchase Order), SO (Sales Order) และ Return
- **History** — ประวัติการเข้า-ออกสต็อกทุกรายการ
- **Audit Log** — บันทึกการกระทำทั้งหมดในระบบ
- **Dark Mode** — สลับธีมมืด/สว่างได้ (บันทึกค่าผ่าน LocalStorage)
- **Export CSV** — ดาวน์โหลดข้อมูลสินค้าทั้งหมดเป็น CSV
- **Role-based Access** — Admin เท่านั้นที่แก้ไข/ลบสินค้าได้

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3 (CSS Variables), Vanilla JavaScript |
| Authentication | AWS Cognito (amazon-cognito-identity-js) |
| Backend / API | AWS API Gateway + Lambda |
| Charts | Chart.js |
| Icons | Phosphor Icons |
| Alerts | SweetAlert2 |

---

## Project Structure

```
Stockify-Web/
├── index.html      # หน้าหลัก (Dashboard + ทุก View)
├── login.html      # หน้าเข้าสู่ระบบ
├── app.js          # Logic ทั้งหมด (fetch API, render, events)
├── auth.js         # Auth helper สำหรับ AWS Cognito (legacy)
└── style.css       # Stylesheet + Dark Mode
```

---

## Getting Started

### Prerequisites

- XAMPP หรือ Web server ใด ๆ ที่เสิร์ฟไฟล์ Static ได้
- บัญชี AWS พร้อม Cognito User Pool และ API Gateway ที่ตั้งค่าไว้แล้ว

### Setup

1. Clone หรือวางไฟล์ทั้งหมดไว้ใน `htdocs/Stockify-Web/`
2. เปิดไฟล์ [app.js](app.js) และแก้ไขค่าต่อไปนี้ให้ตรงกับ AWS ของคุณ:

```js
const API_URL      = "https://<your-api-id>.execute-api.<region>.amazonaws.com/default/Stockify-GetProducts";
const USER_POOL_ID = '<your-user-pool-id>';
const CLIENT_ID    = '<your-client-id>';
const ADMIN_EMAIL  = "<your-admin-email>";
```

3. เปิด `login.html` ใน [`login.html`](login.html) และตรวจสอบค่า `UserPoolId` / `ClientId` ให้ตรงกัน
4. เปิดเบราว์เซอร์ไปที่ `http://localhost/Stockify-Web/login.html`

---

## AWS Architecture

```
Browser
  │
  ├─► AWS Cognito          (Authentication / JWT Token)
  │
  └─► API Gateway
          │
          └─► Lambda Function  (GET / POST / DELETE สินค้าและ Transaction)
                  │
                  └─► DynamoDB (ฐานข้อมูลสินค้าและ History)
```

### API Endpoints (ผ่าน Lambda เดียว)

| Method | Body / Query | Action |
|---|---|---|
| `GET` | — | ดึงรายการสินค้าทั้งหมด |
| `GET` | `?table=history` | ดึงประวัติ Transaction |
| `POST` | `{ sku, name, ... }` | เพิ่ม / แก้ไขสินค้า |
| `POST` | `{ action: "TRANSACTION", ... }` | บันทึก In/Out / Order |
| `DELETE` | `{ sku }` | ลบสินค้า |

---

## Role Management

- **Admin** (`ADMIN_EMAIL` ใน `app.js`): เห็นปุ่ม Edit / Delete และปุ่ม New Item
- **User ทั่วไป**: ดูข้อมูล, ทำ In/Out สต็อก และสร้างออเดอร์ได้ แต่ไม่สามารถแก้ไข/ลบสินค้าได้

---

## Order Types

| Type | ความหมาย | ผลต่อสต็อก |
|---|---|---|
| PO (Purchase Order) | สั่งซื้อสินค้าเข้า | +stock |
| SO (Sales Order) | ขายสินค้าออก | −stock |
| Return | ลูกค้าคืนสินค้า | +stock |
