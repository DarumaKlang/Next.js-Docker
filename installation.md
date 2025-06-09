# Next.js Application Deployment with Docker and Nginx

นี่คือคำแนะนำทีละขั้นตอนสำหรับการติดตั้งและ Deploy Next.js Application โดยใช้ Docker สำหรับการจัดการ Environment และ Nginx เป็น Reverse Proxy

## 1. โครงสร้างโปรเจกต์ (Project Structure)

โปรเจกต์ของคุณควรมีโครงสร้างดังนี้:

```
your-nextjs-docker-project/
├── .gitignore               # (Optional) สำหรับ Git
├── docker-compose.yml       # กำหนด Docker services
├── Dockerfile               # Dockerfile สำหรับ Next.js application
├── next.config.js           # Configuration สำหรับ Next.js
├── package.json             # Next.js project dependencies
├── package-lock.json        # Next.js project dependency lock file
├── public/                  # Static assets สำหรับ Next.js (รูปภาพ, favicon, ฯลฯ)
│   ├── file.svg
│   ├── globe.svg
│   ├── next.svg
│   ├── vercel.svg
│   └── window.svg
├── src/                     # Source code ของ Next.js (เช่น app/page.tsx)
│   └── app/
│       └── page.tsx
└── nginx/                   # โฟลเดอร์สำหรับ Nginx configuration
    └── nginx.conf           # Nginx configuration file
```

---

## 2. ขั้นตอนการติดตั้งและเตรียมการ

### 2.1. สร้าง Next.js Project (ถ้ายังไม่มี)

หากคุณยังไม่มี Next.js project คุณสามารถสร้างใหม่ได้โดยใช้ `create-next-app`:

```bash
npx create-next-app@latest my-nextjs-app
```

* **เมื่อระบบถาม:**
    * `Would you like to use TypeScript?` **Yes**
    * `Would you like to use ESLint?` **Yes**
    * `Would you like to use Tailwind CSS?` **Yes** (หรือไม่ก็ได้ ตามต้องการ)
    * `Would you like to use `src/` directory?` **Yes**
    * `Would you like to use App Router? (recommended)` **Yes**
    * `Would you like to customize the default import alias (@/*)?` **No**

จากนั้น, **ย้ายเข้าไปในโฟลเดอร์โปรเจกต์ Next.js ของคุณ:**

```bash
cd my-nextjs-app
```

จากนี้ไป คำสั่งทั้งหมดจะรันจาก `my-nextjs-app` (หรือชื่อโปรเจกต์ของคุณ)

### 2.2. สร้างโฟลเดอร์ `nginx` และไฟล์ `nginx.conf`

สร้างโฟลเดอร์ชื่อ `nginx` ใน root ของโปรเจกต์ Next.js ของคุณ และสร้างไฟล์ชื่อ `nginx.conf` ภายใน:

```bash
mkdir nginx
touch nginx/nginx.conf
```

---

## 3. การกำหนดค่า (Configuration)

### 3.1. ไฟล์ `next.config.js`

เปิดไฟล์ `next.config.js` และเพิ่ม `output: 'standalone'` เพื่อให้ Next.js สร้าง build output ที่เหมาะกับการรันใน Docker:

**`next.config.js`**
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone', // เพิ่มบรรทัดนี้
};

module.exports = nextConfig;
```

### 3.2. ไฟล์ `Dockerfile`

สร้างหรือแก้ไขไฟล์ `Dockerfile` ใน root ของโปรเจกต์ เพื่อกำหนดวิธีการ Build และ Run Next.js Application ใน Docker Container:

**`Dockerfile`**
```dockerfile
# Dockerfile สำหรับ Next.js Application (ใช้ npm)

# ----- Stage 1: Build the Next.js application (ขั้นตอนการสร้าง) -----
# ใช้ Node.js version 20 (alpine คือเวอร์ชั่นขนาดเล็ก) เป็น base image สำหรับ build
FROM node:20-alpine AS builder

# กำหนด Working Directory ภายใน Container สำหรับ stage นี้
WORKDIR /app

# (Step 1) คัดลอก package.json และ package-lock.json มาก่อน
# การทำแบบนี้จะช่วยให้ Docker สามารถใช้ build cache ได้อย่างมีประสิทธิภาพ
# หากไฟล์เหล่านี้ไม่มีการเปลี่ยนแปลง Docker จะไม่ติดตั้ง dependencies ใหม่
COPY package.json package-lock.json ./

# (Step 2) ติดตั้ง dependencies ทั้งหมดที่ระบุใน package.json
RUN npm ci

# (Step 3) คัดลอกโค้ดแอปพลิเคชันที่เหลือทั้งหมดจากเครื่อง Host ไปยัง Working Directory ใน Container
# รวมถึงโฟลเดอร์ public ด้วย
COPY . .

# (Step 4) สร้าง Next.js application สำหรับ production
# คำสั่งนี้จะสร้างไฟล์และโฟลเดอร์ที่จำเป็น (.next) สำหรับการรัน Next.js ในโหมด production
# เนื่องจากเราได้ตั้งค่า `output: 'standalone'` ใน next.config.js
# ผลลัพธ์ส่วนใหญ่ที่จำเป็นจะอยู่ใน .next/standalone และไฟล์ server.js
RUN npm run build

