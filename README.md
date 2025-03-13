# PostgreSQL Comment

การใช้ COMMENT ใน PostgreSQL

คำสั่ง `COMMENT` ใช้สำหรับเพิ่มคำอธิบาย (metadata) ให้กับ object ต่าง ๆ ในฐานข้อมูล PostgreSQL เช่น ตาราง คอลัมน์ อินเด็กซ์ ฯลฯ เพื่อช่วยให้เข้าใจการใช้งานของแต่ละส่วนได้ง่ายขึ้น

## 1. การเพิ่มคอมเมนต์
โครงสร้างคำสั่ง

```sql
COMMENT ON object_type object_name IS 'your_comment';
```
