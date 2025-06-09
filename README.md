# Next.js Application Deployment with Docker and Nginx

[![Linux](https://img.shields.io/badge/OS-Linux-FCC624?style=flat-square&logo=linux&logoColor=black)](https://www.linux.org/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-E95420?logo=ubuntu&logoColor=white&style=flat-square)](https://ubuntu.com/)
[![Uses Docker](https://img.shields.io/badge/Uses-Docker-blue?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![Made by Vercel](https://img.shields.io/badge/MADE%20BY-VERCEL-black?style=flat-square)](https://vercel.com/)
[![npm version](https://img.shields.io/badge/npm-V15.3.3-blue?style=flat-square)](https://www.npmjs.com/package/your-package-name)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](https://opensource.org/licenses/MIT)

โปรเจกต์นี้เป็นตัวอย่างและเทมเพลตสำหรับการ Deploy Next.js Application โดยใช้ Docker สำหรับการจัดการ Environment และ Nginx เป็น Reverse Proxy เพื่อ Serve Static Files และ Proxy Requests ไปยัง Next.js App

## โครงสร้างโปรเจกต์

```
your-nextjs-docker-project/
├── .gitignore               # สำหรับ Git เพื่อ ignore ไฟล์ที่ไม่จำเป็น
├── docker-compose.yml       # กำหนด Docker services (Next.js App และ Nginx)
├── Dockerfile               # Dockerfile สำหรับ Next.js application (Multi-stage build)
├── next.config.js           # Configuration สำหรับ Next.js (ตั้งค่า output: 'standalone')
├── package.json             # Next.js project dependencies
├── package-lock.json        # Next.js project dependency lock file
├── public/                  # Static assets สำหรับ Next.js (รูปภาพ, favicon, ฯลฯ)
│   └── (ไฟล์ static assets ของคุณ)
├── src/                     # Source code ของ Next.js (เช่น app/page.tsx)
│   └── app/
│       └── page.tsx
└── nginx/                   # โฟลเดอร์สำหรับ Nginx configuration
    └── nginx.conf           # Nginx configuration file
```

## การนำไปใช้งาน (Getting Started)

### Prerequisites (สิ่งที่ต้องมีก่อนเริ่ม)

ก่อนที่จะเริ่มใช้งานโปรเจกต์นี้ คุณต้องติดตั้งซอฟต์แวร์ต่อไปนี้บนเครื่องของคุณ:

* **Git**: สำหรับการ Clone Repository
    * [Download Git](https://git-scm.com/downloads)
* **Docker Desktop**: ซึ่งรวม Docker Engine และ Docker Compose ไว้ในตัว
    * [Download Docker Desktop](https://www.docker.com/products/docker-desktop)

### ขั้นตอนการใช้งาน

ทำตามขั้นตอนเหล่านี้เพื่อ Clone โปรเจกต์, Build Images และรัน Container:

1.  **Clone Repository:**
    เปิด Terminal หรือ Command Prompt และใช้คำสั่ง `git clone` เพื่อดาวน์โหลดโปรเจกต์นี้:

    ```bash
    git clone https://github.com/DarumaKlang/Next.js-Docker.git
    ```

2.  **เข้าสู่โฟลเดอร์โปรเจกต์:**

    ```bash
    cd next-for-docker
    ```

3.  **Build Docker Images และรัน Containers:**
    ใช้ Docker Compose เพื่อ Build Images และเริ่มต้น Services ทั้ง Next.js App และ Nginx ในโหมดเบื้องหลัง:

    ```bash
    docker compose up --build -d
    ```
    * `--build`: จะบังคับให้ Docker Build Images ใหม่จาก `Dockerfile` เสมอ ซึ่งสำคัญในการใช้งานครั้งแรกหรือเมื่อมีการเปลี่ยนแปลงโค้ด
    * `-d`: จะรัน Container ในโหมด "detached" (ทำงานเบื้องหลัง) ทำให้ Terminal ของคุณว่าง

4.  **เข้าถึงแอปพลิเคชัน:**
    เมื่อ Container ทำงานขึ้นมาแล้ว (อาจใช้เวลาสักครู่ในการ Build ครั้งแรก) คุณสามารถเข้าถึง Next.js Application ของคุณได้โดยเปิด Web Browser และไปที่:

    ```
    http://localhost
    ```

    คุณควรจะเห็นหน้าเริ่มต้นของ Next.js application แสดงขึ้นมา

## คำสั่ง Docker ที่มีประโยชน์ (Useful Docker Commands)

นี่คือคำสั่ง Docker และ Docker Compose ที่มีประโยชน์สำหรับการจัดการและ Debugging โปรเจกต์นี้:

* **ดูสถานะของ Docker Containers ที่กำลังทำงาน:**
    ```bash
    docker ps
    ```

* **ดู Logs ของ Next.js App Service (สำหรับการ Debug):**
    ```bash
    docker compose logs nextjs-app
    ```

* **ดู Logs ของ Nginx Service (สำหรับการ Debug):**
    ```bash
    docker compose logs nginx
    ```

* **ดู Logs ของ Service แบบ Real-time (ติดตาม Log ไปเรื่อยๆ):**
    ```bash
    docker compose logs -f nextjs-app
    ```

* **หยุดและลบ Services ทั้งหมด (ไม่ลบ Image):**
    ```bash
    docker compose down
    ```

* **หยุดและลบ Services พร้อมทั้งลบ Docker Images ที่เกี่ยวข้อง:**
    ```bash
    docker compose down --rmi all
    ```

* **เข้าถึง Shell ภายใน Next.js Container (สำหรับการ Debug เชิงลึก):**
    ```bash
    docker compose exec nextjs-app sh
    # หรือ
    # docker compose exec nextjs-app bash
    ```

## การนำไปใช้ต่อ (Further Usage)

โปรเจกต์นี้ถูกออกแบบมาให้เป็น Base Template ที่สามารถนำไปพัฒนาต่อได้ง่าย:

* **พัฒนาโค้ด Next.js ของคุณ:** ทำการแก้ไขไฟล์ในโฟลเดอร์ `src/` และ `public/` ได้ตามปกติ
* **เมื่อมีการเปลี่ยนแปลงโค้ด:** หลังจากแก้ไขโค้ด Next.js แล้ว ให้รัน `docker compose up --build -d` อีกครั้ง เพื่อ Build Image ใหม่และ Restart Container
* **ปรับแต่ง Nginx:** หากต้องการเพิ่ม SSL (HTTPS), ตั้งค่า Domain Name, หรือปรับแต่ง Nginx configurations เพิ่มเติม ให้แก้ไขไฟล์ `nginx/nginx.conf`
* **เพิ่ม Services อื่นๆ:** คุณสามารถเพิ่ม Services อื่นๆ เข้าไปใน `docker-compose.yml` ได้ เช่น Database (MongoDB, PostgreSQL), Redis, หรือ API Gateway อื่นๆ

## เครดิต (Credits)

โปรเจกต์นี้เป็นไปได้ด้วยซอฟต์แวร์และแพลตฟอร์ม Open Source ที่ยอดเยี่ยมเหล่านี้:

* **Next.js**: The React framework for the web.
    * [Website](https://nextjs.org/) | [GitHub](https://github.com/vercel/next.js)
* **React**: A JavaScript library for building user interfaces.
    * [Website](https://react.dev/) | [GitHub](https://github.com/facebook/react)
* **Node.js**: JavaScript runtime built on Chrome's V8 JavaScript engine.
    * [Website](https://nodejs.org/) | [GitHub](https://github.com/nodejs/node)
* **npm**: The package manager for JavaScript.
    * [Website](https://www.npmjs.com/)
* **Docker**: A platform for developing, shipping, and running applications in containers.
    * [Website](https://www.docker.com/) | [GitHub](https://github.com/docker)
* **Docker Compose**: A tool for defining and running multi-container Docker applications.
    * [Website](https://docs.docker.com/compose/) | [GitHub](https://github.com/docker/compose)
* **Nginx**: A high-performance HTTP and reverse proxy server.
    * [Website](https://nginx.org/) | [GitHub](https://github.com/nginx)

---

**Special Thanks (เครดิตพิเศษ):**

* **Gemini AI by Google:** สำหรับคำแนะนำ, การวิเคราะห์ปัญหา, และการให้โค้ด/ขั้นตอนการแก้ไขปัญหาที่ช่วยให้โปรเจกต์นี้สำเร็จลุล่วงไปได้

---
