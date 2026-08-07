# 2. Dockerfile — ทำอะไร

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

Docker image สร้างเป็น "ชั้น" (layer) ทับกัน แต่ละคำสั่งใน Dockerfile คือ 1 ชั้น อ่านทีละบรรทัด:

- **`FROM nginx:alpine`** — เริ่มจาก image สำเร็จรูปที่มี nginx (web server) ติดตั้งไว้แล้ว บน base OS "alpine" (Linux เบามาก ~5MB) รวมแล้ว image ทั้งก้อน ~40MB — เทียบกับ `nginx` เฉยๆ (ไม่มี `-alpine`) ที่หนักหลายร้อย MB
- **`COPY . /usr/share/nginx/html`** — copy ไฟล์ทั้งหมดจาก build context (โฟลเดอร์ปัจจุบันตอนรัน `docker build`) เข้าไปที่ `/usr/share/nginx/html` ข้างใน image — path นี้คือ **document root default** ของ nginx คือโฟลเดอร์ที่ nginx serve ไฟล์ static ออกมาให้ browser

ไม่มี build step อื่นเพราะเป็น static site ล้วนๆ (ไม่มี npm build, ไม่มี backend compile, ไม่มี dependency ต้อง install)

**⚠️ Best practice ที่ยังไม่ได้ทำในโปรเจกต์นี้ — `.dockerignore`**

`COPY .` copy **ทุกไฟล์ในโฟลเดอร์** เข้า image รวมถึง `.git/`, `.github/`, `README.md`, `docker-compose.yml` ที่ไม่จำเป็นต้องอยู่ใน image เลย ทำให้ image ใหญ่ขึ้นโดยไม่จำเป็นและหลุดข้อมูล repo metadata เข้าไปด้วย

แก้โดยสร้างไฟล์ `.dockerignore` (เพื่อนที่ทำ lab ต่อควรเพิ่มเอง):
```
.git
.github
README.md
docker-compose.yml
```

**เช็คก่อน push จริง — build/run local**
```bash
docker build -t test .
docker run -p 8080:80 test
# เปิด http://localhost:8080 เช็คก่อนดันขึ้น server จริง — ไม่ต้องรอ CI/CD ทุกครั้งที่แก้ CSS
```

---
[← Architecture](01-architecture.md) | [กลับ README](../README.md) | [ถัดไป: docker-compose.yml →](03-docker-compose.md)
