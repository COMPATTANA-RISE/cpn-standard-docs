# มาตรฐาน Frontend สำหรับป้องกันข้อมูลซ้ำ (Double Submit Prevention)

> ใช้สำหรับทุกหน้า/ฟอร์ม ที่มีการส่งข้อมูลไปยัง API เพื่อป้องกันการสร้างข้อมูลซ้ำจากการกดปุ่มหรือยิง request ซ้ำ

---

## 🎯 เป้าหมาย
- ป้องกันผู้ใช้กด Submit ซ้ำ (double click / enter รัว / refresh หลังส่ง)
- ลดจำนวน request ซ้ำที่เกิดจาก UX หรือ network delay
- ให้ระบบปลอดภัยและมีผลลัพธ์เพียงครั้งเดียว (idempotent)

---

## 1. หลักการสำคัญ
| ชั้น | วิธีป้องกัน |
|------|---------------|
| **UI** | ปุ่มต้องปิด (disabled) ระหว่างส่ง |
| **State** | ใช้สถานะ `submitting` |
| **Network** | ใช้ `Idempotency-Key` |
| **Client Interceptor** | กันยิง request เดิมซ้ำ |
| **Server** | รองรับ idempotency-key และคืนผลเดิมถ้ามี |

---

## 2. รูปแบบสถานะฟอร์ม
```txt
idle → submitting → success | error
```

### ตัวอย่างโค้ด React/Next.js
```tsx
import { useState } from "react";

function SubmitButton() {
  const [submitting, setSubmitting] = useState(false);

  async function handleSubmit() {
    if (submitting) return; // ป้องกันกดซ้ำ
    setSubmitting(true);

    try {
      await fetch("/api/form", { method: "POST", body: JSON.stringify({ ... }) });
      alert("สำเร็จ!");
    } finally {
      setSubmitting(false);
    }
  }

  return (
    <button onClick={handleSubmit} disabled={submitting}>
      {submitting ? "กำลังบันทึก..." : "บันทึก"}
    </button>
  );
}
```

---

## 3. ป้องกันระดับ Network: Idempotency-Key
ใช้ UUID เพื่อบอก backend ว่า request นี้ “เคยส่งแล้ว”  
ถ้า key ซ้ำ → backend ต้องคืนผลเดิม ไม่สร้างข้อมูลใหม่

```ts
import { v4 as uuid } from "uuid";
import axios from "axios";

export async function postOnce(url: string, data: any) {
  const key = uuid();
  return axios.post(url, data, {
    headers: { "Idempotency-Key": key },
  });
}
```

> ทุก request ที่เปลี่ยนข้อมูล (POST / PUT / PATCH / DELETE) ต้องมี header นี้

---

## 4. Interceptor กันยิง request ซ้ำ
กันกรณีที่ผู้ใช้กด submit ซ้ำก่อน response กลับ

```ts
const inFlight = new Map<string, AbortController>();

axios.interceptors.request.use((config) => {
  const key = `${config.method}|${config.url}|${JSON.stringify(config.data)}`;
  if (inFlight.has(key)) throw new axios.Cancel(`duplicate: ${key}`);
  const ctl = new AbortController();
  config.signal = ctl.signal;
  inFlight.set(key, ctl);
  return config;
});

axios.interceptors.response.use(
  (res) => {
    const key = `${res.config.method}|${res.config.url}|${JSON.stringify(res.config.data)}`;
    inFlight.delete(key);
    return res;
  },
  (err) => {
    const cfg = err.config || {};
    const key = `${cfg.method}|${cfg.url}|${JSON.stringify(cfg.data)}`;
    inFlight.delete(key);
    return Promise.reject(err);
  }
);
```

---

## 5. UX เสริม
- เปลี่ยนข้อความปุ่ม เช่น `Save → Saving...`
- เพิ่ม spinner หรือ progress bar
- ปิดการกด Enter ซ้ำใน `<form>`:
  ```tsx
  <form onKeyDown={(e) => e.key === "Enter" && e.preventDefault()}>
  ```
- Debounce action ที่ไม่ต้องยิงบ่อย เช่น search
- ในมือถือ: ป้องกัน double-tap ด้วย delay 300ms

---

## 6. ตรวจซ้ำด้วย Idempotency-Key ฝั่ง Server
ตัวอย่าง Response เมื่อซ้ำ:
```json
{
  "system": {
    "code": 3002,
    "status": "DUPLICATE_REQUEST",
    "message": "Request นี้ถูกส่งมาแล้ว"
  },
  "data": { "id": "abc123", "status": "success" }
}
```

---

## 7. Checklist ทีม Frontend
- [ ] ปุ่มทุกตัวที่ยิง API ปิด (disabled) ระหว่างส่ง
- [ ] ทุก request POST/PUT/PATCH/DELETE มี `Idempotency-Key`
- [ ] HTTP Client มี Interceptor กันยิงซ้ำ
- [ ] ป้องกัน Enter ซ้ำใน form
- [ ] มี feedback ผู้ใช้ขณะโหลด
- [ ] ผ่านการทดสอบ double-click, tap ซ้ำ, refresh หลัง submit

---

## 8. สรุป
> UI ล็อก, Network มี idempotency, Client กันซ้ำ, Server คืนผลเดิม  
> 4 ชั้นนี้ร่วมกัน = ไม่มีข้อมูลซ้ำ แม้กดรัวหรือ refresh
