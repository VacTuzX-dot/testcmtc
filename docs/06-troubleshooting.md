# 6. ปัญหาที่เจอจริงระหว่าง lab นี้ (และวิธีแก้)

| อาการ | สาเหตุ | วิธีแก้ |
|---|---|---|
| `permission denied ... docker.sock` | user SSH ไม่อยู่ใน `docker` group (หรืออยู่แล้วแต่ session เก่ายังไม่ refresh) | `sudo usermod -aG docker $USER` แล้ว **SSH เข้าใหม่** (ไม่ใช่แค่ `groups $USER` เพราะอันนั้นอ่านจาก `/etc/group` ตรงๆ ไม่ใช่ active session) |
| `network web declared as external, but could not be found` | docker-compose.yml อ้างถึง network `web` ที่ยังไม่ถูกสร้าง | `docker network create web` |
| เข้า `http://<PUBLIC_IP>` ไม่ได้ | docker-compose.yml ตอนแรกไม่มี `ports:` — container เข้าถึงได้แค่ผ่าน Traefik (ซึ่ง server lab นี้ไม่มีรัน) | เพิ่ม `ports: ["9834:80"]` ใน docker-compose.yml แล้ว push ใหม่ |

**ถ้าเจอปัญหาอื่นที่ไม่อยู่ในตาราง — ลำดับการ debug ที่ควรทำ:**
1. `docker ps` — container รันอยู่จริงไหม สถานะ `Up` หรือ `Restarting`/`Exited`
2. `docker logs student01_web` — nginx error อะไรไหม
3. ไปดู tab **Actions** บน GitHub — job ไหน fail, log ว่าอะไร
4. เช็คว่า secrets 3 ตัว (`SERVER_IP`/`SERVER_USER`/`SERVER_PASSWORD`) ยังถูกต้องอยู่ไหม (server เปลี่ยน IP/password บ่อยเป็นสาเหตุยอดฮิต)

---
[← Setup Guide](05-setup-guide.md) | [กลับ README](../README.md) | [ถัดไป: HTTPS →](07-https.md)
