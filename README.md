# DevOps Subject Test — CMTC

Lab project สอน Docker + Docker Compose + GitHub Actions CI/CD — deploy static HTML site ขึ้น server จริงอัตโนมัติทุกครั้งที่ push เข้า `main`.

---

## 1. ภาพรวมสถาปัตยกรรม

```
┌─────────────┐   git push    ┌──────────────────┐   SSH    ┌─────────────────────┐
│  Local /    │ ────────────> │  GitHub Actions   │ ───────> │   Server (Linux)    │
│  GitHub repo│                │  (CI/CD pipeline) │          │   Docker Engine     │
└─────────────┘                └──────────────────┘          └─────────┬───────────┘
                                                                          │ docker compose up -d --build
                                                                          ▼
                                                              ┌─────────────────────┐
                                                              │ container: student01_web │
                                                              │ image: nginx:alpine  │
                                                              │ port 9834 → 80       │
                                                              └─────────────────────┘
```

**ส่วนประกอบ:**

| ไฟล์ | หน้าที่ |
|---|---|
| [`index.html`](index.html) | หน้าเว็บ static (ไม่มี backend) |
| [`Dockerfile`](Dockerfile) | สร้าง image: เอา nginx + copy ไฟล์ static เข้าไป |
| [`docker-compose.yml`](docker-compose.yml) | สั่งรัน container, ตั้ง port, network, restart policy |
| [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml) | Pipeline: validate HTML → SSH เข้า server → pull code → build → deploy |

---

## 2. Dockerfile — ทำอะไร

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

- `FROM nginx:alpine` — ใช้ web server nginx บน base image alpine (เบามาก ~40MB)
- `COPY . /usr/share/nginx/html` — copy ไฟล์ทั้งหมดในโปรเจกต์เข้า document root ของ nginx (path default ที่ nginx serve ไฟล์ static)

ไม่มี build step อื่นเพราะเป็น static site ล้วนๆ — ไม่มี npm build, ไม่มี backend compile

**⚠️ Best practice ที่ยังไม่ได้ทำในโปรเจกต์นี้ — `.dockerignore`**

`COPY . /usr/share/nginx/html` copy **ทุกไฟล์ใน directory** เข้า image รวมถึง `.git/`, `.github/`, `README.md`, `docker-compose.yml` ที่ไม่จำเป็นต้องอยู่ใน image เลย ทำให้ image ใหญ่ขึ้นโดยไม่จำเป็นและหลุดข้อมูล repo metadata เข้าไปด้วย

แก้โดยสร้างไฟล์ `.dockerignore` (เพื่อนที่ทำ lab ต่อควรเพิ่มเอง):
```
.git
.github
README.md
docker-compose.yml
```

---

## 3. docker-compose.yml — ทำอะไร

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

อธิบายทีละบรรทัด:

- **`build: .`** — build image จาก Dockerfile ใน directory นี้ (ไม่ได้ pull image สำเร็จรูป)
- **`container_name: student01_web`** — ตั้งชื่อ container ตายตัว เรียกใช้ `docker exec student01_web ...` ได้ง่าย
- **`restart: unless-stopped`** — container ตาย (crash, server reboot) ให้ Docker restart ให้อัตโนมัติ ยกเว้นมีคนสั่ง stop เอง
- **`ports: ["9834:80"]`** — map port 9834 ของเครื่อง host → port 80 ใน container (nginx ฟังที่ 80 ข้างใน) ทำให้เข้าได้ตรงผ่าน `http://<PUBLIC_IP>:9834`
- **`labels: traefik.*`** — metadata สำหรับ reverse proxy ชื่อ **Traefik** (ถ้ามีรันอยู่บน server) บอกว่า route domain `student01-lab.cmtc.ac.th` มาที่ service นี้ — *ปัจจุบัน server lab นี้ยังไม่มี Traefik รัน label พวกนี้เลยยังไม่มีผลอะไร แต่ทิ้งไว้เผื่ออนาคต*
- **`networks: web` (external: true)** — เชื่อม container เข้า Docker network ชื่อ `web` ที่ต้องสร้างไว้ก่อนแล้วบน host (`docker network create web`) — ใช้สำหรับให้ Traefik หรือ container อื่นคุยกันได้ผ่าน network เดียวกัน

---

## 4. GitHub Actions Pipeline — ทำงานยังไง

ไฟล์: [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml)

Trigger: ทุกครั้งที่ `git push` เข้า branch `main`

### Step 1 — Checkout
ดึง source code ล่าสุดเข้า runner ของ GitHub Actions

### Step 2 — Test (validate HTML)
```bash
sudo apt-get update && sudo apt-get install -y tidy
tidy -q -e index.html; [ $? -le 1 ]
```
รัน `tidy` เช็ค syntax ของ `index.html` — ยอมให้ผ่านถ้า exit code ≤ 1 (warning ผ่านได้ แต่ error จริงจะ fail)

### Step 3 — Deploy (SSH เข้า server)
ใช้ action `appleboy/ssh-action` SSH เข้า server ด้วย credential ใน **GitHub Secrets**:

| Secret | ใช้ทำอะไร |
|---|---|
| `SERVER_IP` | IP server ปลายทาง |
| `SERVER_USER` | username SSH |
| `SERVER_PASSWORD` | password SSH |

