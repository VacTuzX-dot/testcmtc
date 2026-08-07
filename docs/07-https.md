# 7. HTTPS — ทำต่อยังไง (ยังไม่ได้ทำใน lab นี้)

ตอนนี้ site รันเป็น **HTTP เปิด port ตรง** (`http://IP:9834`) — Let's Encrypt **ออก cert ให้ IP ตรงๆ ไม่ได้** ต้องมี domain เท่านั้น

ทางเลือก:

1. **Cloudflare proxy (ง่ายสุด)** — ถ้า domain อยู่หลัง Cloudflare (orange cloud เปิด) Cloudflare จะ terminate TLS ให้ฟรี โดยที่ server ข้างหลังยังเป็น HTTP ปกติ ไม่ต้องแก้ docker-compose เลย
2. **Traefik + Let's Encrypt (ของจริง)** — labels ใน [`docker-compose.yml`](../docker-compose.yml) เตรียม `traefik.*` ไว้แล้ว แต่ server lab นี้ยังไม่มี Traefik รันอยู่ ต้อง deploy Traefik container แยกต่างหากก่อน ถึงจะ route domain `student01-lab.cmtc.ac.th` ให้อัตโนมัติพร้อมออก cert ให้เองได้

---
[← Troubleshooting](06-troubleshooting.md) | [กลับ README](../README.md) | [ถัดไป: Cheat Sheet →](08-cheatsheet.md)
