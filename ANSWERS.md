# BÁO CÁO GIẢI TRÌNH KỸ THUẬT (ANSWERS.MD)
## Lab 28 Track 2 — Modern AI Platform Integration & Production Readiness
**Người thực hiện / Nhóm:** Nguyễn Hoàng Việt (Mã số: 2A202601940)  
**Nhánh Git:** `ca-nhan-nguyenhoangviet`  
**Ngày hoàn thành:** 2026-09-04  

---

## 1. PHÂN TÍCH CÁC ĐÁNH ĐỔI KỸ THUẬT (TRADE-OFFS)

### 1.1. Ingestion qua Kafka Buffer vs Ghi trực tiếp vào Storage
- **Lựa chọn:** Toàn bộ dữ liệu phản hồi (`feedback`) và tài liệu (`documents`) từ người dùng được gửi qua HTTP API, kiểm tra schema và đẩy vào topic `data.raw` của Kafka với message key là `idempotency_key`, thay vì ghi trực tiếp vào Delta Lake hoặc database.
- **Đánh đổi:**
  - *Ưu điểm:* Giải phóng luồng HTTP tức thì (độ trễ phản hồi < 15ms), API đạt chuẩn 202 Accepted. Kafka đóng vai trò là bộ đệm (shock absorber) khi lưu lượng tăng đột biến, hỗ trợ replay bản tin khi các dịch vụ hạ tầng phía sau gặp sự cố mà không làm mất dữ liệu của client.
  - *Nhược điểm:* Tăng thêm độ phức tạp trong kiến trúc (cần vận hành Kafka cluster, Zookeeper/KRaft, quản lý offset, retention và consumer lag). Dữ liệu không đạt trạng thái real-time tức thì trong lakehouse mà có độ trễ micro-batching (eventual consistency).

### 1.2. Idempotency qua Delta MERGE & Deterministic UUIDs
- **Lựa chọn:**
  - Tại tầng Delta Lake (IP03): Sử dụng hàm `dedupe_latest` ở client để lọc trùng các bản tin cùng `idempotency_key` trong một batch (ưu tiên bản ghi có `occurred_at` mới nhất), kết hợp lệnh Spark SQL `MERGE INTO ... ON target.idempotency_key = source.idempotency_key WHEN MATCHED THEN UPDATE SET * WHEN NOT MATCHED THEN INSERT *`.
  - Tại tầng Qdrant (IP05): Sử dụng hàm `uuid5(NAMESPACE, doc_id)` để sinh UUID tất định cho mỗi điểm vector.
- **Đánh đổi:**
  - *Ưu điểm:* Đảm bảo tính chống trùng lặp tuyệt đối (idempotency). Khi Kafka replay một lô dữ liệu cũ 2 hoặc nhiều lần (đã kiểm chứng qua bài test `IT-J2-idempotent-replay`), số dòng trong Delta Lake và số điểm vector trong Qdrant không bao giờ bị nhân bản.
  - *Nhược điểm:* Lệnh `MERGE INTO` trong Spark tốn nhiều tài nguyên CPU/I-O hơn so với lệnh `INSERT INTO` thông thường (append-only) do phải scan index và so sánh key. Việc sinh UUID tất định đòi hỏi kiểm soát chặt chẽ namespace và doc_id.

### 1.3. Phân tách Offline Snapshot (Delta) và Online Serving (Feast)
- **Lựa chọn:** Tính toán các thống kê người dùng (`feedback_count`, `avg_rating`, `negative_ratio`) theo chu kỳ trên Delta Lake, kết xuất thành offline snapshot và nạp vào Feast Online Store (IP04).
- **Đánh đổi:**
  - *Ưu điểm:* Đường dẫn phục vụ câu hỏi (`/api/v1/ask`) chỉ cần truy vấn Feast Online Store với thời gian phản hồi cực nhanh (< 5ms), không bao giờ phải chạy truy vấn aggregate nặng nề trên Data Lake tại thời điểm user gửi prompt.
  - *Nhược điểm:* Dữ liệu đặc trưng có độ trễ (freshness delay) phụ thuộc vào chu kỳ chạy Airflow DAG / Feast materialize.