# ----- Stage 2: Run the Next.js application in production (ขั้นตอนการรัน) -----
# ใช้ Node.js version 20 จาก alpine distribution อีกครั้ง เป็น base image สำหรับรัน
# เลือกเวอร์ชั่นที่เบาที่สุดเพื่อลดขนาด Image สุดท้าย
FROM node:20-alpine AS runner

# กำหนด Working Directory ภายใน Container สำหรับ stage นี้
WORKDIR /app

# ตั้งค่า Environment Variable สำหรับ Next.js ให้เป็นโหมด Production
ENV NODE_ENV production

# ปิดการส่งข้อมูล Telemetry ของ Next.js (ไม่จำเป็นในการ Production)
ENV NEXT_TELEMETRY_DISABLED 1

# คัดลอกไฟล์ที่จำเป็นจาก Stage "builder" มายัง Stage "runner"
# เนื่องจากเราใช้ output: 'standalone' ใน next.config.js
# ไฟล์ทั้งหมดที่จำเป็นในการรัน server (รวมถึง server.js และ node_modules ที่จำเป็น)
# จะถูกสร้างไว้ใน /app/.next/standalone
COPY --from=builder /app/.next/standalone ./

# คัดลอกโฟลเดอร์ public (สำหรับ static assets ของคุณเอง เช่น รูปภาพ, favicon)
# โฟลเดอร์ public นี้อยู่ที่ root ของ project และจะถูก serve โดย Nginx
COPY --from=builder /app/public ./public

# คัดลอก static assets ที่ Next.js สร้างขึ้นมา (อยู่ใน .next/static)
# สิ่งนี้จำเป็นสำหรับ Nginx configuration ที่เราตั้งค่าไว้เพื่อ serve ไฟล์เหล่านี้โดยตรง
COPY --from=builder /app/.next/static ./.next/static

# เปิดพอร์ต 3000 ภายใน Next.js Container
# นี่คือพอร์ตที่ Next.js server กำลังฟังอยู่
EXPOSE 3000

# คำสั่งที่จะรันเมื่อ Container เริ่มต้นทำงาน
# เมื่อใช้ standalone output, Next.js จะสร้างไฟล์ server.js ไว้ใน root ของ standalone output
# ดังนั้น เราสามารถเรียกใช้ server.js ได้โดยตรงด้วย Node.js
CMD ["node", "server.js"]
```

### 3.3. ไฟล์ `docker-compose.yml`

สร้างหรือแก้ไขไฟล์ `docker-compose.yml` ใน root ของโปรเจกต์ เพื่อกำหนด Docker services สำหรับ Next.js และ Nginx:

**`docker-compose.yml`**
```yaml
version: '3.8' # (คุณสามารถลบ attribute 'version' นี้ออกได้ใน Docker Compose v2+)

services:
  nextjs-app:
    build:
      context: . # ระบุว่า Dockerfile อยู่ใน current directory
      dockerfile: Dockerfile # ระบุชื่อ Dockerfile
    ports:
      - '3000:3000' # Map พอร์ต 3000 ของ Host ไปยังพอร์ต 3000 ของ Next.js Container (สำหรับการเข้าถึงโดยตรงถ้าต้องการ)
    environment:
      # ตั้งค่า environment variables ที่จำเป็นสำหรับ Next.js ใน production
      - NODE_ENV=production
      - NEXT_TELEMETRY_DISABLED=1
    restart: always # กำหนดให้ Container Restart โดยอัตโนมัติหากหยุดทำงาน

  nginx:
    image: nginx:stable-alpine # ใช้ Nginx official image ที่มีขนาดเล็ก
    ports:
      - '80:80' # Map พอร์ต 80 ของ Host ไปยังพอร์ต 80 ของ Nginx Container (สำหรับการเข้าถึงผ่าน browser)
    volumes:
      # Mount configuration file ของ Nginx จากโฟลเดอร์ ./nginx/nginx.conf
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - nextjs-app # กำหนดให้ Nginx เริ่มหลังจาก nextjs-app เริ่มทำงาน เพื่อให้ Next.js app พร้อมใช้งานก่อน
    restart: always # กำหนดให้ Container Restart โดยอัตโนมัติหากหยุดทำงาน
```

### 3.4. ไฟล์ `nginx/nginx.conf`

สร้างหรือแก้ไขไฟล์ `nginx/nginx.conf` เพื่อตั้งค่า Nginx เป็น Reverse Proxy สำหรับ Next.js App และ Serve Static Files:

**`nginx/nginx.conf`**
```nginx
worker_processes 1; # จำนวน worker processes ที่ Nginx จะใช้ (ปกติ 1-4)

events {
    worker_connections 1024; # จำนวน connection สูงสุดที่ worker process หนึ่งๆ สามารถจัดการได้
}

