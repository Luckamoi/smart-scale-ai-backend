# ใช้ Python image แบบ slim เพื่อลดขนาด container [cite: 3]
FROM python:3.10-slim

WORKDIR /app

# ติดตั้ง dependencies ก่อนเพื่อใช้ประโยชน์จาก Docker Cache [cite: 4, 6]
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# คัดลอกโค้ดทั้งหมดเข้าสู่ container
COPY . .

# เปิดพอร์ต 5000 สำหรับ Flask [cite: 5]
EXPOSE 5000

CMD ["python", "app.py"]
