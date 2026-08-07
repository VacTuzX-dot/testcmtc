# 5. Setup จากศูนย์ — ทำตามนี้ทีละขั้น

### บน GitHub (ครั้งเดียว)
1. ไปที่ repo → **Settings → Secrets and variables → Actions**
2. เพิ่ม secret 3 ตัว: `SERVER_IP`, `SERVER_USER`, `SERVER_PASSWORD`

⚠️ lab นี้ใช้ **password auth** ผ่าน secret — ใช้งานได้แต่อ่อนกว่า SSH key จริง production ควรเปลี่ยนเป็น key-based auth + ปิด `PasswordAuthentication` ใน sshd_config

### บน Server (ครั้งเดียว)

**0. Prerequisite — server ต้องมีของพวกนี้ติดตั้งไว้ก่อน:**
- Docker Engine (`docker --version`)
- Docker Compose plugin (`docker compose version`)
- Git (`git --version`)
- เปิด SSH ให้ GitHub Actions เข้าได้ (password auth หรือ key — lab นี้ใช้ password)

ถ้ายังไม่มี Docker ติดตั้งด้วย script ทางการ:
```bash
curl -fsSL https://get.docker.com | sudo sh
```

**a. user ต้องอยู่ใน docker group** (ไม่งั้นรัน `docker` ผ่าน SSH script จะเจอ `permission denied`)
```bash
sudo usermod -aG docker $USER
# ต้อง logout แล้ว SSH เข้าใหม่จริงๆ — groups ไม่ refresh ใน session เดิม
exit
# ssh กลับเข้ามาใหม่ แล้วเช็ค
groups        # ต้องเห็น docker ในลิสต์ของ session ปัจจุบัน
docker ps     # ต้องไม่ error
```
⚠️ **`docker` group = root-equivalent** — user ใน group นี้ mount host `/` ผ่าน container ได้ (`docker run -v /:/host ...`) เท่ากับมี root บนเครื่องจริง โอเคสำหรับ lab ส่วนตัว **ห้ามทำแบบนี้บน shared/prod server** โดยไม่รู้ตัว

**b. สร้าง external network ที่ docker-compose.yml ต้องการ**
```bash
docker network create web
```
ถ้าไม่สร้างก่อน `docker compose up` จะ error:
```
network web declared as external, but could not be found
```

### Deploy
```bash
git push origin main
```
ไปดู progress ที่ tab **Actions** ของ repo บน GitHub

### เช็คว่า deploy สำเร็จ
```bash
# บน server
docker ps                                    # เห็น container student01_web สถานะ Up
docker exec student01_web curl -s localhost  # ได้ HTML กลับมา = nginx ทำงานถูก
```
จาก browser เครื่องไหนก็ได้:
```
http://<PUBLIC_IP>:9834
```

---
[← CI/CD Pipeline](04-cicd-pipeline.md) | [กลับ README](../README.md) | [ถัดไป: Troubleshooting →](06-troubleshooting.md)
