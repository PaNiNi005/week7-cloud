# Cloud Deployment Analysis

---

## ส่วนที่ 1: ข้อมูล Deployment (10 คะแนน)

### 1.1 URLs ของระบบที่ Deploy

| Service | URL |
|---------|-----|
| Frontend | https://_________________________.railway.app |
| Backend API | https://_________________________.railway.app |
| Database | (Internal - ไม่มี public URL) |

### 1.2 Screenshot หลักฐาน (5 รูป)

1. [ ] Railway Dashboard แสดง 3 Services → `screenshots/01_railway_dashboard.png`
2. [ ] Frontend ทำงานบน Browser → `screenshots/02_frontend_browser.png`
3. [ ] API Health check response → `screenshots/03_api_health.png`
4. [ ] Logs แสดง requests → `screenshots/04_logs.png`
5. [ ] Metrics แสดง CPU/Memory → `screenshots/05_metrics.png`

---

## ส่วนที่ 2: เปรียบเทียบ Docker vs Cloud (15 คะแนน)

### 2.1 ความแตกต่างที่สังเกตเห็น (10 คะแนน)

| ด้าน | Docker (Week 6) | Railway (Week 7) |
|------|-----------------|------------------|
| เวลา Deploy | รัน `docker compose up` บนเครื่อง ใช้เวลา build image ทุกครั้ง | Push code ขึ้น GitHub แล้ว Railway build และ deploy ให้อัตโนมัติ |
| การตั้งค่า Network | กำหนดใน `docker-compose.yml` เอง เช่น custom bridge network | Railway จัดการ Internal Network ให้อัตโนมัติ ไม่ต้องตั้งค่าเอง |
| การจัดการ ENV | ใช้ไฟล์ `.env` บนเครื่อง อาจหลุดถ้า commit ผิดพลาด | ตั้งค่าใน Variables tab บน Dashboard ปลอดภัยกว่า และรองรับ Reference (`${{Postgres.DATABASE_URL}}`) |
| การดู Logs | ดูผ่าน `docker compose logs` ใน Terminal | ดูได้จาก Logs tab บน Railway Dashboard แบบ real-time |
| การ Scale | ต้องแก้ไข config ใน `docker-compose.yml` เอง | ปรับจำนวน Replicas ได้ผ่าน UI |

### 2.2 ข้อดี/ข้อเสีย ของแต่ละแบบ (5 คะแนน)

**Docker Local:**
- ข้อดี: ฟรีทั้งหมด ทำงานได้แม้ไม่มี internet ควบคุม environment ได้เต็มที่ เหมาะสำหรับ development และ testing
- ข้อเสีย: เข้าถึงได้เฉพาะ local network เท่านั้น ต้องจัดการ SSL, Backup, Monitoring เอง ไม่มี uptime 24/7 เมื่อปิดเครื่อง

**Railway Cloud:**
- ข้อดี: เข้าถึงได้จากทั่วโลก มี Auto SSL, Auto Deploy, Built-in Monitoring ไม่ต้องดูแล Infrastructure เอง
- ข้อเสีย: มีค่าใช้จ่าย ($5/เดือน) ต้องพึ่ง internet และ Railway platform ถ้า platform มีปัญหาระบบก็มีปัญหาด้วย

---

## ส่วนที่ 3: Cloud Service Models (10 คะแนน)

### 3.1 Railway เป็น Service Model แบบไหน?

[ ] IaaS   [x] PaaS   [ ] SaaS

เพราะ: Railway จัดการ Infrastructure ทั้งหมดให้ เช่น OS, Runtime (Node.js), Network, SSL และ Load Balancing โดยผู้ใช้รับผิดชอบแค่ Application Code และ Data เท่านั้น ซึ่งตรงกับนิยามของ Platform as a Service (PaaS)

### 3.2 ถ้าใช้ IaaS (เช่น AWS EC2) ต้องทำอะไรเพิ่มอีก? (ยกตัวอย่าง 4 ข้อ)

1. ติดตั้งและตั้งค่า OS (เช่น Ubuntu) รวมถึง security patches เอง
2. ติดตั้ง Runtime เอง เช่น Node.js, PostgreSQL และจัดการ version ให้เอง
3. ตั้งค่า SSL Certificate เอง เช่น ใช้ Let's Encrypt และต้อง renew เอง
4. ตั้งค่า Firewall, Load Balancer และ Reverse Proxy (เช่น Nginx) เอง

