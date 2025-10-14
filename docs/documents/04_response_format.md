# รูปแบบ Response (Response Format)

## Response เมื่อสำเร็จ
```json
{
  "success": true,
  // "statusCode": 0, // custom status code (ถ้ามี) ถ้าไม่มี ให้ return http status code เช่น 200
  "message": "fetch data successfuly",
  "data": {
    "userId": "u001",
    "email": "user@example.com"
  },
  "meta": {
    "requestId": "123e4567-e89b-12d3-a456-426614174000"
  }
}
```

## Response เมื่อเกิดข้อผิดพลาด
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "กรุณากรอกอีเมล",
    "details": {
      "field": "email"
    }
  }
}
```

## รูปแบบ Pagination มาตรฐาน
```json
{
  "data": [ ... ],
  "page": 1,
  "limit": 20,
  "total": 120
}
```

## รหัสสถานะ HTTP (ใช้จริงในทีม)
| รหัส | ความหมาย | ใช้เมื่อ |
|------|-----------|-----------|
| **200 OK** | สำเร็จ | อ่านหรืออัปเดตข้อมูลได้สำเร็จ |
| **201 Created** | สร้างข้อมูลสำเร็จ | POST บันทึกใหม่ |
| **202 Accepted** | เข้าคิวประมวลผล | ทำงาน async |
| **204 No Content** | สำเร็จแต่ไม่มีข้อมูล | ลบหรือตั้งค่า toggle |
| **304 Not Modified** | cache ยังใช้ได้ | ใช้กับ ETag |
| **400 Bad Request** | รูปแบบไม่ถูกต้อง | body/query ผิด |
| **401 Unauthorized** | ไม่มีสิทธิ์ / token หมด | ต้อง login ใหม่ |
| **403 Forbidden** | มี token แต่ไม่มีสิทธิ์ | role ไม่พอ |
| **404 Not Found** | ไม่พบข้อมูล | id หรือ route ผิด |
| **409 Conflict** | ข้อมูลซ้ำหรือชนกัน | รหัสซ้ำ / version conflict |
| **422 Validation Error** | ข้อมูลไม่ผ่าน rule | เช่น กรอกไม่ครบ |
| **429 Too Many Requests** | เกิน rate limit | เรียกถี่เกิน |
| **500 Internal Server Error** | ระบบล้ม | exception ภายใน |
| **502 Bad Gateway** | ปลายน้ำล่ม | microservice error |
| **503 Service Unavailable** | ปิดปรับปรุง / โหลดสูง | service down |
| **504 Gateway Timeout** | ปลายน้ำไม่ตอบ | timeout |