บน server มันรันสคริปต์นี้ (ของจริงจาก [`deploy.yml`](.github/workflows/deploy.yml)):
```bash
if [ -d ~/app/.git ]; then
  git -C ~/app remote set-url origin https://x-access-token:${{ github.token }}@github.com/${{ github.repository }}.git
  git -C ~/app pull
  git -C ~/app remote set-url origin https://github.com/${{ github.repository }}.git
else
  git clone https://x-access-token:${{ github.token }}@github.com/${{ github.repository }}.git ~/app
  git -C ~/app remote set-url origin https://github.com/${{ github.repository }}.git
fi
cd ~/app
docker compose up -d --build
```

`docker compose up -d --build` = build image ใหม่ + รัน container แบบ background (`-d`) แทนที่ตัวเก่า

**จุดสำคัญด้าน security — ทำไมต้อง `remote set-url` สองรอบ:**

`${{ github.token }}` คือ token ชั่วคราวที่ GitHub Actions ออกให้เอง อายุสั้น (หมดอายุเมื่อ job จบ) สคริปต์ทำ:
1. **ใส่ token เข้า remote URL ชั่วคราว** → ใช้ pull/clone โดยไม่ต้องตั้ง `SERVER_PASSWORD` เป็น git credential ถาวร
2. **`pull` เสร็จปุ๊บ → เซต URL กลับเป็นไม่มี token ทันที** → ป้องกัน token ค้างอยู่ใน `~/app/.git/config` บน server (ถ้าใครมาอ่านไฟล์ config ทีหลังจะไม่เจอ credential หลุด)

นี่คือ pattern ที่ถูกต้องสำหรับ CI/CD: ใช้ credential ที่ **อายุสั้นที่สุดเท่าที่จำเป็น** แล้วเช็ดทิ้งทันทีหลังใช้งาน — ไม่ใช่ฝัง token ถาวรไว้บน disk ของ server ปลายทาง

---

## 5. Setup จากศูนย์ — ทำตามนี้ทีละขั้น

### บน GitHub (ครั้งเดียว)
1. ไปที่ repo → **Settings → Secrets and variables → Actions**
2. เพิ่ม secret 3 ตัว: `SERVER_IP`, `SERVER_USER`, `SERVER_PASSWORD`

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

## 6. ปัญหาที่เจอจริงระหว่าง lab นี้ (และวิธีแก้)

| อาการ | สาเหตุ | วิธีแก้ |
|---|---|---|
| `permission denied ... docker.sock` | user SSH ไม่อยู่ใน `docker` group (หรืออยู่แล้วแต่ session เก่ายังไม่ refresh) | `sudo usermod -aG docker $USER` แล้ว **SSH เข้าใหม่** (ไม่ใช่แค่ `groups $USER` เพราะอันนั้นอ่านจาก `/etc/group` ตรงๆ ไม่ใช่ active session) |
| `network web declared as external, but could not be found` | docker-compose.yml อ้างถึง network `web` ที่ยังไม่ถูกสร้าง | `docker network create web` |
| เข้า `http://<PUBLIC_IP>` ไม่ได้ | docker-compose.yml ตอนแรกไม่มี `ports:` — container เข้าถึงได้แค่ผ่าน Traefik (ซึ่ง server lab นี้ไม่มีรัน) | เพิ่ม `ports: ["9834:80"]` ใน docker-compose.yml แล้ว push ใหม่ |

---

## 7. HTTPS — ทำต่อยังไง (ยังไม่ได้ทำใน lab นี้)

ตอนนี้ site รันเป็น **HTTP เปิด port ตรง** (`http://IP:9834`) — Let's Encrypt **ออก cert ให้ IP ตรงๆ ไม่ได้** ต้องมี domain เท่านั้น

ทางเลือก:

1. **Cloudflare proxy (ง่ายสุด)** — ถ้า domain อยู่หลัง Cloudflare (orange cloud เปิด) Cloudflare จะ terminate TLS ให้ฟรี โดยที่ server ข้างหลังยังเป็น HTTP ปกติ ไม่ต้องแก้ docker-compose เลย
2. **Traefik + Let's Encrypt (ของจริง)** — labels ใน [`docker-compose.yml`](docker-compose.yml) เตรียม `traefik.*` ไว้แล้ว แต่ server lab นี้ยังไม่มี Traefik รันอยู่ ต้อง deploy Traefik container แยกต่างหากก่อน ถึงจะ route domain `student01-lab.cmtc.ac.th` ให้อัตโนมัติพร้อมออก cert ให้เองได้

---

## 8. คำสั่งที่ใช้บ่อย (cheat sheet)

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

## 9. สรุป concept สำคัญสำหรับสอนเพื่อน

1. **Dockerfile** = สูตรสร้าง image (เปรียบเหมือน recipe)
2. **docker-compose.yml** = วิธีรัน container จาก image นั้น (network, port, restart policy, labels)
3. **External network** ใน compose ต้องมีอยู่แล้วบน host ก่อน — compose ไม่สร้างให้เองถ้าเป็น `external: true`
4. **Group membership ใน Linux ไม่ refresh แบบ real-time** — ต้อง re-login session ถึงจะมีผล (เป็นกับดักที่เจอบ่อยมาก)
5. **`ports:` vs reverse proxy label** — สองทางเข้าเว็บคนละแบบ เลือกใช้ตามว่ามี reverse proxy (Traefik/Nginx) อยู่หน้า server หรือไม่
6. **CI/CD pipeline = automate สิ่งที่เคยทำมือ** — validate → SSH → pull → rebuild → restart ทุก push โดยไม่ต้องเข้า server เอง
