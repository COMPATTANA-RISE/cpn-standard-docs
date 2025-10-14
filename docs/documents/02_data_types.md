# ประเภทข้อมูล (Data Types)

| ประเภท | มาตรฐาน | หมายเหตุ |
|------|-----------|-----------|
| **String** | UTF-8 | ตัดช่องว่างก่อนบันทึก |
| **Number** | int / float / decimal | ใช้ decimal สำหรับจำนวนเงิน |
| **Boolean** | true / false | หลีกเลี่ยงการใช้ 0/1 |
| **DateTime** | ISO8601 (`YYYY-MM-DDTHH:mm:ssZ`) | เก็บเป็น UTC แปลงเป็นเวลาท้องถิ่นตอนส่งให้ client |
| **Enum** | เก็บเป็น string | เช่น `"PENDING"`, `"APPROVED"` |
| **Array/Object** | JSON schema | กำหนดโครงสร้างชัดเจน |
