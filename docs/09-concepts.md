# 9. สรุป concept สำคัญสำหรับสอนเพื่อน

1. **Dockerfile** = สูตรสร้าง image (เปรียบเหมือน recipe)
2. **docker-compose.yml** = วิธีรัน container จาก image นั้น (network, port, restart policy, labels)
3. **External network** ใน compose ต้องมีอยู่แล้วบน host ก่อน — compose ไม่สร้างให้เองถ้าเป็น `external: true`
4. **Group membership ใน Linux ไม่ refresh แบบ real-time** — ต้อง re-login session ถึงจะมีผล (เป็นกับดักที่เจอบ่อยมาก)
5. **`ports:` vs reverse proxy label** — สองทางเข้าเว็บคนละแบบ เลือกใช้ตามว่ามี reverse proxy (Traefik/Nginx) อยู่หน้า server หรือไม่
6. **CI/CD pipeline = automate สิ่งที่เคยทำมือ** — validate → SSH → pull → rebuild → restart ทุก push โดยไม่ต้องเข้า server เอง
7. **แยก job เป็น build/test/deploy + `needs:`** = fail fast — job ที่พังจะบล็อก job ถัดไปไม่ให้รัน ไม่ต้องเสียเวลาไป SSH เข้า server แล้วเจอ error ที่จริงๆ ควรจับได้ตั้งแต่ก่อน build เสร็จ

---
[← Cheat Sheet](08-cheatsheet.md) | [กลับ README](../README.md)
