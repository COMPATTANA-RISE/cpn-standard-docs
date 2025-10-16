# คู่มือการใช้งาน Query Builder สำหรับ Frontend

## ภาพรวม
Query Builder เป็น utility ที่ช่วยในการสร้าง query parameters สำหรับ API endpoints ที่รองรับการกรอง, ค้นหา, เรียงลำดับ และ pagination

## พารามิเตอร์พื้นฐาน

### Pagination
| พารามิเตอร์ | ประเภท | ค่าเริ่มต้น | คำอธิบาย |
|------------|--------|------------|----------|
| `page` | number | 1 | หน้าที่ต้องการ (ต้อง >= 1) |
| `limit` | number | 20 | จำนวนรายการต่อหน้า (1-100) |
| `cursor` | string | - | สำหรับ cursor-based pagination |

### การเรียงลำดับ
| พารามิเตอร์ | ประเภท | ค่าเริ่มต้น | คำอธิบาย |
|------------|--------|------------|----------|
| `sort` | string | created_at | ฟิลด์ที่ใช้เรียงลำดับ |
| `order` | string | desc | ทิศทางการเรียง (asc/desc) |

### การค้นหา
| พารามิเตอร์ | ประเภท | คำอธิบาย |
|------------|--------|----------|
| `q` | string | คำค้นหาหลัก |
| `search` | string | คำค้นหา (legacy) |
| `searchBy` | string | ฟิลด์ที่ต้องการค้นหา (คั่นด้วย comma) |
| `searchType` | string | ประเภทการค้นหา (exact/prefix/contains/fuzzy) |

### การเลือกข้อมูล
| พารามิเตอร์ | ประเภท | คำอธิบาย |
|------------|--------|----------|
| `fields` | string | ฟิลด์ที่ต้องการเลือก (คั่นด้วย comma) |
| `include` | string | Relations ที่ต้องการ include (คั่นด้วย comma) |

## ตัวกรอง (Filters)

### รูปแบบการใช้งาน
```
field[operator]=value
```

### Operators ที่รองรับ

#### ตัวกรองพื้นฐาน
| Operator | คำอธิบาย | ตัวอย่าง |
|----------|----------|----------|
| (ไม่มี) | เท่ากับ | `name=John` |
| `ne` | ไม่เท่ากับ | `name[ne]=John` |
| `isnull` | เป็น null หรือไม่ | `email[isnull]=true` |

#### ตัวกรองสำหรับ Array
| Operator | คำอธิบาย | ตัวอย่าง |
|----------|----------|----------|
| `in` | อยู่ในรายการ | `status[in]=active,pending` |
| `nin` | ไม่อยู่ในรายการ | `status[nin]=deleted,archived` |

#### ตัวกรองสำหรับตัวเลข
| Operator | คำอธิบาย | ตัวอย่าง |
|----------|----------|----------|
| `gt` | มากกว่า | `price[gt]=1000` |
| `gte` | มากกว่าหรือเท่ากับ | `price[gte]=1000` |
| `lt` | น้อยกว่า | `price[lt]=5000` |
| `lte` | น้อยกว่าหรือเท่ากับ | `price[lte]=5000` |
| `between` | ระหว่าง | `price[between]=1000,5000` |
| `bt` | ระหว่าง (ย่อ) | `price[bt]=1000,5000` |
| `notbetween` | ไม่ระหว่าง | `price[notbetween]=1000,5000` |
| `nbt` | ไม่ระหว่าง (ย่อ) | `price[nbt]=1000,5000` |

#### ตัวกรองสำหรับข้อความ
| Operator | คำอธิบาย | ตัวอย่าง |
|----------|----------|----------|
| `contains` | ประกอบด้วย | `name[contains]=John` |
| `like` | เหมือน contains | `name[like]=John` |
| `fuzzy` | ค้นหาแบบคลุมเครือ | `name[fuzzy]=John` |
| `prefix` | ขึ้นต้นด้วย | `name[prefix]=Jo` |

## OR Blocks

สำหรับการสร้างเงื่อนไข OR สามารถใช้รูปแบบ `or[index]`:

```
or[0][field1]=value1
or[0][field2]=value2
or[1][field3]=value3
```

## ตัวอย่างการใช้งาน

### 1. การค้นหาพื้นฐาน
```javascript
// ค้นหาผู้ใช้ชื่อ "John"
const params = {
  q: "John",
  page: 1,
  limit: 20
};
```

### 2. การกรองข้อมูล
```javascript
// กรองผู้ใช้ที่ active และมี role เป็น admin
const params = {
  active: true,
  role: "admin",
  page: 1,
  limit: 10
};
```

### 3. การกรองแบบซับซ้อน
```javascript
// กรองผู้ใช้ที่สร้างระหว่างวันที่ 1-31 มกราคม 2024
const params = {
  "created_at[between]": "2024-01-01,2024-01-31",
  "status[in]": "active,pending",
  "age[gte]": 18,
  page: 1,
  limit: 50
};
```

### 4. การค้นหาในหลายฟิลด์
```javascript
// ค้นหาในฟิลด์ name และ email
const params = {
  q: "john",
  searchBy: "name,email",
  searchType: "contains",
  page: 1,
  limit: 20
};
```

### 5. การใช้ OR Blocks
```javascript
// ค้นหาผู้ใช้ที่ชื่อ "John" หรือ email "john@example.com"
const params = {
  "or[0][name]": "John",
  "or[0][email]": "john@example.com",
  page: 1,
  limit: 20
};
```

### 6. การเลือกฟิลด์เฉพาะ
```javascript
// เลือกเฉพาะฟิลด์ id, name, email
const params = {
  fields: "id,name,email",
  page: 1,
  limit: 20
};
```

### 7. การ Include Relations
```javascript
// Include relations profile และ orders
const params = {
  include: "profile,orders",
  page: 1,
  limit: 20
};
```

### 8. การเรียงลำดับ
```javascript
// เรียงตามชื่อจาก A-Z
const params = {
  sort: "name",
  order: "asc",
  page: 1,
  limit: 20
};
```

### 9. การใช้ Cursor Pagination
```javascript
// ใช้ cursor สำหรับ pagination
const params = {
  cursor: "eyJpZCI6IjEyMyJ9", // base64 encoded cursor
  limit: 20
};
```

## ตัวอย่าง URL ที่สมบูรณ์

```
GET /api/users?q=john&status[in]=active,pending&created_at[between]=2024-01-01,2024-01-31&sort=name&order=asc&page=1&limit=20&fields=id,name,email&include=profile
```

## ข้อควรระวัง

1. **Field Types**: ระบบจะแปลงค่าตาม field type ที่กำหนด
   - `string`: ข้อความ
   - `number`: ตัวเลข
   - `date`: วันที่
   - `boolean`: true/false

2. **การกรองแบบ Range**: ไม่สามารถใช้ `gte` และ `lte` หรือ `gt` และ `lt` พร้อมกันได้ ต้องใช้ `between` แทน

3. **Pagination**: ไม่สามารถใช้ `cursor` และ `page` พร้อมกันได้

4. **Limit**: ต้องอยู่ระหว่าง 1-100

5. **Page**: ต้อง >= 1

## Error Handling

ระบบจะส่ง BadRequestException เมื่อ:
- ใช้ field ที่ไม่อนุญาต
- ใช้ operator ที่ไม่ถูกต้อง
- ค่า limit หรือ page ไม่ถูกต้อง
- ใช้ cursor และ page พร้อมกัน
- ใช้ gte/lte หรือ gt/lt พร้อมกัน

## Best Practices

1. ใช้ `q` สำหรับการค้นหาทั่วไป
2. ใช้ `searchBy` เพื่อระบุฟิลด์ที่ต้องการค้นหา
3. ใช้ `between` แทน `gte` + `lte`
4. ใช้ `fields` เพื่อลดขนาด response
5. ใช้ `include` อย่างระมัดระวังเพื่อหลีกเลี่ยง N+1 queries
6. ใช้ cursor pagination สำหรับข้อมูลจำนวนมาก
