# Data Visualization.

การสร้าง Dashboard สำหรับ Visualization โดยใช้ Grafana และ FlowCharting คือการนำข้อมูลจากเซ็นเซอร์มาวิเคราะห์และแสดงผลในรูปแบบแผนผังสถานที่ผ่าน FlowCharting ซึ่งเป็น Plugin ของ Grafana ที่ช่วยให้ผู้ใช้สามารถสร้างและปรับแต่งแผนผัง (Flowchart) เพื่อแสดงสถานะหรือข้อมูลต่างๆ ให้สามารถเข้าใจง่าย สำหรับการสร้างแผนผังสถานที่ที่ต้องการ ผู้ใช้สามารถสร้างแผนผังนั้นใน Draw.io ซึ่งเป็นเครื่องมือสำหรับการออกแบบ Diagram จากนั้นนำแผนผังที่สร้างเสร็จแล้วมาใช้ใน Grafana ผ่าน FlowCharting Plugin เพื่อเชื่อมต่อกับข้อมูลเซ็นเซอร์ และ แสดงผลข้อมูลบน Dashboard ซึ่งจะช่วยให้ผู้ใช้สามารถติดตามสถานะและข้อมูลต่าง ๆ ได้อย่างมีประสิทธิภาพและเข้าใจง่าย

## Mount volumn for grafana

ในการติดตั้ง FlowCharting plugin สำหรับ Grafana ขั้นตอนแรกที่สำคัญคือการ mount volume เพื่อจัดเก็บ plugin นี้อย่างถาวร การ mount volume จะช่วยให้ข้อมูลและ plugin ที่ติดตั้งอยู่ในพื้นที่เก็บข้อมูลนี้สามารถเข้าถึงได้อย่างต่อเนื่อง แม้ในกรณีที่ระบบต้องรีสตาร์ทหรือมีการอัปเดต

grafana:
  image: grafana/grafana:latest-ubuntu
  container_name: grafana
  user: "0"
  volumes:
    - ./grafana/data:/var/lib/grafana 
    - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
    - ./grafana/datasources:/etc/grafana/provisioning/datasources
  environment:
    - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
    - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
    - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-worldmap-panel,grafana-piechart-panel
    - GF_USERS_ALLOW_SIGN_UP=false
    - GF_SECURITY_ANGULAR_SUPPORT_eNABLED=True
    - GF_FEATURE_TOGGLES_ANGULARDEPRECATIONUI=FALSE
  restart: unless-stopped
  links:
    - prometheus
  ports:
    - "8085:3000"

## Install flowcharting on grafana

### Step 1
เข้าไปที่ https://github.com/skyfrank/grafana-flowcharting/releases/tag/v1.0.0e 

### Step 2
Download Assets
agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip 

### Step 3
เข้า Folder plugins แล้วเก็บไฟล์ zip ไว้ใน Folder 
cd iot_event_streaming_architechture/grafana/data/plugins/src

### Step 4
extract unzip ไฟล์ zip ที่ได้จากขั้นตอนที่ 2
unzip agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip -d ../grafana-flowcharting

### Step 5
เมื่อ extract ไฟล์เรียบร้อยแล้ว จะต้อง restart grafana ใหม่
docker compose restart grafana

### Step 6 
เข้าไปที่ grafana เพื่อดูว่ามี plugin ชื่อ FlowCharting ติดตั้งแล้วหรือยังเพื่อตรวจสอบว่าติดตั้งเรียบร้อยแล้ว


<!-- 
docker exec -it grafana /bin/bash
cd /etc/grafana/
apt-get update
apt-get install vim
vim grafana.ini
แก้ไข vim 
cd var/lib/grafana/plugins
grafana-cli plugins install agenty-flowcharting-panel -->


## Using flowcharting on grafana

ในตัวอย่างนี้จะบอกอุณหภูมิที่ sensor ทั้งหมด 10 ตัวตรวจจับได้จากโซนต่างๆของบ้าน 1 ชั้นเพื่อจำลองเป็น smart home

### Step 1
กด Add ในหน้า grafana dashboard โดยเลือกเป็น flowcharting เพื่อสร้าง widget แปลนบ้าน 

### Step 2
กด edit เพื่อเข้าไปแก้ไข draw.io เพื่อเอารูปแปลนบ้าน 1 ชั้นมาใส่