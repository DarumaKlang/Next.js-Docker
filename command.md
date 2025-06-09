## สรุปคำสั่ง Docker และ Docker Compose ที่ใช้ในการ Deploy Next.js App

นี่คือชุดคำสั่งหลักที่คุณจะใช้ในการจัดการ Docker Containers และ Services สำหรับ Next.js Application ของคุณ:

### 1. คำสั่งหลักในการจัดการ Docker Compose Services

* **Build และรัน Services ทั้งหมดในโหมด Detached (ทำงานเบื้องหลัง):**
    คำสั่งนี้จะ Build Docker Images จาก `Dockerfile` (ถ้ามีการเปลี่ยนแปลง) และสร้าง/รัน Container ตามที่กำหนดใน `docker-compose.yml`
    ```bash
    docker compose up --build -d
    ```
    * `--build`: บังคับให้ Build Images ใหม่เสมอ (มีประโยชน์ในช่วงพัฒนา)
    * `-d`: รัน Container ในโหมด "detached" (ทำงานเบื้องหลัง)

* **หยุดและลบ Services ทั้งหมด:**
    คำสั่งนี้จะหยุด Container ทั้งหมดที่กำหนดใน `docker-compose.yml` และลบมันออกไป แต่จะไม่ลบ Docker Images ที่ Build ขึ้นมา
    ```bash
    docker compose down
    ```

* **หยุดและลบ Services พร้อมทั้งลบ Docker Images ที่เกี่ยวข้อง:**
    คำสั่งนี้จะทำเหมือน `docker compose down` แต่จะลบ Docker Images ที่ถูกสร้างขึ้นมา (โดย service ใน `docker-compose.yml`) ด้วย
    ```bash
    docker compose down --rmi all
    ```
    * `--rmi all`: ลบ Image ที่เกี่ยวข้องกับ Services ทั้งหมด

### 2. คำสั่งสำหรับการตรวจสอบและ Debug

* **ดูสถานะของ Docker Containers ที่กำลังทำงาน:**
    คำสั่งนี้จะแสดงรายชื่อ Container ที่กำลังทำงานอยู่ พร้อมข้อมูลสำคัญ เช่น ID, Image, Command, Status, Ports ที่ถูก Map และชื่อ Container
    ```bash
    docker ps
    ```

* **ดู Logs ของ Service/Container เฉพาะ:**
    นี่เป็นคำสั่งที่สำคัญที่สุดในการ Debug เมื่อ Container ไม่สามารถเริ่มต้นได้ หรือมีข้อผิดพลาดระหว่างการทำงาน
    * **ดู Logs ของ Next.js App Service:**
        ```bash
        docker compose logs nextjs-app
        ```
    * **ดู Logs ของ Nginx Service:**
        ```bash
        docker compose logs nginx
        ```
    * **ดู Logs ของ Service แบบ Real-time (ติดตาม Log ไปเรื่อยๆ):**
        เพิ่ม `--follow` หรือ `-f` เข้าไป
        ```bash
        docker compose logs -f nextjs-app
        ```

* **เข้าถึง Shell ภายใน Container (สำหรับการ Debug เชิงลึก):**
    คุณสามารถเข้าสู่ Shell (เช่น Bash) ภายใน Container เพื่อตรวจสอบไฟล์, รันคำสั่ง, หรือ Debug ปัญหาต่างๆ ได้
    ```bash
    docker compose exec nextjs-app sh
    # หรือถ้า Container นั้นมี bash
    # docker compose exec nextjs-app bash
    ```
    * `exec`: รันคำสั่งใน Container ที่กำลังทำงานอยู่
    * `nextjs-app`: ชื่อ Service ที่กำหนดใน `docker-compose.yml`
    * `sh` หรือ `bash`: คำสั่ง Shell ที่จะรันภายใน Container (เลือกอันที่มีอยู่ใน Image)

* **ดูรายชื่อ Docker Images ทั้งหมดในเครื่องของคุณ:**
    ```bash
    docker images
    ```

* **ดูรายละเอียดของ Docker Image (เช่น Layer History):**
    ```bash
    docker inspect <image_name_or_id>
    # ตัวอย่าง: docker inspect next-for-docker-nextjs-app
    ```

* **ลบ Docker Image (ที่ไม่ถูกใช้โดย Container):**
    ```bash
    docker rmi <image_name_or_id>
    # ตัวอย่าง: docker rmi next-for-docker-nextjs-app
    ```
    * หาก Image ถูกใช้โดย Container คุณต้องหยุดและลบ Container นั้นก่อน หรือใช้ `--force` (ไม่แนะนำหากไม่แน่ใจ)

---

ชุดคำสั่งเหล่านี้จะช่วยให้คุณสามารถจัดการ, ตรวจสอบ, และ Debug Dockerized Next.js Application ของคุณได้อย่างมีประสิทธิภาพครับ!