### 1.4. Quản lý Release qua MLflow Champion Alias
- **Lựa chọn:** Cấu hình RAG (Prompt template, model ID, top_k, delta_version liên kết) được đóng gói thành một model artifact và đăng ký trên MLflow Registry (IP06), trỏ bằng alias `champion`.
- **Đánh đổi:**
  - *Ưu điểm:* Cho phép Zero-downtime Rollback tức thì. Khi một phiên bản prompt v2 gặp vấn đề, kỹ sư chỉ cần gọi `lab28 rollback` để trỏ alias `champion` về v1 mà không cần build lại Docker image, không sửa code và không khởi động lại API Pod.
  - *Nhược điểm:* Cần duy trì dịch vụ MLflow Server và cơ sở dữ liệu tracking backend SQLite/PostgreSQL.

### 1.5. Tri-state Readiness Semantics (`ready` vs `degraded` vs `not_ready`)
- **Lựa chọn:** Phân loại rõ ràng mức độ phụ thuộc: Kafka, MLflow, Qdrant là bắt buộc (`mandatory=True`); Feast và vLLM (ở môi trường local không GPU) là tùy chọn (`mandatory=False`).
- **Đánh đổi:**
  - *Ưu điểm:* Tăng khả năng chịu lỗi (resilience). Khi Feature Store gặp sự cố, API chuyển sang trạng thái `degraded` thay vì trả về 503, giúp người dùng vẫn nhận được câu trả lời RAG cơ bản.
  - *Nhược điểm:* Đòi hỏi logic fallback trong mã nguồn để xử lý các trường hợp dữ liệu đặc trưng bị khuyết thiếu.

---

## 2. KHOẢNG CÁCH VỚI MÔI TRƯỜNG SẢN XUẤT (PRODUCTION GAPS)

Mặc dù hệ thống trong bài lab đã mô phỏng đầy đủ 10 ranh giới kỹ thuật, nhưng để triển khai ra môi trường Production quy mô lớn cấp doanh nghiệp, cần thu hẹp các khoảng cách kỹ thuật sau:

1. **Bảo mật và Quản lý Danh tính (Security & Secrets Management):**
   - *Hiện tại:* Biến môi trường được nạp tĩnh qua `ports.template` và Docker environment variables. Kafka và Feast chạy chế độ PLAINTEXT không mã hóa.
   - *Production Cần:* Tích hợp HashiCorp Vault, AWS Secrets Manager hoặc Kubernetes External Secrets Operator; bật giao thức SASL/SCRAM và mTLS cho Kafka; cấu hình OAuth2/OIDC JWT validation tại Envoy Gateway.
2. **Lưu trữ Lakehouse Phân tán (Distributed Object Storage & Catalog):**
   - *Hiện tại:* Delta Lake lưu trữ trên local file system (`.lab28/delta`) được mount qua volume.
   - *Production Cần:* Đưa Delta Lake lên AWS S3, Google Cloud Storage hoặc MinIO với định dạng Parquet nén Snappy/ZSTD; kết hợp với Unity Catalog hoặc Apache Polaris / AWS Glue để quản lý quyền truy cập cấp dòng/cột (fine-grained access control).
3. **Cụm Tính toán Phân tán (Distributed Spark & Kubernetes Engine):**
   - *Hiện tại:* Spark Connect chạy dạng single-container master/worker.
   - *Production Cần:* Triển khai Spark on Kubernetes (Spark Operator) hoặc Databricks/EMR tự động co giãn (autoscaling) theo tải lượng dữ liệu của từng pipeline run.
4. **Hạ tầng Inference LLM Quy mô lớn (vLLM High-Availability Cluster):**
   - *Hiện tại:* vLLM chạy trên 1 GPU cục bộ hoặc mượn qua tunnel Kaggle T4.
   - *Production Cần:* Cụm vLLM đa node với Tensor Parallelism và Pipeline Parallelism; đặt sau bộ cân bằng tải suy luận (ví dụ: vLLM Router / Text Generation Inference / Triton) có khả năng dynamic batching, prefix caching và preemptive priority queue.
