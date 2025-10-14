# มาตรฐานโค้ด (Code Style)

## Backend (Node/NestJS)
- ใช้ `eslint` + `prettier` + `husky` ตรวจโค้ดก่อน commit
- ใช้ async/await (ไม่ใช้ callback)
- ตรวจ DTO ด้วย Zod/Joi
- รวม error handling กลางไว้ middleware

## Frontend (React/Vue)
- ใช้ ESLint + Prettier ที่แชร์ผ่าน monorepo
- ชื่อ Component → PascalCase
- Props → camelCase
- หลีกเลี่ยง inline style ใช้ CSS modules/Tailwind แทน
