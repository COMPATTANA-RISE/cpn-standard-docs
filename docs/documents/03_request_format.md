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
