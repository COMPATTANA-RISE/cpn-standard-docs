# ความปลอดภัย (Security & Compliance)

## การตรวจสอบข้อมูลเข้า (Input Validation)
- ตรวจสอบ input ทุกครั้งในฝั่ง backend  
- ใช้ schema กลาง เช่น **Zod**
- ต้อง reject input ที่ไม่ผ่านการ validate ก่อนถึง business logic
- จำกัดขนาดของ body และ payload เพื่อป้องกัน DDoS

---

## การยืนยันตัวตน (Authentication)
- ใช้ **JWT**, **OAuth2**, หรือ **Sanctum** (กรณี Laravel)
- กำหนดอายุ token ชัดเจน (`access_token` สั้น, `refresh_token` ยาว)
- เก็บ refresh token ใน Secure HTTP-Only Cookie เท่านั้น
- ห้ามเก็บ token ใน localStorage หรือ URL
- ทุก API Endpoint ต้องตรวจสอบ token validator ทุกครั้ง (ยกเว้น open api **แต่ต้องไม่มีข้อมูลที่ละเอียดอ่อน**)
- หากใช้ session ต้องผูก session กับ IP / User-Agent (Laravel กรณีทำ Custom Rate Limit)

---

## การอนุญาตสิทธิ์ (Authorization)
- ใช้โมเดล **RBAC** (Role-Based Access Control) หรือ **ABAC** (Attribute-Based Access Control)
- ตรวจสิทธิ์ในทุก endpoint ฝั่ง backend
- ห้ามเชื่อถือข้อมูล role/permission จากฝั่ง client
- ทุก resource ต้องตรวจสอบ “สิทธิ์เฉพาะเจ้าของ” (owner-based access)
- ใช้ Policy หรือ Guard กลาง (middleware / decorator)

---

## ความเป็นส่วนตัวของข้อมูล (Data Privacy)
- เข้ารหัสข้อมูลที่ละเอียดอ่อน เช่น เลขบัตรประชาชน เบอร์โทร รหัสผ่าน
- Hash รหัสผ่านด้วย **bcrypt**, **argon2**, หรือเทียบเท่า
- ไม่ log ข้อมูลที่มีความละเอียดอ่อน เช่น password, token, หรือข้อมูลทางการแพทย์
- ปฏิบัติตาม **PDPA/GDPR**
  - ต้องขอ consent ชัดเจน
  - ผู้ใช้มีสิทธิ์ “ลบข้อมูล” ได้ตามกฎหมาย
- ใช้ `mask()` หรือ `redact()` ใน log viewer สำหรับข้อมูลสำคัญ

---

## ความปลอดภัยของการเชื่อมต่อ (Transport Security)
- บังคับใช้ **HTTPS** ทั้ง UAT และ Production
- ใช้ SSL/TLS certificate ที่ถูกต้องจาก CA (เช่น Let’s Encrypt)
- ปฏิเสธ request ที่ไม่ปลอดภัย เช่น HTTP ธรรมดา
- ป้องกัน **MITM (Man-In-The-Middle)** attack ด้วย HSTS
- หากใช้ WebSocket หรือ MQTT ให้ใช้ **wss://** หรือ **mqtts://**

---

## การเข้ารหัส (Encryption)
- ใช้ **AES-256-GCM** สำหรับข้อมูลที่ต้องเข้ารหัส
- ห้าม hardcode key, password, หรือ credential ในโค้ด

---

## การจัดการ Token และ Session
- JWT ต้องมี claim อย่างน้อย: `sub`, `iat`, `exp`
- ทุก request ที่ต้อง auth ต้องแนบ `Authorization: Bearer <token>`
- หาก logout → blacklist token หรือเปลี่ยน signing key
- ใช้ short-lived token (เช่น 15 นาที)
- Refresh token ต้องตรวจสอบ IP / User-Agent ก่อนออกใหม่

---

## การเก็บ Log (Logging Security)
- Log ต้องไม่เก็บ PII (Personal Identifiable Information)
- มีการหมุน log (log rotation) อัตโนมัติ
- ใช้ระบบ log กลาง เช่น ELK, Loki, หรือ Datadog
- ต้องมี traceId หรือ requestId สำหรับทุก request
- Log error แยกจาก access log

---

## การอัปโหลดไฟล์ (File Security)
- ตรวจ MIME type และนามสกุลไฟล์ก่อนบันทึก
- จำกัดขนาดไฟล์ (`max 10MB` สำหรับภาพ / `20MB` สำหรับเอกสาร)
- เปลี่ยนชื่อไฟล์ก่อนเก็บ (UUID หรือ hash)
- ห้ามเก็บไฟล์ใน public folder โดยตรง
- เก็บใน Storage ปลอดภัย เช่น **MinIO**, **S3**, หรือ **Cloudflare R2**
- ป้องกันการ execute script จากไฟล์ เช่น `.php`, `.js`

---

## ความปลอดภัยของ Dependency (Dependency Security)
- ใช้เวอร์ชันที่เสถียรของ package
- ตรวจสอบช่องโหว่ด้วย `npm audit` หรือ `composer audit`
- ห้ามใช้ library ที่ไม่มี maintain หรือไม่มี license ชัดเจน
- ล็อก dependency ด้วย `package-lock.json` / `composer.lock` / `pnpm-lock.yaml`

---

## ความปลอดภัยในระดับ Infrastructure
- จำกัดสิทธิ์การเข้าถึงเซิร์ฟเวอร์ด้วย SSH key เท่านั้น
- ใช้ Firewall / Security Group กำหนดพอร์ตที่อนุญาต
- ปิดบริการที่ไม่จำเป็นในเครื่อง production
- สำรองข้อมูลอัตโนมัติและทดสอบ restore เป็นระยะ
- ใช้ระบบ Monitor เช่น Grafana, Prometheus, Zabbix
- ตรวจสอบ log การเข้าถึงระบบ (audit log)

---

## การจัดการ Incident (Incident Response)
- เมื่อเกิดเหตุ security breach ต้อง:
  1. ปิดการเข้าถึงทันที
  2. แจ้งทีม security ภายใน 1 ชั่วโมง
  3. เก็บหลักฐาน (log, trace, request)
  4. วิเคราะห์ root cause และออก patch
  5. แจ้งผลให้ผู้เกี่ยวข้อง
- บันทึกทุกขั้นตอนลงใน Incident Report

---

## สรุปแนวทางทีม
- Security ต้องถูก design-in ตั้งแต่เริ่ม ไม่ใช่แก้ภายหลัง  
- ใช้ principle: **“Least Privilege”** และ **“Zero Trust”**  
- มีการ review code ด้าน security ก่อน release ทุกครั้ง  
- ตั้ง CI/CD scan ด้วย SAST/DAST เช่น SonarQube, Trivy, Snyk  
- เก็บ log ของทุก API พร้อม trace ID เพื่อสอบสวนภายหลัง
