# รูปแบบ Request (Request Format)

## กฎทั่วไป
- ใช้ **JSON** สำหรับทุก API payload
- ใช้ key แบบ **camelCase**
- ไม่ส่ง key ที่มีค่า `null` หากไม่จำเป็น

### ตัวอย่าง
```json
{
  "userId": "u001",
  "email": "user@example.com",
  "isActive": true
}
```

## Query Parameters
- ใช้ตัวพิมพ์เล็กและ snake_case ใน URL  
  ตัวอย่าง: `/api/v1/users?status=active&page=1`

## Headers
- `Content-Type: application/json`
- `Accept: application/json`
- `Authorization: Bearer <token>`
- `X-Request-ID: <uuid>` → สำหรับติดตาม log ระหว่างระบบ
- `X-Client-Version: <semver>` → แสดงเวอร์ชันของ client ที่ส่งคำขอ

## อนุญาต HTTP Methods

| Method | ใช้สำหรับ | หมายเหตุ |
|---------|-------------|-----------|
| **GET** | ดึงข้อมูล | ห้ามเปลี่ยนสถานะของระบบ |
| **POST** | สร้างข้อมูลใหม่ | ใช้กับ resource ใหม่ เช่น `POST /users` |
| **PUT** | อัปเดตข้อมูลทั้ง object | ต้องส่งข้อมูลครบทั้งหมด |
| **PATCH** | อัปเดตข้อมูลบางส่วน | ใช้เมื่อแก้บาง field |
| **DELETE** | ลบข้อมูล | ควรตอบ `204 No Content` |
| **HEAD** | ตรวจ header เท่านั้น | ไม่ส่ง body กลับ |
| **OPTIONS** | แจ้ง method ที่ endpoint รองรับ | ใช้กับ preflight request (CORS) |

### แนวทางทีม
- Backend ต้องตรวจสอบว่า endpoint รองรับเฉพาะ method ที่กำหนด (405 หากไม่ตรง)
- Frontend ต้องใช้ method ให้ตรงกับวัตถุประสงค์ เช่น:
  - `GET /users/1`
  - `POST /users`
  - `PUT /users/1`
  - `PATCH /users/1`
  - `DELETE /users/1`
- ห้ามใช้ `GET` เพื่อเปลี่ยนข้อมูล (เช่น toggle หรือ delete)


