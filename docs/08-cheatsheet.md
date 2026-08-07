# 8. คำสั่งที่ใช้บ่อย (cheat sheet)

```bash
# ดู container ที่รันอยู่
docker ps

# ดู log ของ container
docker logs student01_web

# เข้าไปรันคำสั่งข้างใน container
docker exec -it student01_web sh

# restart container โดยไม่ build ใหม่
docker compose restart

# build ใหม่ + restart (เหมือนที่ pipeline ทำ)
docker compose up -d --build

# ลบ container (ข้อมูล container หาย แต่ image ยังอยู่)
docker compose down

# ดู network ทั้งหมด
docker network ls
```

---
[← HTTPS](07-https.md) | [กลับ README](../README.md) | [ถัดไป: สรุป Concept →](09-concepts.md)
