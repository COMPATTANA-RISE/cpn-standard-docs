# การจัดการเวอร์ชัน API (Versioning)

- ทุก route ควรมี prefix version → `/api/v1/...`
- เวอร์ชันเก่าควรถูก deprecate แบบค่อยเป็นค่อยไป
- รองรับ backward compatibility อย่างน้อย 6 เดือน
- เขียน changelog ทุกครั้งที่มีการเปลี่ยน API

## การจัดการ Migration ฐานข้อมูล
- ใช้เครื่องมือ migration กลาง (Prisma, Liquibase)
- ต้องมีแผน rollback เสมอ
