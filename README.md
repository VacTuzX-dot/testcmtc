# DevOps Subject Test — CMTC

Lab project สอน Docker + Docker Compose + GitHub Actions CI/CD — deploy static HTML site ขึ้น server จริงอัตโนมัติทุกครั้งที่ push เข้า `main`.

---

## TL;DR — เริ่มใช้งานเร็ว

ทำ 5 ขั้นนี้ก็ deploy ได้เลย เหตุผล/รายละเอียดของแต่ละขั้นอยู่ในเอกสารแยกไฟล์ด้านล่าง (เปิดอ่านตอนอยากรู้ลึกพอ)

**1. ตั้ง GitHub Secrets** — repo → Settings → Secrets and variables → Actions:
`SERVER_IP`, `SERVER_USER`, `SERVER_PASSWORD`

**2. เตรียม server** (ทำครั้งเดียว):
```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER && exit   # ต้อง SSH เข้าใหม่หลังจากนี้
docker network create web
```

**3. Push โค้ด** — pipeline รันเอง (build → test → deploy):
```bash
git push origin main
```

**4. เช็คผล** — tab **Actions** บน GitHub หรือ:
```bash
docker ps
```

**5. เปิดเว็บ**: `http://<PUBLIC_IP>:9834`

**สอนเพื่อน?** ก็อป [`docker-compose.template.yml`](docker-compose.template.yml) ไปแทน `docker-compose.yml` ของตัวเอง แก้แค่เลขนักศึกษา 2 จุดตามในไฟล์

ติดปัญหา? ดู [docs/06-troubleshooting.md](docs/06-troubleshooting.md) — มีคำตอบของ error ที่เจอบ่อยที่สุดครบแล้ว

---

## สารบัญ — รายละเอียดแยกไฟล์

เอกสารแยกเป็นไฟล์ย่อยตามหัวข้อ กดลิงก์เข้าไปอ่านรายละเอียดแต่ละเรื่องได้เลย:

| # | หัวข้อ | เนื้อหาในไฟล์ |
|---|---|---|
| 1 | [Architecture Overview](docs/01-architecture.md) | ภาพรวม flow ทั้งระบบตั้งแต่ push จนเว็บขึ้น + ตารางไฟล์สำคัญของโปรเจกต์ |
| 2 | [Dockerfile](docs/02-dockerfile.md) | อธิบายทีละบรรทัดว่า image ถูกสร้างยังไง + วิธี build/run local ก่อน push |
| 3 | [docker-compose.yml](docs/03-docker-compose.md) | อธิบายทีละ field ว่าทำไมต้องตั้งค่าแบบนี้ |
| 4 | [CI/CD Pipeline](docs/04-cicd-pipeline.md) | 3 jobs (build/test/deploy) ทำงานยังไง + security pattern ของ token |
| 5 | [Setup จากศูนย์](docs/05-setup-guide.md) | ทำตามทีละขั้น ตั้งแต่ GitHub Secrets จนถึง deploy สำเร็จ |
| 6 | [Troubleshooting](docs/06-troubleshooting.md) | ปัญหาที่เจอจริงระหว่างทำ lab นี้ + ลำดับ debug ทั่วไป |
| 7 | [HTTPS](docs/07-https.md) | ทำไมยังไม่มี HTTPS และทำต่อยังไง |
| 8 | [Cheat Sheet](docs/08-cheatsheet.md) | คำสั่ง docker ที่ใช้บ่อย |
| 9 | [สรุป Concept](docs/09-concepts.md) | concept สำคัญ สรุปให้สอนเพื่อนต่อได้ |
| 10 | [Port Assignment Sheet](docs/10-port-assignments.md) | ตาราง port/container name สำหรับเพื่อน 37 คน กันชนกัน |

---

## โครงสร้างโปรเจกต์

```
.
├── index.html                        # หน้าเว็บที่ deploy จริง
├── Dockerfile                        # สร้าง image
├── docker-compose.yml                # สั่งรัน container
├── .github/workflows/deploy.yml      # CI/CD pipeline
├── README.md                         # ไฟล์นี้ — TL;DR + สารบัญ
└── docs/                             # รายละเอียดแยกตามหัวข้อ (1 ไฟล์ = 1 เรื่อง)
    ├── 01-architecture.md
    ├── 02-dockerfile.md
    ├── 03-docker-compose.md
    ├── 04-cicd-pipeline.md
    ├── 05-setup-guide.md
    ├── 06-troubleshooting.md
    ├── 07-https.md
    ├── 08-cheatsheet.md
    └── 09-concepts.md
```
