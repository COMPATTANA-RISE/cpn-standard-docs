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

# มาตรฐานการใช้งาน Query String สำหรับ Search

> ใช้ได้กับ REST ทุก service ให้พฤติกรรมสอดคล้องกัน

## 1) พารามิเตอร์มาตรฐาน
| ชื่อ | ตัวอย่าง | ความหมาย |
|---|---|---|
| `q` | `q=brake pad` | คำค้นหากลาง (full-text) |
| `page` | `page=1` | หน้าเริ่มที่ 1 |
| `limit` | `limit=20` | จำนวนต่อหน้า (สูงสุด 100) |
| `sort` | `sort=createdAt` | จัดเรียงตามฟิลด์ |
| `order` | `order=asc` | `asc` หรือ `desc` |
| `fields` | `fields=id,name,price` | เลือกฟิลด์ที่ต้องการ |
| `include` | `include=brand,stocks` | รวมความสัมพันธ์ (eager) |
| `lang` | `lang=th` | ภาษาสำหรับ highlight/normalize |
| `cursor` | `cursor=eyJpZCI6…` | ใช้แทน page/limit ใน cursor pagination |

ข้อกำหนด:
- `limit` ค่าเริ่มต้น 20 สูงสุด 100
- ถ้าใช้ `cursor` ห้ามใช้ `page`
- ค่าทุกตัวต้อง URL-encoded

---

## 2) ฟิลเตอร์พื้นฐาน (Equality / In / Null)
รูปแบบ: `field[operator]=value`

| ตัวดำเนินการ | ความหมาย | ตัวอย่าง |
|---|---|---|
| (ว่าง) | เท่ากับ | `status=ACTIVE` |
| `ne` | ไม่เท่ากับ | `status[ne]=ACTIVE` |
| `in` | อยู่ในชุดค่า | `status[in]=ACTIVE,PAUSED` |
| `nin` | ไม่อยู่ในชุดค่า | `status[nin]=DRAFT,ARCHIVED` |
| `isnull` | เป็นค่าว่าง/ไม่มี | `deletedAt[isnull]=true` |

หลายค่าให้ใช้ comma (`,`). ช่องว่างไม่อนุญาต

---

## 3) ช่วงตัวเลข/วันที่ (Range)
| ตัวดำเนินการ | ตัวอย่าง |
|---|---|
| `gt` มากกว่า | `price[gt]=100` |
| `gte` มากกว่าหรือเท่ากับ | `price[gte]=100` |
| `lt` น้อยกว่า | `price[lt]=500` |
| `lte` น้อยกว่าหรือเท่ากับ | `price[lte]=500` |
| ช่วงรวดเดียว | `created_at[between]=2025-10-01,2025-10-14` (ISO8601, UTC) |

วันที่ใช้ ISO8601: `YYYY-MM-DD` หรือ `YYYY-MM-DDTHH:mm:ssZ`

---

## 4) ข้อความ (Text Search)
โหมดกำหนดโดย `searchType` หรืออัตโนมัติเมื่อมี `q`
| โหมด | พารามิเตอร์ | ตัวอย่าง | พฤติกรรม |
|---|---|---|---|
| Exact | `name` | `name=Nissan%20Almera` (Nissan Almera) | ตรงเป๊ะ (case-insensitive) |
| Prefix | `name[prefix]` | `name[prefix]=nis` | เริ่มต้นด้วย |
| Contains | `name[contains]` | `name[contains]=almera` | มีคำย่อย |
| Fuzzy | `name[fuzzy]` | `name[fuzzy]=almra` | ผิดสะกดเล็กน้อย |
| Full-text | `q` | `q=brake pad` | ค้นหาทุกฟิลด์ที่จัดทำดัชนี |
| Regex* | `name[regex]` | `name[regex]=^A[0-9]{3}$` | จำกัดเฉพาะ internal/admin |

\* ปิดใน public API โดยค่าเริ่มต้น

Highlight:
- `highlight=true` → ส่ง `data[i].highlight` กลับ

---

## 5) หลายเงื่อนไข AND/OR
- ค่าหลายตัวของ “ฟิลด์เดียวกัน” → AND ภายในฟิลด์  
  `status=in=ACTIVE,PAUSED` คือ (ACTIVE OR PAUSED)  
- หลายฟิลด์พร้อมกัน → AND ระหว่างฟิลด์  
- OR ข้ามฟิลด์: ใช้บล็อค `or[...]`  