http {
    include mime.types;     # รวมประเภท MIME มาตรฐาน (เช่น text/html, image/jpeg)
    default_type application/octet-stream; # กำหนด Content-Type เริ่มต้นสำหรับไฟล์ที่ไม่รู้จัก

    sendfile on;           # เปิดใช้งาน sendfile เพื่อเพิ่มประสิทธิภาพการส่งไฟล์
    keepalive_timeout 65; # กำหนด timeout สำหรับ keep-alive connections (เวลาที่ connection จะถูกเปิดทิ้งไว้)

    # กำหนด upstream สำหรับ Next.js application
    # 'nextjs-app' คือชื่อ service ที่เรากำหนดใน docker-compose.yml
    # '3000' คือพอร์ตที่ Next.js App listens ภายใน Container
    upstream nextjs_upstream {
        server nextjs-app:3000;
    }

    server {
        listen 80; # Nginx จะฟัง request ที่พอร์ต 80 ของ Container
        server_name localhost; # กำหนดชื่อ Server (สามารถเปลี่ยนเป็น IP address หรือ domain name ของคุณได้)

        # Location block สำหรับการส่งต่อ Request ทั้งหมดไปยัง Next.js App
        location / {
            proxy_pass http://nextjs_upstream; # ส่งต่อ Request ไปยัง Next.js App
            proxy_http_version 1.1;             # ใช้ HTTP/1.1
            proxy_set_header Upgrade $http_upgrade; # สำหรับ WebSockets
            proxy_set_header Connection 'upgrade';   # สำหรับ WebSockets
            proxy_set_header Host $host;             # ส่งต่อ Host header ไปยัง Next.js App
            proxy_cache_bypass $http_upgrade;       # ไม่ Cache หากเป็น Upgrade request
        }

        # Location block สำหรับ Next.js static assets ที่ถูกสร้างขึ้น (อยู่ใน .next/static)
        # Nginx จะส่งต่อ Request สำหรับไฟล์เหล่านี้ไปยัง Next.js App
        location /_next/static/ {
            proxy_pass http://nextjs_upstream; # ส่งต่อ Request ไปยัง Next.js App
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        # คุณสามารถเพิ่ม location block สำหรับ API ได้ถ้า Next.js มี API Routes
        # location /api/ {
        #     proxy_pass http://nextjs_upstream;
        # }
    }
}
```

### 3.5. ตรวจสอบโฟลเดอร์ `public`

ตรวจสอบให้แน่ใจว่าโฟลเดอร์ `public` มีอยู่จริงใน root directory ของ Next.js project ของคุณ (`/media/devg/GitHub/GitHub/DarumaKlang/Next.js-Docker/next-for-docker/public`) หากไม่มี ให้สร้างขึ้นมา:

```bash
mkdir -p public
```
(ปกติ `create-next-app` จะสร้างให้โดยอัตโนมัติ)

---

## 4. การ Deploy (Build & Run)

เมื่อคุณตั้งค่าไฟล์ทั้งหมดเรียบร้อยแล้ว ให้ทำตามขั้นตอนเหล่านี้เพื่อ Build และรัน Docker Containers:

1.  **ลบ Docker Containers และ Images เก่า (เพื่อเคลียร์ Cache และรับรองว่าใช้โค้ดใหม่ทั้งหมด):**
    เปิด Terminal ใน root directory ของโปรเจกต์ของคุณ (`your-nextjs-docker-project/`) และรัน:
    ```bash
    docker compose down --rmi all
    ```

2.  **Build และรัน Container ในโหมด Detached (ทำงานเบื้องหลัง):**
    ```bash
    docker compose up --build -d
    ```

    * `--build`: จะบังคับให้ Docker สร้าง Image ใหม่จาก `Dockerfile` ทุกครั้ง (ดีสำหรับการพัฒนา)
    * `-d`: จะรัน Container ในโหมด "detached" (เบื้องหลัง) ทำให้ Terminal ของคุณว่าง

## 5. การทดสอบ (Testing)

เมื่อ Container ทำงานแล้ว ให้เปิด Web Browser และไปที่:

* `http://localhost`

คุณควรจะเห็นหน้าเริ่มต้นของ Next.js application แสดงขึ้นมา

### คำสั่ง Docker ที่มีประโยชน์ในการ Debug:

* **ตรวจสอบสถานะ Container:**
    ```bash
    docker ps
    ```
* **ดู Logs ของ Next.js Container:**
    ```bash
    docker compose logs nextjs-app
    ```
* **ดู Logs ของ Nginx Container:**
    ```bash
    docker compose logs nginx
    ```
* **หยุดและลบ Container ทั้งหมด (โดยไม่ลบ Image):**
    ```bash
    docker compose down
    ```

---

หวังว่าคู่มือนี้จะสมบูรณ์และเป็นประโยชน์สำหรับการบันทึกไว้ใน GitHub และการนำไปใช้ในอนาคตนะครับ!

ข้อมูลเพิ่มเติม
-   [สรุปคำสั่ง Docker และ Docker Compose ที่ใช้ในการ Deploy Next.js App](commander.md)