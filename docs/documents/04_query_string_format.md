# การใช้งาน Query String Parameter

> คู่มือสำหรับสร้าง URL parameters ตามมาตรฐานการใช้งาน Query String สำหรับ Search

## 🎯 การใช้งานพื้นฐาน

## 📊 พารามิเตอร์มาตรฐาน

| ชื่อ | ตัวอย่าง | ความหมาย | ค่าเริ่มต้น |
|---|---|---|---|
| `q` | `q=brake pad` | คำค้นหากลาง (full-text) | - |
| `page` | `page=1` | หน้าเริ่มที่ 1 | 1 |
| `limit` | `limit=20` | จำนวนต่อหน้า (สูงสุด 100) | 20 |
| `sort` | `sort=created_at` | จัดเรียงตามฟิลด์ | `created_at` |
| `order` | `order=asc` | `asc` หรือ `desc` | `desc` |
| `fields` | `fields=id,name,price` | เลือกฟิลด์ที่ต้องการ | - |
| `include` | `include=brand,stocks` | รวมความสัมพันธ์ (eager) | - |
| `lang` | `lang=th` | ภาษาสำหรับ highlight/normalize | - |
| `cursor` | `cursor=eyJpZCI6…` | ใช้แทน page/limit ใน cursor pagination | - |

### ข้อกำหนด:
- `limit` ค่าเริ่มต้น 20 สูงสุด 100
- ถ้าใช้ `cursor` ห้ามใช้ `page`
- ค่าทุกตัวต้อง URL-encoded

## 🔍 ฟิลเตอร์พื้นฐาน (Equality / In / Null)

รูปแบบ: `field[operator]=value`

| ตัวดำเนินการ | ความหมาย | ตัวอย่าง |
|---|---|---|
| (ว่าง) | เท่ากับ | `status=ACTIVE` |
| `ne` | ไม่เท่ากับ | `status[ne]=ACTIVE` |
| `in` | อยู่ในชุดค่า | `status[in]=ACTIVE,PAUSED` |
| `nin` | ไม่อยู่ในชุดค่า | `status[nin]=DRAFT,ARCHIVED` |
| `isnull` | เป็นค่าว่าง/ไม่มี | `deletedAt[isnull]=true` |

**ตัวอย่าง:**
```typescript
const query = {
  status: 'ACTIVE',
  'status[ne]': 'DRAFT',
  'category[in]': 'automotive,electronics',
  'deletedAt[isnull]': 'true'
};
```

## 📈 ช่วงตัวเลข/วันที่ (Range)

| ตัวดำเนินการ | ตัวอย่าง |
|---|---|
| `gt` มากกว่า | `price[gt]=100` |
| `gte` มากกว่าหรือเท่ากับ | `price[gte]=100` |
| `lt` น้อยกว่า | `price[lt]=500` |
| `lte` น้อยกว่าหรือเท่ากับ | `price[lte]=500` |
| `between` ช่วงรวดเดียว | `created_at[between]=2025-01-01,2025-12-31` |

**ตัวอย่าง:**
```typescript
const query = {
  'price[gte]': '100',
  'price[lte]': '500',
  'created_at[between]': '2025-01-01,2025-12-31'
};
```

## 🔤 การค้นหาข้อความ (Text Search)

โหมดกำหนดโดย `searchType` หรืออัตโนมัติเมื่อมี `q`

| โหมด | พารามิเตอร์ | ตัวอย่าง | พฤติกรรม |
|---|---|---|---|
| Exact | `name` | `name=Nissan%20Almera` | ตรงเป๊ะ (case-insensitive) |
| Prefix | `name[prefix]` | `name[prefix]=nis` | เริ่มต้นด้วย |
| Contains | `name[contains]` | `name[contains]=almera` | มีคำย่อย |
| Fuzzy | `name[fuzzy]` | `name[fuzzy]=almra` | ผิดสะกดเล็กน้อย |
| Full-text | `q` | `q=brake pad` | ค้นหาทุกฟิลด์ที่จัดทำดัชนี |
| Regex* | `name[regex]` | `name[regex]=^A[0-9]{3}$` | จำกัดเฉพาะ internal/admin |

\* ปิดใน public API โดยค่าเริ่มต้น

**ตัวอย่าง:**
```typescript
const query = {
  q: 'brake pad',                    // Full-text search
  'name[contains]': 'brake',        // Contains search
  'category[prefix]': 'auto',        // Prefix search
  searchType: 'fuzzy'                // Fuzzy search mode
};
```

## 🔗 หลายเงื่อนไข AND/OR

### AND ภายในฟิลด์เดียวกัน
```typescript
// ค่าหลายตัวของฟิลด์เดียวกัน → AND ภายในฟิลด์
const query = {
  'status[in]': 'ACTIVE,PAUSED'  // (ACTIVE OR PAUSED)
};
```

### AND ระหว่างฟิลด์
```typescript
// หลายฟิลด์พร้อมกัน → AND ระหว่างฟิลด์
const query = {
  status: 'ACTIVE',
  category: 'automotive',
  'price[gte]': '100'
};
```

### OR ข้ามฟิลด์
```typescript
// OR ข้ามฟิลด์: ใช้บล็อค or[...]
const query = {
  'or[0][category]': 'automotive',
  'or[0][brand]': 'toyota',
  'or[1][price][lt]': '1000',
  'or[1][status]': 'SALE'
};
```

**ผลลัพธ์:**
```sql
WHERE (
  (category = 'automotive' OR brand = 'toyota') 
  OR 
  (price < 1000 OR status = 'SALE')
)

```

## 📝 ตัวอย่างการใช้งาน

### 1. การค้นหาพื้นฐาน
```typescript
// URL: /api/products?q=brake&page=1&limit=20
const query = {
  q: 'brake',
  page: 1,
  limit: 20
};

```

### 2. การกรองข้อมูล
```typescript
// URL: /api/products?status=ACTIVE&price[gte]=100&price[lte]=500
const query = {
  status: 'ACTIVE',
  'price[gte]': '100',
  'price[lte]': '500'
};

```

### 3. การค้นหาข้อความแบบเฉพาะ
```typescript
// URL: /api/products?name[contains]=brake&category[prefix]=auto
const query = {
  'name[contains]': 'brake',
  'category[prefix]': 'auto'
};

```

### 4. การใช้ OR Blocks
```typescript
// URL: /api/products?or[0][category]=auto&or[0][brand]=toyota&or[1][price][lt]=1000
const query = {
  'or[0][category]': 'auto',
  'or[0][brand]': 'toyota',
  'or[1][price][lt]': '1000'
};

```

### 5. การใช้ Cursor Pagination
```typescript
// URL: /api/products?cursor=eyJpZCI6MTIzfQ&limit=20
const query = {
  cursor: 'eyJpZCI6MTIzfQ',
  limit: 20
};

```

## ⚠️ ข้อควรระวัง

1. **Regex Search**: เปิดใช้งานเฉพาะ internal/admin APIs
2. **Cursor vs Page**: ห้ามใช้ `cursor` และ `page` พร้อมกัน
3. **Limit**: สูงสุด 100 รายการต่อหน้า
4. **URL Encoding**: ค่าทุกตัวต้อง URL-encoded

---

> 📖 **หมายเหตุ**: เครื่องมือนี้ถูกออกแบบให้รองรับมาตรฐานการใช้งาน Query String สำหรับ Search ทุก service ให้พฤติกรรมสอดคล้องกัน
