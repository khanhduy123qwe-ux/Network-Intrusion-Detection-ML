# Network Intrusion Detection System using Machine Learning

Dự án xây dựng hệ thống phát hiện xâm nhập mạng (NIDS) bằng Machine Learning,
sử dụng bộ dữ liệu CIC-IDS2017. Mục tiêu chính là tiền xử lý dữ liệu network
flow, huấn luyện một số mô hình phân loại và triển khai mô hình tốt nhất để
phát hiện lưu lượng bất thường.

## Mục lục

- [Giới thiệu](#giới-thiệu)
- [Kiến trúc xử lý](#kiến-trúc-xử-lý)
- [Dataset](#dataset)
- [Kết quả mô hình](#kết-quả-mô-hình)
- [Cài đặt](#cài-đặt)
- [Cách sử dụng](#cách-sử-dụng)
- [Cấu trúc thư mục](#cấu-trúc-thư-mục)
- [Thành viên](#thành-viên)
- [Tài liệu tham khảo](#tài-liệu-tham-khảo)

## Giới thiệu

Hệ thống tập trung vào các bước chính sau:

- Gộp và làm sạch dữ liệu CIC-IDS2017.
- Chọn 18 đặc trưng quan trọng từ dữ liệu network flow.
- Xử lý mất cân bằng lớp bằng SMOTE và RandomUnderSampler.
- Huấn luyện và so sánh 5 mô hình: Logistic Regression, SVM, Naive Bayes, KNN và Random Forest.
- Dùng mô hình tốt nhất để phát hiện traffic bất thường và ghi cảnh báo.

Các nhóm traffic được xét trong dự án:

| Loại traffic | Mô tả                                                     |
| ---          | ---                                                       |   
| DoS / DDoS   | Tấn công từ chối dịch vụ, làm quá tải hệ thống mục tiêu   |
| PortScan     | Quét cổng để tìm dịch vụ đang mở                          |
| Web Attack   | Các kiểu tấn công web như SQL Injection, XSS, Brute Force |
| Infiltration | Truy cập trái phép vào hệ thống nội bộ                    |
| Botnet       | Lưu lượng từ máy bị điều khiển trong botnet               |
| BENIGN       | Lưu lượng mạng bình thường                                |

## Kiến trúc xử lý

```text
Đầu vào: dữ liệu lưu lượng mạng
78 đặc trưng của network flow
        |
        v
Tiền xử lý dữ liệu
- Gộp 8 file CSV của bộ CIC-IDS2017
- Làm sạch giá trị NaN, Inf và dữ liệu không hợp lệ
- Khảo sát dữ liệu và chọn đặc trưng
- Giữ lại 18 đặc trưng dùng cho mô hình
        |
        v
Huấn luyện mô hình
- Cân bằng dữ liệu bằng SMOTE và RandomUnderSampler
- Huấn luyện 5 mô hình phân loại
- So sánh Accuracy, Precision, Recall và F1-score
        |
        v
Triển khai phát hiện
- Load mô hình Random Forest đã huấn luyện
- Dự đoán nhãn cho từng network flow
- Ghi cảnh báo vào alerts.log khi phát hiện traffic bất thường
```

## Dataset

Dự án sử dụng bộ dữ liệu CIC-IDS2017 của Canadian Institute for Cybersecurity.

| Thông tin            | Chi tiết                                    |
| ---                  | ---                                         |
| Nguồn                | [Kaggle - CIC-IDS2017](https://www.kaggle.com/datasets/chethuhn/network-intrusion-dataset)                                           |
| Số file CSV          | 8 file                                      |
| Tổng số mẫu          | Khoảng 2.8 triệu dòng                       |
| Số đặc trưng gốc     | 78 features + 1 label                       |
| Số đặc trưng sử dụng | 18 features                                 |
| Loại traffic         | 1 lớp BENIGN và 14 lớp tấn công             |
| Vấn đề chính         | Dữ liệu mất cân bằng, BENIGN chiếm phần lớn |

18 đặc trưng được dùng để huấn luyện:

```python
selected_features = [
    "Destination Port", "Flow Duration", "Total Fwd Packets",
    "Total Backward Packets", "Total Length of Fwd Packets",
    "Total Length of Bwd Packets", "Fwd Packet Length Mean",
    "Bwd Packet Length Mean", "Flow Bytes/s", "Flow Packets/s",
    "Packet Length Mean", "Packet Length Std", "SYN Flag Count",
    "ACK Flag Count", "FIN Flag Count", "RST Flag Count",
    "PSH Flag Count", "URG Flag Count"
]
```


## Cài đặt

Yêu cầu:

- Python 3.8 trở lên.
- RAM tối thiểu 8GB, khuyến nghị 16GB nếu xử lý toàn bộ dataset.
- Khoảng 5GB dung lượng trống cho dữ liệu.

Các bước cài đặt:

```bash
git clone https://github.com/khanhduy123qwe-ux/Network-Intrusion-Detection-ML.git
cd Network-Intrusion-Detection-ML

python -m venv venv
source venv/bin/activate      # Linux/macOS
venv\Scripts\activate         # Windows

pip install -r requirements.txt
```

Tải dataset từ Kaggle:

```text
https://www.kaggle.com/datasets/chethuhn/network-intrusion-dataset
```

Sau khi tải, giải nén các file CSV vào thư mục:

```text
data/raw/
```

## Cách sử dụng

Chạy các notebook theo thứ tự:

```bash
jupyter notebook notebooks/B_EDA_Preprocessing.ipynb
jupyter notebook notebooks/C_Model_Training.ipynb
jupyter notebook notebooks/D_Realtime_Alert.ipynb
```

Ví dụ dùng mô hình đã train:

```python
import joblib
import pandas as pd

model = joblib.load("models/random_forest_ids.pkl")
scaler = joblib.load("models/scaler.pkl")
label_encoder = joblib.load("models/label_encoder.pkl")


def detect(flow_features: dict):
    df = pd.DataFrame([flow_features])
    df_scaled = scaler.transform(df)

    prediction = model.predict(df_scaled)
    label = label_encoder.inverse_transform(prediction)[0]

    if label != "BENIGN":
        print(f"[ALERT] Suspicious traffic detected: {label}. Destination Port: 80.")

    return label
```

## Cấu trúc thư mục

```text
Network-Intrusion-Detection-ML/
|-- data/
|   |-- raw/
|   |   `-- 8 CSV files from CIC-IDS2017
|   `-- processed/
|       |-- cleaned_dataset.parquet
|       `-- dataset_18features.csv
|
|-- notebooks/
|   |-- B_EDA_Preprocessing.ipynb
|   |-- C_Model_Training.ipynb
|   `-- D_Realtime_Alert.ipynb
|
|-- models/
|   |-- random_forest_ids.pkl
|   |-- scaler.pkl
|   `-- label_encoder.pkl
|
|-- reports/
|   |-- eda_label_distribution.png
|   |-- eda_correlation_heatmap.png
|   |-- eda_feature_boxplot.png
|   `-- alerts.log
|
|-- requirements.txt
`-- README.md
```

## Thành viên

| Thành viên   | Vai trò          | Phụ trách                                          |
| ---          | ---              | ---                                                |
| Thành viên B | Quản lý dữ liệu  | Git setup, EDA, Preprocessing                      |
| Thành viên C | Kỹ sư ML         | Class balancing, feature selection, model training |
| Thành viên D | Kỹ sư triển khai | Real-time alert và deliverables                    |

## Tài liệu tham khảo

- [CIC-IDS2017 Dataset](https://www.unb.ca/cic/datasets/ids-2017.html)
- [Kaggle - Network Intrusion Dataset](https://www.kaggle.com/datasets/chethuhn/network-intrusion-dataset)
- [imbalanced-learn Documentation](https://imbalanced-learn.org)
- [scikit-learn Documentation](https://scikit-learn.org)
- [Reference Implementation](https://github.com/marxgoo/Network-intrusion-detection-ml)