5. **Giám sát & Đánh giá Chất lượng RAG Liên tục (LLM Observability & Guardrails):**
   - *Hiện tại:* Mới chỉ đo đạc latency, golden signals và distributed tracing qua Jaeger.
   - *Production Cần:* Tích hợp LangSmith / TruLens / Phoenix Arize để chấm điểm tự động chất lượng câu trả lời (Groundedness, Context Relevance, Answer Faithfulness) và phát hiện ảo giác (hallucination detection) theo thời gian thực.

---

## 3. BẢNG PHÂN CÔNG VÀ ĐÓNG GÓP THỰC TẾ

| Thành viên / Vai trò | Trách nhiệm chính | Kết quả & Bằng chứng nghiệm thu | Tỷ lệ đóng góp |
|---|---|---|:---:|
| **Nguyễn Hoàng Việt**<br>*(Đảm nhiệm toàn bộ 5 vai trò)* | **1. Ingestion & Orchestration (IP01, IP02):**<br>- Hoàn thiện hàm `event_headers` (W3C traceparent & idempotency-key bytes).<br>- Vận hành Kafka broker, topics và Airflow 3 DAG.<br><br>**2. Data & ML Platform (IP03, IP04, IP06):**<br>- Hoàn thiện hàm `dedupe_latest` (chống trùng lặp Delta Lake).<br>- Hoàn thiện hàm `feast_online_request` (truy vấn Feast feature).<br>- Thực hiện MLflow release versioning & alias champion rollback.<br><br>**3. Serving & Retrieval (IP05, IP07):**<br>- Đánh chỉ mục Qdrant với UUID tất định.<br>- Cấu hình degraded mode fallback khi thiếu LLM GPU.<br><br>**4. Platform & Observability (IP08, IP09, IP10):**<br>- Cấu hình Envoy Gateway rate-limiting & trace propagation.<br>- Thu thập đầy đủ 10 file evidence đạt chuẩn.<br>- Vượt qua toàn bộ 83 unit tests và 56 integration tests.<br><br>**5. Incident Commander & Presenter:**<br>- Soạn thảo tài liệu hướng dẫn và kịch bản demo. | - `src/lab28_platform/integration_tasks.py`: hoàn thiện 4/4 hàm chuẩn xác.<br>- `pyproject.toml`: chuẩn hóa `--basetemp` cho Windows.<br>- Đủ 10 file JSON trong `evidence/`.<br>- 83/83 unit tests PASS (100%).<br>- 56/56 integration tests non-GPU PASS (100%).<br>- Bộ manifest K8s/GitOps hợp lệ 100%. | **100%** |

---

## 4. TỔNG HỢP KẾT QUẢ KIỂM THỬ THỰC TẾ (EVIDENCE SUMMARY)

- **Fast Unit Tests:** `83 passed in 3.32s`
- **Starter Tasks:** `4 passed in 0.10s`
- **Code Linter (Ruff):** `All checks passed!`
- **Matrix Verifier:** `OK 245 checks passed`
- **Portability Check:** `OK supported workflow is host-path and shell independent`
- **Kubernetes / GitOps Validation:** `Kubernetes and GitOps manifest contracts passed`
- **Integration Test J1 (Golden Path):** `12 passed, 3 skipped (gpu)`
- **Integration Test J2 (Idempotent Replay):** `9 passed in 58.95s`
- **Full Integration Suite:** `56 passed, 16 deselected in 212.36s (0:03:32)`
- **Load Test Profile (API 200 requests, 8 workers):**
  - Trạng thái phản hồi: 200/200 thành công (100%)
  - Độ trễ P50: 483.28 ms
  - Độ trễ P95: 1,147.82 ms
  - Độ trễ P99: 1,583.99 ms
- **Gateway Rate Limiting Proof:** Envoy Gateway chặn vượt ngưỡng sau 10 req/s, trả về HTTP 429 (`local_rate_limited`), bảo vệ backend thành công.
- **Delta Lake Commit History:** 17 rows documents, 19 rows feedback, commit log tăng phiên bản chính xác mà không bị nhân bản dữ liệu khi replay.
