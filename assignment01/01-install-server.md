# Install Server and Docker


## How to install Server

1. เริ่มต้นเลือกใช้งาน OS (โดยได้หาไฟล์ img มาเป็น Ubuntu server เวอร์ชัน 24.04 LTS) 

2. ใช้โปรแกรม Refus ในการเตรียม แฟลชไดรฟ์ ให้พร้อมใช้งานเป็น (bootable media) จากไฟล์อิมเมจISOที่เลือกมา

3. ทำการลบ OS เดิม ใน boot option และติดตั้ง อันใหม่เข้าไป

## How to install Docker

1. เริ่มต้นโดยทำการเช็ค Packages และเป็นการเช็ค internet ไปในตัว
โดยการ sudo apt update

2. ทำการติดตั้ง Docker โดยใช้ command ดังนี้
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh

3. ทำการดริ่มต้นและ enable Docker 
    sudo systemctl start docker
    sudo systemctl enable docker

4. ทำการเช็ค version เพื่อมั่นใจว่าสำเร็จแล้ว
    sudo docker --version

5. จัดการให้เป็น Non-root user (ทำหรือไม่ทำก็ได้)
    sudo usermod -aG docker your_username


