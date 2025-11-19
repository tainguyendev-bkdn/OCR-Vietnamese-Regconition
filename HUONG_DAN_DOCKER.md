# 🐳 Hướng Dẫn Chạy Dự Án Bằng Docker

Hướng dẫn chi tiết cách chạy dự án OCR Tiếng Việt bằng Docker.

## 📋 Yêu Cầu

- Docker đã được cài đặt
- Docker Compose (tùy chọn, cho cách 2)

---

## 🚀 Cách 1: Training Model với Docker

### Bước 1: Build Docker Image

```bash
docker build -t ocr_train .
```

Lệnh này sẽ:
- Tải Python 3.11.2
- Cài đặt các thư viện từ `requirements.txt`
- Cài đặt OpenCV dependencies (ffmpeg, libsm6, libxext6)
- Copy toàn bộ source code vào container

### Bước 2: Chạy Container để Training

```bash
docker run --name ocr_train ocr_train
```

**Lưu ý:**
- Container sẽ tự động chạy training khi khởi động
- Quá trình training có thể mất nhiều thời gian (vài giờ đến vài ngày tùy vào GPU/CPU)
- Dataset sẽ được tải tự động trong container

### Bước 3: Xem Logs

Để xem logs trong quá trình training:

```bash
docker logs -f ocr_train
```

### Bước 4: Lưu Kết Quả

Sau khi training xong, copy các file kết quả ra ngoài:

```bash
# Copy checkpoint
docker cp ocr_train:/app/checkpoints/checkpoint.weights.h5 ./checkpoints/

# Copy model đầy đủ
docker cp ocr_train:/app/model/ocr_model.h5 ./model/

# Copy prediction (nếu có)
docker cp ocr_train:/app/prediction.txt ./
```

---

## 🎯 Cách 2: Prediction với Docker

### Bước 1: Build Image (nếu chưa build)

```bash
docker build -t ocr_train .
```

### Bước 2: Chạy Container ở chế độ Interactive

```bash
docker run -it --name ocr_predict ocr_train bash
```

### Bước 3: Chạy Prediction trong Container

Sau khi vào container, chạy:

```bash
python ./src/predict.py
```

Kết quả sẽ được lưu tại `/app/prediction.txt`

### Bước 4: Copy Kết Quả Ra Ngoài

Mở terminal mới (không thoát container) và chạy:

```bash
docker cp ocr_predict:/app/prediction.txt ./
```

Hoặc nếu container đã dừng:

```bash
docker start ocr_predict
docker exec -it ocr_predict bash
# Chạy prediction
python ./src/predict.py
# Thoát và copy
docker cp ocr_predict:/app/prediction.txt ./
```

---

## 🌐 Cách 3: Web Demo với Docker

### Tạo Dockerfile cho Web Demo

Tạo file `Dockerfile.demo`:

```dockerfile
FROM python:3.11.2-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# Cài đặt dependencies
RUN apt-get update && apt-get install -y \
    ffmpeg \
    libsm6 \
    libxext6 \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements và cài đặt
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy source code
COPY . .

# Expose port cho Streamlit
EXPOSE 8501

# Chạy Streamlit
CMD ["streamlit", "run", "run.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

### Build và Chạy Web Demo

```bash
# Build image
docker build -f Dockerfile.demo -t ocr_demo .

# Chạy container
docker run -p 8501:8501 --name ocr_demo ocr_demo
```

Sau đó mở trình duyệt: `http://localhost:8501/`

**Lưu ý:** Bạn cần có file `./checkpoints/checkpoint.weights.h5` trong container. Có thể:
- Copy vào trước khi build: `COPY checkpoints/ ./checkpoints/`
- Hoặc mount volume: `docker run -p 8501:8501 -v ./checkpoints:/app/checkpoints ocr_demo`

---

## 🔧 Cách 4: Sử dụng Docker Compose

### Cập nhật compose.yaml

Tạo file `docker-compose.yml` (hoặc cập nhật `compose.yaml`):

