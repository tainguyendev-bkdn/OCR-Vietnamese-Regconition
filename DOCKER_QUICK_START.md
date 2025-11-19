# 🐳 Docker Quick Start

Hướng dẫn nhanh để chạy dự án bằng Docker.

## ⚡ Cách Nhanh Nhất

### Windows:
```cmd
docker_run.bat
```

### Linux/Mac:
```bash
chmod +x docker_run.sh
./docker_run.sh
```

---

## 📝 Các Lệnh Cơ Bản

### 1️⃣ Training

```bash
docker build -t ocr_train .
docker run --name ocr_train -v ./checkpoints:/app/checkpoints -v ./model:/app/model ocr_train
```

### 2️⃣ Prediction

```bash
docker run --name ocr_predict \
    -v ./checkpoints:/app/checkpoints \
    -v ./model:/app/model \
    ocr_train python ./src/predict.py
```

### 3️⃣ Web Demo

```bash
docker build -f Dockerfile.demo -t ocr_demo .
docker run -p 8501:8501 --name ocr_demo -v ./checkpoints:/app/checkpoints ocr_demo
```

Mở trình duyệt: http://localhost:8501/

---

## 🎯 Docker Compose

```bash
# Training
docker-compose up training

# Prediction  
docker-compose up prediction

# Web Demo
docker-compose up demo
```

---

## 📚 Xem Thêm

- Chi tiết đầy đủ: [HUONG_DAN_DOCKER.md](./HUONG_DAN_DOCKER.md)
- Hướng dẫn tổng quan: [HUONG_DAN_CHAY.md](./HUONG_DAN_CHAY.md)

