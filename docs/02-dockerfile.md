# 2. Dockerfile — ทำอะไร

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

Docker image สร้างเป็น "ชั้น" (layer) ทับกัน แต่ละคำสั่งใน Dockerfile คือ 1 ชั้น อ่านทีละบรรทัด:

- **`FROM nginx:alpine`** — เริ่มจาก image สำเร็จรูปที่มี nginx (web server) ติดตั้งไว้แล้ว บน base OS "alpine" (Linux เบามาก ~5MB) รวมแล้ว image ทั้งก้อน ~40MB — เทียบกับ `nginx` เฉยๆ (ไม่มี `-alpine`) ที่หนักหลายร้อย MB
- **`COPY . /usr/share/nginx/html`** — copy ไฟล์ทั้งหมดจาก build context (โฟลเดอร์ปัจจุบันตอนรัน `docker build`) เข้าไปที่ `/usr/share/nginx/html` ข้างใน image — path นี้คือ **document root default** ของ nginx คือโฟลเดอร์ที่ nginx serve ไฟล์ static ออกมาให้ browser

ไม่มี build step อื่นเพราะเป็น static site ล้วนๆ (ไม่มี npm build, ไม่มี backend compile, ไม่มี dependency ต้อง install)

**🔴 เคยเป็นช่องโหว่จริง — `.dockerignore` มีอยู่แล้วแต่ไม่ครบ**

`COPY .` copy **ทุกไฟล์ในโฟลเดอร์** เข้า image ยกเว้นที่ list ไว้ใน [`.dockerignore`](../.dockerignore) ก่อนหน้านี้ไฟล์นั้น**ไม่ได้กัน** `.git/`, `README.md`, `docs/` ไว้ — ผลคือ `.git/` (ประวัติ commit ทั้งหมด) กับเอกสารทุกไฟล์ถูก serve ออกทาง URL จริงบน production (เจอผ่าน `curl http://<IP>:9834/.git/config` ได้ HTTP 200) ตอนนี้แก้แล้ว ไฟล์ปัจจุบันกันครบ:
```
.git
.github
docker-compose.yml
Dockerfile
.dockerignore
README.md
docs
```

**เช็คก่อน push จริง — build/run local**
```bash
docker build -t test .
docker run -p 8080:80 test
# เปิด http://localhost:8080 เช็คก่อนดันขึ้น server จริง — ไม่ต้องรอ CI/CD ทุกครั้งที่แก้ CSS
```

---
[← Architecture](01-architecture.md) | [กลับ README](../README.md) | [ถัดไป: docker-compose.yml →](03-docker-compose.md)
