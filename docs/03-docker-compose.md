# 3. docker-compose.yml — ทำอะไร

```yaml
services:
  web:
    build: .
    container_name: student01_web
    restart: unless-stopped
    ports:
      - "9834:80"
    labels:
      - traefik.enable=true
      - traefik.http.routers.student01.rule=Host(`student01-lab.cmtc.ac.th`)
      - traefik.http.services.student01.loadbalancer.server.port=80
    networks:
      - web

networks:
  web:
    external: true
```

**ทำไมใช้ docker-compose แทน `docker run` ตรงๆ:** `docker run` ต้องพิมพ์ flag ยาวๆ ทุกครั้ง (`-p`, `--name`, `--restart`, `--network`, ...) ส่วน compose เขียนเป็นไฟล์ครั้งเดียวแล้วสั่ง `docker compose up` ซ้ำได้เรื่อยๆ — ค่าที่ตั้งไว้จะเหมือนเดิมทุกครั้ง ไม่พิมพ์ผิด ไม่ลืม flag

อธิบายทีละบรรทัด:

- **`build: .`** — build image จาก Dockerfile ใน directory นี้ (ไม่ได้ pull image สำเร็จรูปจาก Docker Hub)
- **`container_name: student01_web`** — ตั้งชื่อ container ตายตัว เรียกใช้ `docker exec student01_web ...` ได้ง่าย ไม่ต้องมานั่งหา container ID สุ่มๆ
- **`restart: unless-stopped`** — container ตาย (crash, server reboot) ให้ Docker restart ให้อัตโนมัติ ยกเว้นมีคนสั่ง stop เอง
- **`ports: ["9834:80"]`** — map port 9834 ของเครื่อง host → port 80 ใน container (nginx ฟังที่ 80 ข้างใน) ทำให้เข้าได้ตรงผ่าน `http://<PUBLIC_IP>:9834`
- **`labels: traefik.*`** — metadata สำหรับ reverse proxy ชื่อ **Traefik** (ถ้ามีรันอยู่บน server) บอกว่า route domain `student01-lab.cmtc.ac.th` มาที่ service นี้ — *ปัจจุบัน server lab นี้ยังไม่มี Traefik รัน label พวกนี้เลยยังไม่มีผลอะไร แต่ทิ้งไว้เผื่ออนาคต*
- **`networks: web` (external: true)** — เชื่อม container เข้า Docker network ชื่อ `web` ที่ต้องสร้างไว้ก่อนแล้วบน host (`docker network create web`) — ใช้สำหรับให้ Traefik หรือ container อื่นคุยกันได้ผ่าน network เดียวกัน

---
[← Dockerfile](02-dockerfile.md) | [กลับ README](../README.md) | [ถัดไป: CI/CD Pipeline →](04-cicd-pipeline.md)
