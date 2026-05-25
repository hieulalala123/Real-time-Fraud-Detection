# MLflow

## 1. Khởi tạo Dockerfile

File docker-compose.yaml này định nghĩa một cấu hình Docker Compose để triển khai một hệ thống MLflow Tracking Server với các thành phần sau:

1. postgres_db (Cơ sở dữ liệu PostgreSQL)
Mục đích: Lưu trữ metadata của MLflow (thông tin về các thí nghiệm, mô hình, metrics, v.v.).  
Hình ảnh Docker: postgres:15-alpine.  
Cấu hình:  
Tên người dùng: mlflow_user.  
Mật khẩu: mlflow_password.  
Tên cơ sở dữ liệu: mlflow_metadata.  
Cổng: Mở cổng 5432 để giao tiếp với các dịch vụ khác.  
Volume: Lưu trữ dữ liệu PostgreSQL trên volume postgres_data.    
---
2. minio_s3 (MinIO Object Storage)
Mục đích: Thay thế AWS S3 để lưu trữ artifacts và models của MLflow.  
Hình ảnh Docker: minio/minio:RELEASE.2024-01-11T06-46-16Z.  
Cấu hình:  
Tên người dùng: minio_admin.  
Mật khẩu: minio_password.  
Cổng:  
9000: API để giao tiếp với MinIO.  
9001: Giao diện web quản lý MinIO.  
Volume: Lưu trữ dữ liệu trên volume minio_data.    
---
3. create_bucket (Tạo bucket trong MinIO)
Mục đích: Tự động tạo một bucket tên   mlflow-bucket trong MinIO khi khởi động.
Hình ảnh Docker: minio/mc (MinIO Client).
Cấu hình:
Phụ thuộc vào dịch vụ minio_s3.
Sử dụng lệnh mc để tạo bucket mlflow-bucket.
---
4. mlflow_server (MLflow Tracking Server)
Mục đích: Cung cấp MLflow Tracking Server để theo dõi các thí nghiệm.  
Hình ảnh Docker: ghcr.io/mlflow/mlflow:v2.11.3.  
Cấu hình:  
Kết nối với PostgreSQL (postgres_db) để lưu metadata.  
Kết nối với MinIO (minio_s3) để lưu artifacts/models.  
Biến môi trường:  
AWS_ACCESS_KEY_ID và AWS_SECRET_ACCESS_KEY: Thông tin xác thực MinIO.  
MLFLOW_S3_ENDPOINT_URL: URL của MinIO.
Cổng: Mở cổng 5000 để giao tiếp với MLflow.
Lệnh khởi động  
```python
mlflow server \
  --backend-store-uri postgresql://mlflow_user:mlflow_password@postgres_db:5432/mlflow_metadata \
  --default-artifact-root s3://mlflow-bucket/ \
  --host 0.0.0.0 \
  --port 5000
```
----
Volumes
postgres_data: Lưu trữ dữ liệu của PostgreSQL.
minio_data: Lưu trữ dữ liệu của MinIO.
Tóm tắt
File này triển khai một hệ thống MLflow Tracking Server hoàn chỉnh với:

    - PostgreSQL để lưu metadata.
    - MinIO để lưu artifacts/models.
    - MLflow Server để theo dõi các thí nghiệm.
    - Tự động hóa việc tạo bucket trong MinIO.