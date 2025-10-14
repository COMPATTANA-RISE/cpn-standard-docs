# มาตรฐานการตั้งชื่อ (Naming Convention)

## ตัวแปร (Variables)
- ใช้ **camelCase** สำหรับชื่อตัวแปรและฟังก์ชัน  
  ตัวอย่าง: `userName`, `createdAt`

## ค่าคงที่ (Constants)
- ใช้ **UPPER_SNAKE_CASE**  
  ตัวอย่าง: `MAX_RETRY`, `API_TIMEOUT`

## ไฟล์และโฟลเดอร์
- ใช้ **kebab-case** สำหรับชื่อไฟล์และโฟลเดอร์  
  ตัวอย่าง: `user-service`, `auth-controller.ts`

## ฐานข้อมูล (Database)
- ใช้ **snake_case** สำหรับชื่อตารางและคอลัมน์  
  ตัวอย่าง: `user_profiles`, `created_at`
- Primary key → `id`
- Foreign key → `<table>_id`