---

## ส่วนที่ 4: 12-Factor App Analysis (15 คะแนน)

### 4.1 Factors ที่เห็นจาก Lab (10 คะแนน)

เลือก 5 Factors และอธิบายว่าเห็นจากไหนใน Railway:

| Factor | เห็นจากไหน? | ทำไมสำคัญ? |
|--------|------------|-----------|
| Factor 3: Config | Variables tab ใน Railway Dashboard | แยก config ออกจาก code ทำให้ไม่ต้อง hardcode ค่าใน source code และปลอดภัยกว่าการใส่ใน .env ที่อาจ commit หลุด |
| Factor 1: Codebase | Deploy จาก GitHub repository เดียว | ทุก environment ใช้ codebase เดียวกัน ลดความสับสนและ bug ที่เกิดจาก code ต่างกัน |
| Factor 4: Backing Services | PostgreSQL เป็น Attached Resource แยกจาก App | สามารถเปลี่ยน Database หรือย้ายไป instance อื่นได้โดยแค่เปลี่ยน DATABASE_URL ไม่ต้องแก้ code |
| Factor 11: Logs | stdout ส่งตรงไปที่ Logs tab บน Dashboard | แอปไม่ต้องจัดการไฟล์ log เอง Railway รวบรวมและแสดงให้อัตโนมัติ |
| Factor 5: Build, Release, Run | ประวัติ Deployments แสดงแต่ละ release แยกกัน | แต่ละ deploy เป็น release ที่ immutable สามารถ rollback ได้ถ้ามีปัญหา |

### 4.2 ถ้าไม่ทำตาม 12-Factor จะมีปัญหาอะไร? (5 คะแนน)

ยกตัวอย่าง 2 ปัญหา:

**ปัญหา 1:** ถ้าไม่ทำตาม Factor 3 (Config)
- สิ่งที่จะเกิด: ถ้า hardcode DATABASE_URL ไว้ใน code และ push ขึ้น GitHub จะทำให้ credentials หลุดสู่สาธารณะ และทุกครั้งที่เปลี่ยน environment ต้องแก้ code แล้ว deploy ใหม่ทั้งหมด

**ปัญหา 2:** ถ้าไม่ทำตาม Factor 11 (Logs)
- สิ่งที่จะเกิด: ถ้าแอปเขียน log ลงไฟล์ใน container เมื่อ container ถูก restart ข้อมูล log จะหายหมด ทำให้ debug ปัญหาที่เกิดขึ้นก่อน restart ไม่ได้เลย

---

## ส่วนที่ 5: Reflection (10 คะแนน)

### 5.1 สิ่งที่เรียนรู้จาก Lab นี้

1. เข้าใจความแตกต่างระหว่าง Docker บนเครื่อง local กับการ deploy บน Cloud PaaS จริงๆ ว่า Cloud จัดการ Infrastructure ส่วนใหญ่ให้เรา
2. เห็นว่า Environment Variables มีความสำคัญมากในการแยก config ออกจาก code โดยเฉพาะการใช้ Reference (`${{Postgres.DATABASE_URL}}`) ที่ Railway เชื่อมระหว่าง services ได้อัตโนมัติ
3. เข้าใจ 12-Factor App ในเชิงปฏิบัติจริง ไม่ใช่แค่ทฤษฎี เพราะ Railway ถูกออกแบบมาให้รองรับทุก Factor โดยธรรมชาติ

### 5.2 ความท้าทาย/ปัญหาที่พบ และวิธีแก้ไข

ปัญหา: _______________________________________________  
วิธีแก้: _______________________________________________

### 5.3 จะเลือกใช้ Docker หรือ Cloud เมื่อไหร่?

- ใช้ Docker เมื่อ: พัฒนาและทดสอบบนเครื่อง local ต้องการ environment ที่เหมือนกันในทีม หรือเมื่อต้องการควบคุม infrastructure เต็มรูปแบบและไม่ต้องการค่าใช้จ่ายในช่วง dev
- ใช้ Cloud (PaaS) เมื่อ: ต้องการ deploy ให้ผู้ใช้จริงเข้าถึงได้จากทุกที่ ต้องการ uptime 24/7 หรือเมื่อทีมไม่มีผู้เชี่ยวชาญด้าน Infrastructure โดยเฉพาะ
