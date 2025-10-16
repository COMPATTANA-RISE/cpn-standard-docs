# มาตรฐานโค้ด (Code Style)

## Backend (Node/NestJS)
- ใช้ `eslint` + `prettier` + `husky` ตรวจโค้ดก่อน commit
- ใช้ async/await (ไม่ใช้ callback)
- ตรวจ DTO ด้วย Zod หรือ class-validator (NestJs)

## Frontend (React,NextJs)
- ใช้ ESLint + Prettier
- ชื่อ Component → PascalCase
- Props → camelCase
- หลีกเลี่ยง inline style ใช้ CSS modules/Tailwind แทน
