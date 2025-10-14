# ความปลอดภัย (Security & Compliance)

## การตรวจสอบข้อมูลเข้า (Input Validation)
- ตรวจฝั่ง backend ทุกครั้ง
- ใช้ schema กลาง เช่น Zod หรือ Joi

## การยืนยันตัวตน (Authentication)
- ใช้ **JWT** หรือ **OAuth2**
- ต้องมีระบบ refresh token

## การอนุญาตสิทธิ์ (Authorization)
- ใช้ **RBAC/ABAC**
- ห้ามเชื่อถือ role จาก client

## ความเป็นส่วนตัว (Data Privacy)
- ซ่อนหรือเข้ารหัสข้อมูลสำคัญใน log
- ปฏิบัติตาม PDPA/GDPR

## ความปลอดภัยของการเชื่อมต่อ (Transport Security)
- บังคับใช้ HTTPS
- ปฏิเสธการเชื่อมต่อที่ไม่ปลอดภัย
