# Hướng Dẫn Chạy Dự Án OCR Nhận Dạng Chữ Viết Tay Tiếng Việt

Dự án này có 3 cách chạy chính:

## 📋 Yêu Cầu Hệ Thống

- Python 3.8 trở lên
- TensorFlow
- Các thư viện khác (xem `requirements.txt`)

## 🚀 Cách 1: Chạy Web Demo (Khuyến nghị cho người mới)

Web demo cho phép bạn upload ảnh và nhận dạng chữ viết tay qua giao diện web.

### Bước 1: Cài đặt thư viện

```bash
pip install -r requirements.txt
```

### Bước 2: Tải model checkpoint (nếu chưa có)

Bạn cần có file model checkpoint để chạy web demo. Tạo thư mục `checkpoints` và đặt file `checkpoint.weights.h5` vào đó:

```bash
mkdir checkpoints
# Tải file checkpoint.weights.h5 vào thư mục checkpoints/
```

**Lưu ý:** Nếu bạn chưa có checkpoint, bạn cần train model trước (xem Cách 3).

### Bước 3: Chạy web demo

```bash
streamlit run run.py
```

Sau đó mở trình duyệt và truy cập: `http://localhost:8501/`

---

## 🎯 Cách 2: Chạy Prediction (Dự đoán trên tập test)

Dự đoán trên tập dữ liệu test công khai và lưu kết quả vào file `prediction.txt`.

### Bước 1: Cài đặt thư viện

```bash
pip install -r requirements.txt
```

### Bước 2: Tải dataset (nếu chưa có)

Dataset sẽ được tải tự động khi chạy script, hoặc bạn có thể tải thủ công từ:
- Link trong file `src/utils.py`: https://drive.google.com/drive/folders/1dlhSYYrLE0GMUOUV-GDmNcJs2_Tu4KYa?usp=drive_link

### Bước 3: Có model đã train

Bạn cần có file model đã train. Tạo thư mục `model` và đặt file `ocr_model.h5` vào đó:

```bash
mkdir model
# Đặt file ocr_model.h5 vào thư mục model/
```

**Lưu ý:** Nếu chưa có model, bạn cần train trước (xem Cách 3).

### Bước 4: Chạy prediction

```bash
python src/predict.py
```

Kết quả sẽ được lưu vào file `prediction.txt` ở thư mục gốc.

---

## 🏋️ Cách 3: Train Model (Huấn luyện model)

Huấn luyện model từ đầu trên dataset.

### Bước 1: Cài đặt thư viện

```bash
pip install -r requirements.txt
```

### Bước 2: Tải dataset

Dataset sẽ được tải tự động khi chạy script training. Hoặc bạn có thể tải thủ công từ Google Drive link trong `src/utils.py`.

### Bước 3: Chạy training

```bash
python src/train.py
```

**Lưu ý:**
- Quá trình training có thể mất nhiều thời gian (tùy thuộc vào GPU/CPU)
- Model checkpoint sẽ được lưu tại `checkpoint.weights.h5` (trong thư mục gốc khi chạy từ src/)
- Model đầy đủ sẽ được lưu tại `./model/ocr_model.h5`

---

## 🐳 Cách 4: Chạy bằng Docker

**📚 Xem hướng dẫn chi tiết tại: [HUONG_DAN_DOCKER.md](./HUONG_DAN_DOCKER.md)**

### Cách nhanh nhất (sử dụng script):

**Windows:**
```cmd
docker_run.bat
```

**Linux/Mac:**
```bash
chmod +x docker_run.sh
./docker_run.sh
```

### Hoặc chạy thủ công:

#### 1. Training với Docker

```bash
# Build image
docker build -t ocr_train .

# Chạy container để train (với volumes để lưu kết quả)
docker run --name ocr_train \
    -v ./checkpoints:/app/checkpoints \
    -v ./model:/app/model \
    ocr_train
```

#### 2. Prediction với Docker

```bash
docker run --name ocr_predict \
    -v ./checkpoints:/app/checkpoints \
    -v ./model:/app/model \
    -v ./prediction.txt:/app/prediction.txt \
    ocr_train python ./src/predict.py
```

#### 3. Web Demo với Docker

```bash
# Build image cho demo
docker build -f Dockerfile.demo -t ocr_demo .

# Chạy demo
docker run -p 8501:8501 --name ocr_demo \
    -v ./checkpoints:/app/checkpoints \
    ocr_demo
```

Sau đó mở: `http://localhost:8501/`

#### 4. Sử dụng Docker Compose

```bash
# Training
docker-compose up training

# Prediction
docker-compose up prediction

# Web Demo
docker-compose up demo
```

---

## 📁 Cấu Trúc Thư Mục Sau Khi Chạy

Sau khi chạy đầy đủ, cấu trúc thư mục sẽ như sau:

```
OCR-Vietnamese-Regconition/
├── checkpoints/
│   └── checkpoint.weights.h5    # Checkpoint cho web demo
├── model/
│   └── ocr_model.h5              # Model đầy đủ
├── Handwritten OCR/              # Dataset (tự động tải)
│   ├── training_data/
│   ├── public_test_data/
│   └── train_gt.txt
├── prediction.txt                # Kết quả prediction
├── src/
├── run.py
└── requirements.txt
```

---

## ⚠️ Lưu Ý Quan Trọng

1. **Dataset**: Dataset khá lớn (~103,000 ảnh training), việc tải có thể mất thời gian
2. **Model Checkpoint**: Để chạy web demo hoặc prediction, bạn cần có model đã train sẵn
3. **GPU**: Training sẽ nhanh hơn nhiều nếu có GPU (CUDA)
4. **Memory**: Đảm bảo có đủ RAM (khuyến nghị ít nhất 8GB)

---

## 🔧 Xử Lý Lỗi Thường Gặp

### Lỗi: Không tìm thấy checkpoint
- **Giải pháp**: Train model trước hoặc tải checkpoint từ nguồn khác

### Lỗi: Không tìm thấy dataset
- **Giải pháp**: Đảm bảo kết nối internet để tải dataset tự động, hoặc tải thủ công từ Google Drive

### Lỗi: Out of memory
- **Giải pháp**: Giảm `BATCH_SIZE` trong `src/configs.py`

---

## 📞 Hỗ Trợ

Nếu gặp vấn đề, vui lòng kiểm tra:
- File `README.md` gốc
- Logs khi chạy script
- Đảm bảo đã cài đặt đầy đủ dependencies

