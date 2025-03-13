# PostgreSQL Comment

**การใช้ COMMENT ใน PostgreSQL**

คำสั่ง `COMMENT` ใช้สำหรับเพิ่มคำอธิบาย (metadata) ให้กับ object ต่าง ๆ ในฐานข้อมูล PostgreSQL เช่น ตาราง คอลัมน์ อินเด็กซ์ ฯลฯ เพื่อช่วยให้เข้าใจการใช้งานของแต่ละส่วนได้ง่ายขึ้น

## 1. การเพิ่มคอมเมนต์
โครงสร้างคำสั่ง

```sql
COMMENT ON object_type object_name IS 'your_comment';
```

โดยที่
- `object_type` คือ ประเภทของออบเจ็กต์ เช่น `TABLE`, `COLUMN`, `INDEX`, `SCHEMA`, `FUNCTION`
- `object_name` คือ ชื่อของออบเจ็กต์ที่ต้องการเพิ่มคำอธิบาย
- `'your_comment'` คือ ข้อความที่ต้องการเพิ่มเป็นคอมเมนต์

ตัวอย่าง
- เพิ่มคอมเมนต์ให้กับตาราง:
```sql
COMMENT ON TABLE employees IS 'เก็บข้อมูลพนักงานทั้งหมด';
```

- เพิ่มคอมเมนต์ให้กับคอลัมน์:
```sql
COMMENT ON COLUMN employees.salary IS 'เงินเดือนของพนักงาน';
```

- เพิ่มคอมเมนต์ให้กับอินเด็กซ์:
```sql
COMMENT ON INDEX idx_employees_name IS 'ใช้สำหรับค้นหาพนักงานตามชื่อ';
```

- เพิ่มคอมเมนต์ให้กับฟังก์ชัน:
```sql
COMMENT ON FUNCTION calculate_bonus(int) IS 'คำนวณโบนัสจากคะแนนพนักงาน';
```

---

## 2. การเรียกดูคอมเมนต์

ใช้ `pg_description` สำหรับดูคอมเมนต์ของคอลัมน์

```sql
SELECT 
    obj_description('schema_name.table_name'::regclass, 'pg_class') AS table_comment;
```
หรือสำหรับคอลัมน์:
```sql
SELECT 
    col_description('schema_name.table_name'::regclass, column_position) AS column_comment;
```
โดยที่ `column_position` คือ ลำดับของคอลัมน์ในตาราง (`เริ่มจาก 1`)

ใช้ `pg_catalog` สำหรับดูคอมเมนต์ของออบเจ็กต์ต่าง ๆ
```sql
SELECT 
    c.oid, c.relname, d.description
FROM 
    pg_class c
JOIN 
    pg_description d ON c.oid = d.objoid
WHERE 
    c.relname = 'employees';
```

**ดึงคอมเมนต์ของหลายตาราง**
```sql
SELECT 
    c.relname AS table_name, 
    obj_description(c.oid, 'pg_class') AS table_comment
FROM 
    pg_class c
WHERE 
    c.relkind = 'r'  -- 'r' หมายถึง table
    AND c.relname IN ('employees', 'departments', 'salaries');  -- รายการของตารางที่ต้องการดู
```
อธิบายโค้ด
- `pg_class` ใช้เก็บ metadata ของตาราง
- `obj_description(c.oid, 'pg_class')` ใช้ดึงคอมเมนต์ของตาราง
กรองเฉพาะ `relkind = 'r'` เพื่อให้เลือกเฉพาะตารางปกติ
กรองชื่อที่ต้องการ (`IN ('employees', 'departments', 'salaries')`)

**ดึงคอมเมนต์ของทุกคอลัมน์ในหลายตาราง**
```sql
SELECT 
    c.relname AS table_name, 
    a.attname AS column_name, 
    col_description(a.attrelid, a.attnum) AS column_comment
FROM 
    pg_class c
JOIN 
    pg_attribute a ON c.oid = a.attrelid
WHERE 
    c.relkind = 'r'  -- เฉพาะ table
    AND a.attnum > 0  -- ไม่รวม internal columns
    AND c.relname IN ('employees', 'departments', 'salaries')  -- เลือกเฉพาะบางตาราง
ORDER BY 
    c.relname, a.attnum;
```
อธิบายโค้ด
- `pg_attribute` ใช้เก็บ metadata ของคอลัมน์ในตาราง
- `col_description(a.attrelid, a.attnum)` ใช้ดึงคอมเมนต์ของคอลัมน์
- `a.attnum > 0` กรองไม่ให้รวม internal system columns (`attnum` < 0)
- `ORDER BY c.relname, a.attnum` จัดเรียงตามตารางและลำดับคอลัมน์

**ดึงคอมเมนต์ของทุกตาราง และทุกคอลัมน์ใน Database**
หากต้องการดึงคอมเมนต์ของ ทุกตาราง และทุกคอลัมน์ ในฐานข้อมูลทั้งหมด:
```sql
SELECT 
    c.relname AS table_name, 
    a.attname AS column_name, 
    col_description(a.attrelid, a.attnum) AS column_comment
FROM 
    pg_class c
JOIN 
    pg_attribute a ON c.oid = a.attrelid
WHERE 
    c.relkind = 'r'  -- เฉพาะ table
    AND a.attnum > 0  -- ไม่รวม internal columns
ORDER BY 
    c.relname, a.attnum;
```
📌 โค้ดนี้จะดึงข้อมูลของทุกตารางและคอลัมน์ที่มีคอมเมนต์อยู่

**ใช้ `psql` สำหรับดูคอมเมนต์**
ถ้าใช้ `psql` สามารถใช้คำสั่ง:
```sh
\d+ table_name
```
เพื่อแสดงข้อมูลของตารางรวมถึงคอมเมนต์ของคอลัมน์ด้วย

---

## 3. การลบคอมเมนต์
หากต้องการลบคอมเมนต์ ให้ใช้ `NULL` แทนข้อความคอมเมนต์
```sql
COMMENT ON COLUMN employees.salary IS NULL;
```

---

สรุป
- ใช้ `COMMENT ON` เพื่อเพิ่มคำอธิบายให้กับออบเจ็กต์ เช่น ตาราง คอลัมน์ ฟังก์ชัน ฯลฯ
- ใช้ `pg_description` และ `pg_catalog` เพื่อตรวจสอบคอมเมนต์ที่ถูกกำหนดไว้
- ใช้ `\d+ table_name` ใน `psql` เพื่อตรวจสอบคอมเมนต์ของคอลัมน์
- สามารถลบคอมเมนต์ได้โดยกำหนดค่า `NULL`

