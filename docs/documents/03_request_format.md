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

# Query Builder Utility

> เครื่องมือสำหรับสร้าง Prisma query จาก URL parameters ตามมาตรฐานการใช้งาน Query String สำหรับ Search

## 📋 สารบัญ

- [การติดตั้ง](#การติดตั้ง)
- [การใช้งานพื้นฐาน](#การใช้งานพื้นฐาน)
- [พารามิเตอร์มาตรฐาน](#พารามิเตอร์มาตรฐาน)
- [ฟิลเตอร์พื้นฐาน](#ฟิลเตอร์พื้นฐาน)
- [ช่วงตัวเลข/วันที่](#ช่วงตัวเลขวันที่)
- [การค้นหาข้อความ](#การค้นหาข้อความ)
- [หลายเงื่อนไข AND/OR](#หลายเงื่อนไข-andor)
- [Cursor Pagination](#cursor-pagination)
- [การใช้งานใน Service](#การใช้งานใน-service)
- [ตัวอย่างการใช้งาน](#ตัวอย่างการใช้งาน)

## 🚀 การติดตั้ง

```typescript
import { queryBuilder, pageBuilder, cursorBuilder } from '@/common/utils/query-builder.util';
import { PaginationDto } from '@/common/schemas/pagination.schema';
```

## 🎯 การใช้งานพื้นฐาน

```typescript
const query: PaginationDto = {
  q: 'brake pad',
  page: 1,
  limit: 20,
  sort: 'createdAt',
  order: 'desc'
};

const searchFields = ['name', 'description'];  // ฟิลด์สำหรับการค้นหาข้อความ (q parameter)
const allowedFields = ['name', 'description', 'status', 'created_at']; // ฟิลด์ที่อนุญาตให้ใช้ในการ filter

const result = queryBuilder(query, searchFields, {
  allowedFields: allowedFields,
  haveActive: true,
  defaultSort: 'created_at',
  defaultOrder: 'desc'
});

// ใช้กับ Prisma
const data = await prisma.product.findMany(result);
```

### 📝 คำอธิบาย Parameters:

- **Parameter 1**: `query` - ข้อมูลจาก URL parameters
- **Parameter 2**: `searchFields` - ฟิลด์ที่ใช้สำหรับการค้นหาข้อความ (q parameter)
- **Parameter 3**: `options` - ตัวเลือกเพิ่มเติม
  - `allowedFields` - ฟิลด์ที่อนุญาตให้ใช้ในการ filter (ความปลอดภัย)
  - `haveActive` - เพิ่มเงื่อนไข active: true
  - `defaultSort` - ฟิลด์เรียงลำดับเริ่มต้น
  - `defaultOrder` - ทิศทางการเรียงลำดับเริ่มต้น

## 📊 พารามิเตอร์มาตรฐาน

| ชื่อ | ตัวอย่าง | ความหมาย | ค่าเริ่มต้น |
|---|---|---|---|
| `q` | `q=brake pad` | คำค้นหากลาง (full-text) | - |
| `page` | `page=1` | หน้าเริ่มที่ 1 | 1 |
| `limit` | `limit=20` | จำนวนต่อหน้า (สูงสุด 100) | 20 |
| `sort` | `sort=createdAt` | จัดเรียงตามฟิลด์ | `created_at` |
| `order` | `order=asc` | `asc` หรือ `desc` | `desc` |
| `fields` | `fields=id,name,price` | เลือกฟิลด์ที่ต้องการ | - |
| `include` | `include=brand,stocks` | รวมความสัมพันธ์ (eager) | - |
| `lang` | `lang=th` | ภาษาสำหรับ highlight/normalize | - |
| `cursor` | `cursor=eyJpZCI6…` | ใช้แทน page/limit ใน cursor pagination | - |
| `hasCompany` | `hasCompany=true` | บอกว่าต้องการค้นหาตาม company | - |
| `company_id` | `company_id=123` | company_id ที่ต้องการค้นหา | - |

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

## 📄 Cursor Pagination

```typescript
const query = {
  cursor: 'eyJpZCI6MTIzLCJjcmVhdGVkQXQiOiIyMDI1LTAxLTAxVDAwOjAwOjAwWiJ9',
  limit: 20
};

const result = queryBuilder(query, searchFields, options);

// ใช้กับ Prisma
const data = await prisma.product.findMany(result);

// สร้าง cursor ถัดไป
const nextCursor = generateNextCursor(data[data.length - 1], 'createdAt');
```

## 🏢 Company Filter

```typescript
// วิธีที่ 1: ใช้จาก query
const query = {
  hasCompany: true,
  company_id: '123',
  q: 'brake'
};

// วิธีที่ 2: ส่งผ่าน options
const result = queryBuilder(query, searchFields, {
  hasCompany: true,
  company_id: currentUser.company_id
});
```

## 🛠️ การใช้งานใน Service

### ตัวอย่างพื้นฐาน
```typescript
@Injectable()
export class ProductService {
  constructor(private readonly prisma: PrismaService) {}

  async findAll(query: PaginationDto, currentUser: any) {
    const searchFields = ['name', 'description', 'category'];
    const allowedFields = ['name', 'description', 'status', 'created_at', 'company_id'];

    const condition = queryBuilder(query, searchFields, {
      allowedFields: allowedFields,
      haveActive: true,
      defaultSort: 'created_at',
      defaultOrder: 'desc',
      hasCompany: true,
      company_id: currentUser.company_id,
    });

    const page = pageBuilder(query);

    const products = await this.prisma.product.findMany(condition);
    const total = await this.prisma.product.count({ where: condition.where });

    return {
      data: products,
      meta: {
        total,
        page: page.page,
        limit: page.limit,
        totalPages: Math.ceil(total / page.limit),
      },
    };
  }
}
```

### ตัวอย่างแบบยืดหยุ่น
```typescript
async findAllFlexible(query: PaginationDto, currentUser: any) {
  const searchFields = ['name', 'description'];
  const allowedFields = ['name', 'description', 'status', 'created_at', 'company_id'];

  // ใช้ company_id จาก query ถ้ามี hasCompany=true
  const companyId = query.hasCompany && query.company_id 
    ? query.company_id 
    : currentUser.company_id;

  const condition = queryBuilder(query, searchFields, {
    allowedFields: allowedFields,
    haveActive: true,
    defaultSort: 'created_at',
    defaultOrder: 'desc',
    hasCompany: true,
    company_id: companyId,
  });

  return this.prisma.product.findMany(condition);
}
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

const result = queryBuilder(query, ['name', 'description']);
```

### 2. การกรองข้อมูล
```typescript
// URL: /api/products?status=ACTIVE&price[gte]=100&price[lte]=500
const query = {
  status: 'ACTIVE',
  'price[gte]': '100',
  'price[lte]': '500'
};

const result = queryBuilder(query, [], {
  allowedFields: ['status', 'price']
});
```

### 3. การค้นหาข้อความแบบเฉพาะ
```typescript
// URL: /api/products?name[contains]=brake&category[prefix]=auto
const query = {
  'name[contains]': 'brake',
  'category[prefix]': 'auto'
};

const result = queryBuilder(query, ['name', 'category']);
```

### 4. การใช้ OR Blocks
```typescript
// URL: /api/products?or[0][category]=auto&or[0][brand]=toyota&or[1][price][lt]=1000
const query = {
  'or[0][category]': 'auto',
  'or[0][brand]': 'toyota',
  'or[1][price][lt]': '1000'
};

const result = queryBuilder(query, [], {
  allowedFields: ['category', 'brand', 'price']
});
```

### 5. การใช้ Cursor Pagination
```typescript
// URL: /api/products?cursor=eyJpZCI6MTIzfQ&limit=20
const query = {
  cursor: 'eyJpZCI6MTIzfQ',
  limit: 20
};

const result = queryBuilder(query, ['name']);
```

### 6. การใช้ Company Filter
```typescript
// URL: /api/products?hasCompany=true&company_id=123&q=brake
const query = {
  hasCompany: true,
  company_id: '123',
  q: 'brake'
};

const result = queryBuilder(query, ['name'], {
  hasCompany: true,
  company_id: '123'
});
```

## ⚠️ ข้อควรระวัง

1. **ความปลอดภัย**: ใช้ `allowedFields` เพื่อจำกัดฟิลด์ที่อนุญาต
2. **Regex Search**: เปิดใช้งานเฉพาะ internal/admin APIs
3. **Cursor vs Page**: ห้ามใช้ `cursor` และ `page` พร้อมกัน
4. **Limit**: สูงสุด 100 รายการต่อหน้า
5. **URL Encoding**: ค่าทุกตัวต้อง URL-encoded

## 🔧 Options Parameters

```typescript
interface QueryBuilderOptions {
  haveActive?: boolean;        // เพิ่มเงื่อนไข active: true
  defaultSort?: string;        // ฟิลด์เรียงลำดับเริ่มต้น
  defaultOrder?: 'asc' | 'desc'; // ทิศทางการเรียงลำดับเริ่มต้น
  allowRegex?: boolean;        // อนุญาต regex search
  allowedFields?: string[];    // ฟิลด์ที่อนุญาตให้ใช้
  hasCompany?: boolean;        // ใช้ company filter
  company_id?: string;        // company_id ที่ใช้
}
```

## 📚 ฟังก์ชันเสริม

```typescript
// สร้าง pagination metadata
const page = pageBuilder(query);

// สร้าง cursor pagination
const cursor = cursorBuilder(query);

// สร้าง next cursor
const nextCursor = generateNextCursor(lastItem, 'createdAt');

// สร้าง previous cursor
const prevCursor = generatePrevCursor(firstItem, 'createdAt');
```

---

> 📖 **หมายเหตุ**: เครื่องมือนี้ถูกออกแบบให้รองรับมาตรฐานการใช้งาน Query String สำหรับ Search ทุก service ให้พฤติกรรมสอดคล้องกัน


