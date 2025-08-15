Bài tập lớn môn:
Phân tích và dự báo dữ liệu học sâu

- Sinh viên thực hiện:

  - Mai Xuân Duy
  - Nguyễn Duy Tân

- Các bước thực hiện

  - Build môi trường trên docker, mở terminal và chạy lệnh
    . docker-compose build
    . docker-compose up -d
  - Sau khi đã khởi tạo môi trường xong
    . Khởi tạo topic để truyền dữ liệu, mở terminal và chạy lệnh:
    docker exec -it kafka bash
    kafka-topics.sh --create --topic BTLCDDDHeart --bootstrap-server localhost:9092

    . Kiểm tra các topic đã tạo:
    docker exec -it kafka bash
    kafka-topics.sh --list --bootstrap-server localhost:9092

    . Kiểm tra kafka đã truyền nhận dữ liệu thành công chưa, mở 2 terminal và chạy các lệnh:
    terminal 1:
    docker exec -it kafka bash
    kafka-console-consumer.sh --topic BTLCDDDHeart --from-beginning --bootstrap-server localhost:9092
    terminal 2:
    docker exec -it kafka bash
    kafka-console-producer.sh --topic BTLCDDDHeart --bootstrap-server localhost:9092

    . Bật module giả lập truyền dữ liệu
    docker exec -it python-app bash
    python3 sendStream.py

    . Bật module nhận dữ liệu, thực hiện chuẩn đoán, re-train models AI
    docker exec -it python-app bash
    python3 revStream.py

```mermaid
  flowchart TD
  A[CSV Data<br>heart_disease_uci.csv] --> B[Data Lake<br>CSV/Parquet]
  H[Real-time Data<br>App/Web/IoT] --> I[Kafka Topic<br>BTLCDDDHeart]

  subgraph Batch_Pipeline
  B --> C[EDA & Quality Check<br>- Missing Values<br>- Outliers<br>- Drift]
  C --> D[Feature Engineering<br>- Encode: sex, cp, restecg<br>- Scale: age, trestbps, chol<br>- Save: encoder.pkl, scaler.pkl]
  D --> E[Train Model<br>- Random Forest<br>- CV: Acc, F1, AUC]
  E --> F[Model Registry<br>- Save model.pkl<br>- MLflow Versioning]
  F --> G[Batch Prediction<br>- Score Historical Data<br>- Export Reports]
  end

  subgraph Real_time_Pipeline
  I --> J[Spark Streaming<br>- Read Kafka]
  J --> K[Load Artifacts<br>- encoder.pkl, scaler.pkl, model.pkl]
  K --> L[Real-time Prediction<br>- Random Forest<br>- Risk: Low/Med/High]
  L --> M[Sinks<br>- DB, Dashboard, API<br>- Alerts]
  end

  N[New Labeled Data] --> B
  M --> O[Model Monitoring<br>- Performance<br>- Drift Detection]
  O -->|Retraining Trigger| C
```