```yaml
version: '3.8'

services:
  training:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ocr_training
    volumes:
      - ./checkpoints:/app/checkpoints
      - ./model:/app/model
      - ./Handwritten OCR:/app/Handwritten OCR
    command: python ./src/train.py

  prediction:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ocr_prediction
    volumes:
      - ./checkpoints:/app/checkpoints
      - ./model:/app/model
      - ./Handwritten OCR:/app/Handwritten OCR
      - ./prediction.txt:/app/prediction.txt
    command: python ./src/predict.py
    depends_on:
      - training

  demo:
    build:
      context: .
      dockerfile: Dockerfile.demo
    container_name: ocr_demo
    ports:
      - "8501:8501"
    volumes:
      - ./checkpoints:/app/checkpoints
    command: streamlit run run.py --server.port=8501 --server.address=0.0.0.0
```

### Chạy với Docker Compose

```bash
# Training
docker-compose up training

# Prediction
docker-compose up prediction

# Web Demo
docker-compose up demo
```

---

## 📦 Mount Volumes để Lưu Dữ Liệu

Để lưu dữ liệu ra ngoài container, sử dụng volumes:

```bash
docker run -v $(pwd)/checkpoints:/app/checkpoints \
           -v $(pwd)/model:/app/model \
           -v $(pwd)/Handwritten\ OCR:/app/Handwritten\ OCR \
           --name ocr_train ocr_train
```

**Windows (PowerShell):**
```powershell
docker run -v ${PWD}/checkpoints:/app/checkpoints `
           -v ${PWD}/model:/app/model `
           -v ${PWD}/Handwritten` OCR:/app/Handwritten` OCR `
           --name ocr_train ocr_train
```

**Windows (CMD):**
```cmd
docker run -v %CD%/checkpoints:/app/checkpoints -v %CD%/model:/app/model --name ocr_train ocr_train
```

---

## 🛠️ Các Lệnh Docker Hữu Ích

### Xem danh sách containers

```bash
docker ps -a
```

### Dừng container

```bash
docker stop ocr_train
```

### Xóa container

```bash
docker rm ocr_train
```

### Xóa image

```bash
docker rmi ocr_train
```

### Vào container đang chạy

```bash
docker exec -it ocr_train bash
```

### Xem logs

```bash
docker logs ocr_train
docker logs -f ocr_train  # Theo dõi real-time
```

### Copy file từ container ra ngoài

```bash
docker cp ocr_train:/app/prediction.txt ./
```

### Copy file từ ngoài vào container

```bash
docker cp ./checkpoint.weights.h5 ocr_train:/app/checkpoints/
```

---

## ⚠️ Lưu Ý Quan Trọng

1. **Dataset lớn**: Dataset sẽ được tải trong container, có thể tốn nhiều dung lượng
2. **Training lâu**: Training có thể mất nhiều giờ, đảm bảo container không bị dừng
3. **GPU Support**: Để sử dụng GPU, cần cài Docker với GPU support và thêm flag `--gpus all`
4. **Memory**: Đảm bảo Docker có đủ RAM (khuyến nghị ít nhất 8GB)
5. **Persistent Data**: Sử dụng volumes để lưu checkpoint và model, tránh mất dữ liệu khi xóa container

---

## 🚀 Chạy với GPU (NVIDIA)

Nếu bạn có GPU NVIDIA:

```bash
# Cài đặt nvidia-docker2 trước
docker run --gpus all --name ocr_train ocr_train
```

---

## 📝 Tóm Tắt Nhanh

```bash
# 1. Training
docker build -t ocr_train .
docker run --name ocr_train -v $(pwd)/checkpoints:/app/checkpoints ocr_train

# 2. Prediction
docker run -it --name ocr_predict ocr_train bash
# Trong container: python ./src/predict.py

# 3. Web Demo
docker build -f Dockerfile.demo -t ocr_demo .
docker run -p 8501:8501 -v $(pwd)/checkpoints:/app/checkpoints ocr_demo
```

---

## ❓ Xử Lý Lỗi

### Lỗi: Out of memory
- Tăng memory limit cho Docker
- Giảm BATCH_SIZE trong configs.py

### Lỗi: Permission denied
- Thêm `--user root` vào docker run
- Hoặc fix permissions trong Dockerfile

### Lỗi: Port đã được sử dụng
- Đổi port: `-p 8502:8501`
- Hoặc dừng service đang dùng port đó

