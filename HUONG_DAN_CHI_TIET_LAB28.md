# HƯỚNG DẪN TOÀN DIỆN VÀ CHI TIẾT BÀI THỰC HÀNH LAB 28 TRACK 2
## Platform Integration & Production Readiness — RAG AI Platform

---

## MỤC LỤC
1. [Ý Nghĩa Cốt Lõi Của Bài Lab 28](#1-ý-nghĩa-cốt-lõi-của-bài-lab-28)
   - [1.1. Bối cảnh bài toán: Tại sao không chỉ là "chạy một chatbot"?](#11-bối-cảnh-bài-toán-tại-sao-không-chỉ-là-chạy-một-chatbot)
   - [1.2. Mô hình kiến trúc 5 tầng nền tảng (5 Layers)](#12-mô-hình-kiến-trúc-5-tầng-nền-tảng-5-layers)
   - [1.3. Ý nghĩa chi tiết của 10 Điểm Kết Nối (10 Integration Points: IP01 – IP10)](#13-ý-nghĩa-chi-tiết-của-10-điểm-kết-nối-10-integration-points-ip01--ip10)
   - [1.4. Ý nghĩa 5 Hành trình Kiểm thử Sống (5 Critical Journeys: J1 – J5)](#14-ý-nghĩa-5-hành-trình-kiểm-thử-sống-5-critical-journeys-j1--j5)
   - [1.5. Các nguyên lý Platform Engineering bắt buộc nắm vững](#15-các-nguyên-lý-platform-engineering-bắt-buộc-nắm-vững)
2. [Cơ Cấu Đánh Giá & Phân Vai Nhóm](#2-cơ-cấu-đánh-giá--phân-vai-nhóm)
   - [2.1. Thang điểm 100 chi tiết (Rubric)](#21-thang-điểm-100-chi-tiết-rubric)
   - [2.2. Bảng phân vai nhóm (Team Role Cards)](#22-bảng-phân-vai-nhóm-team-role-cards)
   - [2.3. Yêu cầu hồ sơ nộp bài (Submission Checklist)](#23-yêu-cầu-hồ-sơ-nộp-bài-submission-checklist)
3. [Hướng Dẫn Thực Hành Từng Bước (Step-by-Step Execution)](#3-hướng-dẫn-thực-hành-từng-bước-step-by-step-execution)
   - [Bước 1: Chuẩn bị công cụ và khởi tạo nhánh làm việc](#bước-1-chuẩn-bị-công-cụ-và-khởi-tạo-nhánh-làm-việc)
   - [Bước 2: Thiết lập môi trường Python bằng uv](#bước-2-thiết-lập-môi-trường-python-bằng-uv)
   - [Bước 3: Xác nhận trạng thái ban đầu (Starter Tests)](#bước-3-xác-nhận-trạng-thái-ban-đầu-starter-tests)
   - [Bước 4: Hoàn thiện 4 chức năng cốt lõi trong mã nguồn](#bước-4-hoàn-thiện-4-chức-năng-cốt-lõi-trong-mã-nguồn)
   - [Bước 5: Kiểm tra chất lượng mã nguồn & Hợp đồng tích hợp](#bước-5-kiểm-tra-chất-lượng-mã-nguồn--hợp-đồng-tích-hợp)
   - [Bước 6: Kiểm tra tính hợp lệ của cấu hình Docker](#bước-6-kiểm-tra-tính-hợp-lệ-của-cấu-hình-docker)
   - [Bước 7: Chạy hệ thống cơ bản (Core Stack Checkpoint)](#bước-7-chạy-hệ-thống-cơ-bản-core-stack-checkpoint)
   - [Bước 8: Chạy toàn bộ luồng Data & ML (Full Stack Checkpoint)](#bước-8-chạy-toàn-bộ-luồng-data--ml-full-stack-checkpoint)
   - [Bước 9: Tích hợp vLLM thật trên GPU (Local hoặc Kaggle T4)](#bước-9-tích-hợp-vllm-thật-trên-gpu-local-hoặc-kaggle-t4)
   - [Bước 10: Diễn tập sự cố & Kiểm tra hiệu năng (Failure Injection & Load Testing)](#bước-10-diễn-tập-sự-cố--kiểm-tra-hiệu-năng-failure-injection--load-testing)
   - [Bước 11: Xác thực cấu hình Kubernetes & GitOps](#bước-11-xác-thực-cấu-hình-kubernetes--gitops)
   - [Bước 12: Thu thập bằng chứng nộp bài & Viết ANSWERS.md](#bước-12-thu-thập-bằng-chứng-nộp-bài--viết-answersmd)
   - [Bước 13: Kịch bản trình diễn Demo trước hội đồng (Demo Runbook)](#bước-13-kịch-bản-trình-diễn-demo-trước-hội-đồng-demo-runbook)
4. [Sổ Tay Xử Lý Sự Cố Thường Gặp (Troubleshooting Runbook)](#4-sổ-tay-xử-lý-sự-cố-thường-gặp-troubleshooting-runbook)

---

## 1. Ý NGHĨA CỐT LÕI CỦA BÀI LAB 28

### 1.1. Bối cảnh bài toán: Tại sao không chỉ là "chạy một chatbot"?

Trong môi trường thực tế doanh nghiệp, việc xây dựng một ứng dụng RAG (Retrieval-Augmented Generation) không dừng lại ở việc viết một đoạn mã Python vài chục dòng gọi OpenAI API hay LangChain. Khi đưa vào vận hành thực tế (Production), hệ thống phải giải quyết hàng loạt bài toán kỹ thuật phức tạp:
1. **Lưu lượng lớn & Đảm bảo tính toàn vẹn (Data Ingestion & Integrity):** Dữ liệu tài liệu và phản hồi người dùng (feedback) gửi về liên tục qua HTTP. Nếu ghi trực tiếp vào cơ sở dữ liệu sẽ gây nghẽn (bottleneck). Cần Kafka làm bộ đệm chịu tải và phân phối.
2. **Khả năng chống lặp dữ liệu (Idempotency):** Trong mạng phân tán, hiện tượng gửi lại bản tin (Kafka replay, network retry) xảy ra thường xuyên. Làm sao để dù một bản tin bị gửi lại 10 lần thì hệ thống Lakehouse (Delta Lake), Feature Store (Feast) và Vector Store (Qdrant) vẫn chỉ có đúng **1 bản ghi mới nhất**, không làm sai lệch thống kê hay kết quả tìm kiếm?
3. **Theo dấu vết xuyên suốt (End-to-End Tracing):** Khi người dùng phàn nàn "yêu cầu của tôi bị chậm hoặc trả lời sai", làm sao kỹ sư có thể dùng **1 mã định danh duy nhất (Trace ID)** để tra cứu chính xác hành trình yêu cầu: đi qua Gateway nào, API nào tiếp nhận, bản tin Kafka gửi lúc nào, Airflow chạy task nào, Spark MERGE vào Delta lúc nào, Feast lấy đặc trưng gì, Qdrant tìm kiếm tài liệu nào, MLflow nạp model release nào, và vLLM sinh text mất bao nhiêu milli-giây?
4. **Vận hành linh hoạt & Khả năng chịu lỗi (Degraded Modes & Graceful Degradation):** Nếu Feature Store bị khởi động lại, hệ thống có sập hoàn toàn không? Câu trả lời trong kỹ thuật nền tảng là **KHÔNG**; hệ thống phải chuyển sang chế độ suy giảm dịch vụ có kiểm soát (`degraded mode`), tiếp tục trả lời câu hỏi và ghi nhận log, thay vì ngắt kết nối (`not_ready` / 503).
5. **Khả năng quay lui không downtime (Zero-downtime Rollback):** Khi triển khai một prompt mới hoặc mô hình mới gặp lỗi, làm sao quay về phiên bản trước đó ngay lập tức mà không phải build lại container hay sửa mã nguồn?
6. **Tuân thủ GitOps & Chuẩn hóa hạ tầng:** Toàn bộ cấu hình triển khai Kubernetes, Gateway API, NetworkPolicy phải được khai báo dạng mã (Infrastructure as Code - IaC), có thể kiểm chứng tính hợp lệ trước khi đưa vào cụm và hỗ trợ tự động hoàn nguyên khi có sai lệch trạng thái (drift).

**Mục tiêu tối thượng của Lab 28:** Bạn tiếp quản một nền tảng RAG đang ở dạng khung (scaffold) và phải chứng minh rằng luồng dữ liệu đi qua đầy đủ **10 ranh giới kỹ thuật (Integration Points)**, có hợp đồng dữ liệu rõ ràng, có khả năng tự phục hồi, có số liệu giám sát (metrics/traces) và có bằng chứng xác thực (evidence) không thể làm giả.

---

### 1.2. Mô hình kiến trúc 5 tầng nền tảng (5 Layers)

Hệ thống được thiết kế theo mô hình chuẩn của một Modern Data & AI Platform, chia làm 5 tầng kiến trúc rõ rệt:

```
+-----------------------------------------------------------------------------------+
| L5: PRESENTATION & DEMO                                                           |
|   - Demo Runbook, Evidence Pack, Dashboards (Grafana, Jaeger, MLflow, Airflow)    |
+-----------------------------------------------------------------------------------+
| L4: PLATFORM & OBSERVABILITY                                                      |
|   - Envoy Gateway (:8080)       - OpenTelemetry Collector (:4317/:4318)           |
|   - Prometheus (:9090)          - Grafana (:3000)                                 |
|   - Jaeger Tracing (:16686)     - Kubernetes Manifests & GitOps (Argo CD)         |
+-----------------------------------------------------------------------------------+
| L3: ML & SERVING                                                                  |
|   - FastAPI Serving API (:8000) - MLflow Model Registry (:5000, alias: champion)  |
|   - Feast Online Store (:6566)  - Qdrant Vector Store (:6333, Hybrid Search)      |
+-----------------------------------------------------------------------------------+
| L2: DATA & LAKEHOUSE                                                              |
|   - Apache Kafka (:9092)        - Apache Airflow 3 (:8082, Asset-driven Pipeline) |
|   - Apache Spark Connect (:15002) - Delta Lake (ACID Lakehouse, Time Travel)     |
+-----------------------------------------------------------------------------------+
| L1: COMPUTE & INFERENCE                                                           |
|   - Real vLLM Serving Engine (:8001 / Kaggle T4 / GPU Cluster, OpenAI-compatible) |
+-----------------------------------------------------------------------------------+
```

---

### 1.3. Ý nghĩa chi tiết của 10 Điểm Kết Nối (10 Integration Points: IP01 – IP10)

Mỗi điểm kết nối (Integration Point - IP) là một ranh giới giữa hai thành phần công nghệ trong hệ thống. Việc hiểu rõ ranh giới này giúp kỹ sư phân loại chính xác nguyên nhân khi xảy ra sự cố (root cause analysis):

| ID | Tên Ranh Giới | Đầu vào (Input Contract) | Đầu ra (Output Contract) | Tín hiệu sống (Health Signal) | Ý nghĩa thực tế |
|---|---|---|---|---|---|
| **IP01** | **Data Ingestion → Kafka** | `FeedbackSubmission` / `DocumentSubmission` (HTTP JSON qua Gateway) | `IngestionEvent` đẩy lên topic `data.raw`, message key = `idempotency_key`, kèm headers (`traceparent`, `idempotency-key`) | Broker Kafka kết nối được, topic `data.raw` tồn tại | Tiếp nhận dữ liệu phi đồng bộ, chống nghẽn HTTP API, đảm bảo mọi bản tin đều có mã vết (W3C) và khóa chống trùng ngay từ cửa ngõ. |
| **IP02** | **Kafka → Airflow Pipeline** | `IngestionEvent` tiêu thụ từ `data.raw` mang W3C `traceparent` | Airflow 3 Asset Event kích hoạt trên asset `lab28://delta/feedback`, ghi nhận `run_id` | Airflow API `/api/v2/monitor/health` báo scheduler & triggerer healthy | Điều phối xử lý dữ liệu theo sự kiện (event-driven orchestration), gom lô dữ liệu để nạp vào Lakehouse thay vì ghi vụn vặt từng dòng. |
| **IP03** | **Pipeline → Delta Lake** | Lô `IngestionEvent` đã được lọc trùng bởi `dedupe_latest` | Lệnh `MERGE INTO` ghi vào Delta table, sinh phiên bản mới trong `_delta_log` | `DeltaTable.history()` đọc được và version tăng đơn điệu | Đảm bảo tính nhất quán dữ liệu ACID. Khi Kafka replay một lô cũ, lệnh `MERGE` cập nhật bản ghi cũ thay vì ghi thêm bản ghi mới, bảo toàn tính toàn vẹn (idempotency). |
| **IP04** | **Lakehouse → Feature Store (Feast)** | Snapshot offline kết xuất từ Delta table tại `exports/asker_activity` | Bảng online store trong Feast theo entity `asker_id`, cung cấp Feature Service `asker_serving_v1` | Feast Feature Server `/health` trả về 200, lệnh `get-online-features` trả về entity | Tách biệt hoàn toàn đường dẫn xử lý offline (tính toán thống kê nặng trên Delta) và đường dẫn phục vụ online (truy vấn milli-giây từ Feast online store để nạp vào prompt RAG). |
| **IP05** | **Data → Vector Store (Qdrant)** | Dữ liệu văn bản từ Delta + embedding model định danh cố định | Điểm vector trong collection `lab28_documents` với UUID tất định sinh từ `doc_id` | Qdrant `/readyz` trả về 200, số lượng points > 0 | Tìm kiếm ngữ nghĩa lai (Hybrid Search: dense + sparse). Sử dụng UUID tất định (`uuid5`) từ `doc_id` giúp việc đánh chỉ mục lại (re-index) không làm nhân bản vector points. |
| **IP06** | **MLflow → Model Registry** | Đợt đánh giá mô hình trên phiên bản Delta cụ thể + cấu hình prompt + model id | Model version được đăng ký kèm signature, tags, provenance và gắn alias `champion` | MLflow `/health` trả về 200, truy vấn alias `champion` giải quyết được phiên bản | Quản lý vòng đời mô hình và cấu hình RAG. Cho phép kiểm soát chặt chẽ: prompt nào, chạy với vLLM nào, trên dữ liệu Delta phiên bản nào, và cho phép rollback tức thì. |
| **IP07** | **Model → vLLM Serving** | Prompt hoàn chỉnh gồm ngữ cảnh lấy từ Qdrant + đặc trưng từ Feast + câu hỏi | Phản hồi OpenAI-compatible Chat Completion từ engine vLLM thật; phản hồi kèm model ID | vLLM `/health` 200, `/version` báo build vLLM thật, `/metrics` có tiền tố `vllm:` | Cung cấp năng lực suy luận LLM thông lượng cao (High-throughput inference) qua PagedAttention. Kiểm tra định danh vLLM thật ngăn chặn việc dùng mock server giả mạo. |
| **IP08** | **Serving → API Gateway (Envoy)** | Yêu cầu HTTP từ client gửi đến cổng của Gateway | Yêu cầu được định tuyến kèm header `x-request-id`, áp dụng rate-limit (429) và health routing | Gateway admin `/ready` trả về 200, endpoint `/healthz` phản hồi độc lập | Đóng vai trò Reverse Proxy & API Gateway cấp cụm, phân tách traffic, bảo vệ backend bằng Rate Limiting và định tuyến kiểm tra sức khỏe. |
| **IP09** | **All Components → Prometheus / Grafana** | Thu thập metric (`/metrics`) từ tất cả các dịch vụ | Dashboard hiển thị Golden Signals trên Grafana và cấu hình Alert SLO | Prometheus `/api/v1/targets` báo toàn bộ scrape targets ở trạng thái `UP` | Giám sát vận hành thời gian thực. Theo dõi 4 tín hiệu vàng (Golden Signals: Latency, Traffic, Errors, Saturation) và Kafka Consumer Lag để phát hiện nghẽn. |
| **IP10** | **All Components → Distributed Tracing** | Spans OpenTelemetry (OTLP) gửi từ Gateway, API, Kafka, Airflow, Spark, Feast, Qdrant, vLLM | Một Trace ID duy nhất mang đầy đủ các spans quy định, xuất sang Jaeger và LangSmith | Pipeline collector healthy; truy vấn được toàn bộ cây span trên Jaeger UI theo Trace ID | Khả năng quan sát chuyên sâu (Observability). Liên kết mọi hành động trong toàn bộ hệ thống phân tán vào một cây phân tích thời gian thực duy nhất. |

---

### 1.4. Ý nghĩa 5 Hành trình Kiểm thử Sống (5 Critical Journeys: J1 – J5)

Để kiểm chứng hệ thống không chỉ "xanh trên lý thuyết", bài lab định nghĩa 5 bài kiểm thử tích hợp toàn diện (`integration-tests/`):
- **Journey 1 (`IT-J1-golden-path`):** Hành trình lý tưởng. Một văn bản hoặc phản hồi đi từ Gateway → API → Kafka → Airflow DAG → Spark MERGE vào Delta → Feast materialize → Qdrant index → API `/ask` truy vấn Feast + Qdrant → vLLM sinh câu trả lời → Ghi nhận trace và metric xuyên suốt.
- **Journey 2 (`IT-J2-idempotent-replay`):** Kiểm thử tính chống trùng lặp. Gửi lại cùng một lô dữ liệu 2–3 lần qua Kafka. Kiểm chứng số dòng trong Delta Lake không tăng, Feast không tạo entity rác, Qdrant giữ nguyên số điểm vector.
- **Journey 3 (`IT-J3-promotion-rollback`):** Kiểm thử quản lý phát hành. Đăng ký một cấu hình release mới, chuyển alias `champion` sang bản mới, kiểm tra hành vi phục vụ của API thay đổi tương ứng. Sau đó thực hiện `rollback` alias về phiên bản trước mà không cần sửa một dòng mã nào hay khởi động lại service.
- **Journey 4 (`IT-J4-degraded-recovery`):** Kiểm thử khả năng chịu lỗi và tự phục hồi. Tắt một dịch vụ tùy chọn (ví dụ: Feast). Hệ thống API `/ready` phải phát hiện và chuyển trạng thái sang `degraded` thay vì sập; endpoint `/ask` vẫn trả lời được (bỏ qua phần đặc trưng cá nhân hóa). Khi bật lại Feast, hệ thống tự động nhận diện và khôi phục trạng thái `ready` mà không mất mát dữ liệu.
- **Journey 5 (`IT-J5-trace-metrics-continuity`):** Kiểm thử tính liên tục của quan sát. Xác thực rằng một yêu cầu gửi qua Envoy giữ nguyên Trace ID cho tới khi trả về client, tất cả 11 spans bắt buộc đều xuất hiện trong cây Trace của Jaeger, và các chỉ số trên Prometheus phản ánh đúng lưu lượng vừa gửi.

---

### 1.5. Các nguyên lý Platform Engineering bắt buộc nắm vững

1. **W3C Trace Context Propagation:** Header `traceparent` có định dạng `00-{trace_id}-{parent_id}-{trace_flags}`. Khi đi qua Kafka, nó phải được chuyển đổi sang kiểu `bytes` trong Kafka headers. Nếu không có trace cha, phải bỏ qua (omit), tuyệt đối không gửi chuỗi rỗng để tránh làm hỏng parser chuẩn của OpenTelemetry.
2. **Delta MERGE Idempotency:** Trong Spark SQL, lệnh `MERGE INTO target USING source ON target.idempotency_key = source.idempotency_key WHEN MATCHED THEN UPDATE SET * WHEN NOT MATCHED THEN INSERT *` đòi hỏi lô nguồn (`source`) phải chứa các khóa duy nhất. Nếu trong cùng một lô từ Kafka có 2 bản tin cùng key, Spark sẽ bắn lỗi JVM `AnalysisException`. Do đó, hàm `dedupe_latest` ở client phải tiền xử lý, chọn đúng bản ghi có `(occurred_at, event_id)` mới nhất trước khi đưa vào Spark.
3. **Deterministic Identity trong Vector Store:** Khi đẩy tài liệu vào Qdrant, nếu dùng số thứ tự tự tăng hoặc UUID ngẫu nhiên (`uuid4()`), mỗi lần re-index sẽ tạo ra một điểm vector mới gây phình dữ liệu và sai lệch kết quả tìm kiếm. Sử dụng `uuid5(NAMESPACE, doc_id)` đảm bảo rằng cùng một `doc_id` sẽ luôn sinh ra đúng một UUID duy nhất, biến thao tác upsert thành idempotent.
4. **Ngữ nghĩa Trạng thái Sẵn sàng (Tri-state Readiness Semantics):**
   - `ready`: Toàn bộ các phụ thuộc (dependencies) bắt buộc và tùy chọn đều khỏe mạnh. Pod sẵn sàng nhận lưu lượng tối đa từ Gateway.
   - `degraded`: Toàn bộ các phụ thuộc bắt buộc (`mandatory=True`) đều hoạt động bình thường, nhưng có ít nhất một phụ thuộc tùy chọn (`mandatory=False` - ví dụ Feast cold/down) gặp sự cố. Pod vẫn được giữ trong danh sách phục vụ của Gateway nhưng phục vụ với tính năng rút gọn.
   - `not_ready`: Có ít nhất một phụ thuộc bắt buộc bị lỗi (ví dụ Kafka down, Qdrant down, MLflow down). Endpoint `/ready` trả về HTTP 503 để Gateway lập tức gỡ Pod này ra khỏi luồng điều phối, tránh trả về lỗi 500 cho người dùng cuối.

---

## 2. CƠ CẤU ĐÁNH GIÁ & PHÂN VAI NHÓM

### 2.1. Thang điểm 100 chi tiết (Rubric)

| Tiêu chí | Điểm | Điều kiện đạt điểm tối đa |
|---|:---:|---|
| **10 Integration Points** | **40** | Mỗi IP đạt 4 điểm: hoàn thiện mã nguồn và tạo ra file evidence thật, có dữ liệu sống (live evidence). *(Thiếu IP01–IP07 thì tối đa bài lab chỉ được 60 điểm)*. |
| **Correctness & Recovery** | **15** | Vượt qua 5 journeys (J1–J5), chứng minh tính idempotency khi Kafka replay, chứng minh không mất dữ liệu sau khi khôi phục sự cố. |
| **Observability** | **15** | Đầy đủ 11 required spans trên Jaeger trace, Dashboard hiển thị đủ 4 golden signals trên Grafana, alert rules hoạt động chính xác. |
| **Production Readiness** | **15** | K8s manifests & Gateway API hợp lệ theo `validate_manifests.py`, readiness probes phân biệt rõ degraded, profile tải P50/P95/P99 có phân tích nút thắt cổ chai, quy trình GitOps rollback tái lập được. |
| **Demo & Explanation** | **10** | Trình bày mạch lạc sơ đồ kiến trúc, diễn tập sự cố có giả thuyết và kiểm chứng rõ ràng, trả lời tốt câu hỏi Q&A phản biện. |
| **Engineering Quality** | **5** | Vượt qua linter Ruff, chạy đa nền tảng (Windows/macOS/Linux) không lỗi, không commit secret/cache/database/.env lên Git. |

> [!CAUTION]
> **Quy định nghiêm ngặt:** Làm giả vLLM (dùng mock server không có build vLLM), làm giả trace/metrics hoặc sửa file test để ép pass sẽ nhận **0 điểm** cho phần tương ứng.

---

### 2.2. Bảng phân vai nhóm (Team Role Cards)

Nếu làm theo nhóm, phân chia công việc theo 5 vai trò chuyên biệt. Nếu làm cá nhân, bạn sẽ lần lượt đảm nhiệm tất cả các vai trò này:

1. **Ingestion & Orchestration Engineer (Phụ trách IP01, IP02):**
   - Quản lý Kafka topics, partitions, retention policy.
   - Xử lý mã vết `traceparent` và `idempotency-key` trong Kafka headers.
   - Cấu hình Airflow DAG, asset-driven scheduling, retry chính sách và cơ chế Dead Letter Queue (DLQ).
2. **Data & ML Engineer (Phụ trách IP03, IP04, IP06):**
   - Đảm bảo tính nhất quán của Delta Lake qua hàm `dedupe_latest` và câu lệnh `MERGE SQL`.
   - Kết xuất dữ liệu offline từ Delta và quản lý Feature Store Feast (`asker_activity_v1`).
   - Quản lý đăng ký mô hình, siêu dữ liệu provenance trên MLflow và alias `champion`.
3. **Serving & Retrieval Engineer (Phụ trách IP05, IP07):**
   - Quản lý Qdrant collection, đánh chỉ mục vector với UUID tất định (`doc_id`).
   - Xây dựng logic RAG Grounding và tích hợp với vLLM thật qua OpenAI-compatible API.
   - Thiết lập cơ chế fallback khi mô hình suy luận hoặc vector store gặp sự cố.
4. **Platform & Observability Engineer (Phụ trách IP08, IP09, IP10):**
   - Cấu hình Envoy Gateway (định tuyến, rate limiting 429, header `x-request-id`).
   - Thu thập chỉ số qua Prometheus, thiết kế dashboard Grafana và cảnh báo SLO.
   - Cấu hình OpenTelemetry Collector, đảm bảo tính liên tục của W3C Trace qua Jaeger.
   - Quản lý K8s manifests, NetworkPolicy, kustomization và kịch bản GitOps với Argo CD.
5. **Presenter / Incident Commander:**
   - Xây dựng kịch bản demo tổng thể theo đúng [Demo Runbook](docs/demo-runbook.md).
   - Điều phối diễn tập sự cố: nêu giả thuyết → quan sát tín hiệu → khôi phục → chứng minh toàn vẹn dữ liệu.
   - Chuẩn bị slide kiến trúc và chủ trì phần hỏi đáp phản biện (Q&A).

---

### 2.3. Yêu cầu hồ sơ nộp bài (Submission Checklist)

Theo file [`SUBMISSION.md`](SUBMISSION.md), thư mục nộp bài cần có đủ 8 thành phần sau:
- [ ] 1. File `integration-report.json` và kết quả chạy bộ kiểm thử nhanh (fast suite).
- [ ] 2. Đủ 10 file bằng chứng (evidence) trong thư mục `evidence/` đúng tên quy định trong matrix:
  - `ip01-kafka-consume.json`
  - `ip02-airflow-run.json`
  - `ip03-delta-history.json`
  - `ip04-feast-online.json`
  - `ip05-qdrant-search.json`
  - `ip06-mlflow-release.json`
  - `ip07-vllm-identity.json`
  - `ip08-gateway.json`
  - `ip09-prometheus-targets.json` (và `ip09-grafana-dashboards.json`)
  - `ip10-trace.json`
- [ ] 3. Sơ đồ kiến trúc và phân chia trách nhiệm (Architecture & Ownership diagram).
- [ ] 4. Bằng chứng Happy-path trace có đủ: Run ID, Trace ID, Delta Table Version, MLflow Release Version.
- [ ] 5. Bản ghi kịch bản diễn tập sự cố và bằng chứng không mất mát dữ liệu (No-data-loss proof).
- [ ] 6. Báo cáo đo tải hiệu năng (Load profile) với các chỉ số P50, P95, P99 và phân tích điểm nghẽn.
- [ ] 7. Kết quả xác thực Kubernetes / GitOps manifests và bằng chứng phát hiện drift / rollback.
- [ ] 8. File `ANSWERS.md`: Trình bày các đánh đổi kỹ thuật (trade-offs), khoảng cách giữa bài lab và môi trường production thực tế (production gaps), và bảng đóng góp chi tiết của từng thành viên.

> [!WARNING]
> Tuyệt đối **KHÔNG** đưa vào commit Git: file `.env`, mật khẩu, token, dữ liệu tạm, thư mục `.lab28/`, file SQLite `.db` hay trọng số mô hình LLM.

---

## 3. HƯỚNG DẪN THỰC HÀNH TỪNG BƯỚC (STEP-BY-STEP EXECUTION)

---

### Bước 1: Chuẩn bị công cụ và khởi tạo nhánh làm việc

Mở PowerShell (Windows) hoặc Terminal (macOS/Linux) tại thư mục dự án và kiểm tra các công cụ nền tảng:

```bash
git --version
uv --version
docker version
docker compose version
```

Tạo nhánh làm việc theo định dạng quy định:
- **Nếu làm cá nhân:**
  ```bash
  git switch -c ca-nhan-nguyenhoangviet
  ```
- **Nếu làm theo nhóm:**
  ```bash
  git switch -c nhom-01
  ```

Kiểm tra trạng thái nhánh:
```bash
git status
```

---

### Bước 2: Thiết lập môi trường Python bằng uv

Sử dụng `uv` để tạo môi trường ảo Python 3.11 độc lập, đồng bộ đầy đủ các gói mở rộng phát triển (`dev`) và kiểm thử tích hợp (`integration`):

```bash
uv sync --frozen --python 3.11 --extra dev --extra integration --no-editable
```

> [!NOTE]
> Cờ `--no-editable` giúp tránh các lỗi không tương thích về quyền ghi và đồng bộ hệ thống tệp giữa host OS và container volume.

Kiểm tra công cụ CLI của bài lab:
```bash
uv run lab28 --help
uv run lab28 preflight
```

**Phân tích kết quả lệnh `preflight`:**
- Nếu `profile: local-standard`: Máy của bạn đủ điều kiện chạy Docker Compose cho hệ thống cơ bản hoặc toàn bộ hệ thống.
- Nếu `profile: browser-fallback`: Máy thiếu RAM/CPU hoặc không bật được Docker daemon. Bạn vẫn có thể viết mã hoàn thiện 4 chức năng, chạy toàn bộ bộ test unit cục bộ và kiểm tra manifests; phần chạy Docker có thể mượn máy chung hoặc chạy trên cloud/server do giảng viên cấp.

---

### Bước 3: Xác nhận trạng thái ban đầu (Starter Tests)

Chạy kiểm thử khởi đầu để xác nhận bộ khung ban đầu:

```bash
uv run pytest starter-tests -q
```

**Kết quả mong đợi:**
Chính xác **4 bài test bị FAILED** với lỗi `NotImplementedError`. Đây là trạng thái đúng chuẩn của repo khi bắt đầu, tương ứng với 4 hàm bạn cần tự tay cài đặt trong tệp `src/lab28_platform/integration_tasks.py`.

---

### Bước 4: Hoàn thiện 4 chức năng cốt lõi trong mã nguồn

Mở tệp [`src/lab28_platform/integration_tasks.py`](file:///C:/Users/Nguye/Desktop/aithucchien/track2/lab/Day28-NguyenHoangViet-2A202601940/src/lab28_platform/integration_tasks.py) để chỉnh sửa. Dưới đây là phân tích chi tiết logic và mã cài đặt chuẩn cho từng hàm:

#### 1. Hàm `event_headers` (Ranh giới IP01 & IP10)
- **Mục đích:** Gắn thông tin ngữ cảnh vào bản tin Kafka khi API gửi dữ liệu lên topic `data.raw`.
- **Yêu cầu kỹ thuật:**
  - Header `idempotency-key` là bắt buộc, giá trị phải mã hóa dạng `bytes` (utf-8).
  - Header `traceparent` (mã vết W3C) chỉ được thêm vào khi có giá trị hợp lệ (`traceparent is not None` và không rỗng), giá trị mã hóa dạng `bytes`.
  - Nếu không có trace cha (`traceparent` là None), **tuyệt đối không gửi header rỗng**, phải lược bỏ khỏi danh sách headers.
- **Mã cài đặt:**
```python
def event_headers(
    traceparent: str | None, idempotency_key: str
) -> list[tuple[str, bytes]]:
    """Return byte-valued Kafka headers for trace and replay correlation."""
    headers: list[tuple[str, bytes]] = []
    if traceparent:
        headers.append(("traceparent", traceparent.encode("utf-8")))
    headers.append(("idempotency-key", idempotency_key.encode("utf-8")))
    return headers
```

#### 2. Hàm `dedupe_latest` (Ranh giới IP03)
- **Mục đích:** Lọc trùng lặp dữ liệu từ một lô (batch) sự kiện Kafka trước khi nạp vào Spark Delta MERGE.
- **Yêu cầu kỹ thuật:**
  - Gom nhóm các sự kiện theo `idempotency_key`.
  - Trong trường hợp cùng một key xuất hiện nhiều lần (do Kafka redelivery/replay), chọn bản ghi có cặp giá trị `(occurred_at, event_id)` lớn nhất (mới nhất).
  - Sắp xếp danh sách kết quả cuối cùng theo thứ tự tăng dần của `idempotency_key` để đảm bảo tính tất định (deterministic order) giữa các lần chạy.
  - Xử lý mượt mà trường hợp danh sách đầu vào rỗng.
- **Mã cài đặt:**
```python
def dedupe_latest(events: Iterable[IngestionEvent]) -> list[IngestionEvent]:
    """Return one newest event per idempotency key, in deterministic key order."""
    newest: dict[str, IngestionEvent] = {}
    for event in events:
        key = event.idempotency_key
        if key not in newest:
            newest[key] = event
        else:
            current = newest[key]
            if (event.occurred_at, event.event_id) > (current.occurred_at, current.event_id):
                newest[key] = event
    return [newest[key] for key in sorted(newest)]
```

#### 3. Hàm `feast_online_request` (Ranh giới IP04)
- **Mục đích:** Xây dựng cấu trúc payload truy vấn đặc trưng người dùng từ Feast Feature Server qua endpoint `/get-online-features`.
- **Yêu cầu kỹ thuật:**
  - `entities`: Dictionary ánh xạ entity `"asker_id"` sang mảng chứa `[asker_id]`.
  - `features`: Danh sách 4 đặc trưng chuẩn của Feature View `asker_activity_v1` lấy trực tiếp từ hằng số `FEATURE_REFS` trong `lab28_platform.contracts` (tránh hardcode thủ công):
    - `asker_activity_v1:feedback_count`
    - `asker_activity_v1:avg_rating`
    - `asker_activity_v1:negative_ratio`
    - `asker_activity_v1:delta_version`
  - `full_feature_names`: Đặt là `False`.
- **Mã cài đặt:**
```python
from lab28_platform.contracts import FEATURE_REFS

def feast_online_request(asker_id: str) -> dict[str, Any]:
    """Build the Feast /get-online-features request for asker_activity_v1."""
    return {
        "entities": {"asker_id": [asker_id]},
        "features": list(FEATURE_REFS),
        "full_feature_names": False,
    }
```

#### 4. Hàm `readiness_status` (Ranh giới IP07 & IP08)
- **Mục đích:** Đánh giá mức độ sẵn sàng phục vụ của Pod dựa trên kết quả thăm dò (probes) các dịch vụ phụ thuộc.
- **Yêu cầu kỹ thuật (thứ tự ưu tiên nghiêm ngặt):**
  1. Nếu có **ít nhất một** probe có `mandatory=True` mà `ready=False` → Trả về `"not_ready"`.
  2. Nếu tất cả probe bắt buộc đều sẵn sàng (`mandatory=True` và `ready=True`), nhưng có probe tùy chọn (`mandatory=False`) bị `ready=False` → Trả về `"degraded"`.
  3. Nếu tất cả probe đều `ready=True` → Trả về `"ready"`.
- **Mã cài đặt:**
```python
def readiness_status(probes: Iterable[dict[str, Any]]) -> str:
    """Return ready, degraded or not_ready from probe severity."""
    has_degraded = False
    for probe in probes:
        is_ready = bool(probe.get("ready", True))
        is_mandatory = bool(probe.get("mandatory", True))
        if not is_ready:
            if is_mandatory:
                return "not_ready"
            has_degraded = True
    return "degraded" if has_degraded else "ready"
```

---

### Bước 5: Kiểm tra chất lượng mã nguồn & Hợp đồng tích hợp

Sau khi điền xong 4 hàm trên, chạy toàn bộ bộ kiểm tra nhanh (fast check suite):

```bash
# 1. Kiểm tra 4 hàm khởi đầu (bây giờ phải pass 100%)
uv run pytest starter-tests -q

# 2. Kiểm tra bộ unit tests mở rộng (Delta merge idempotency, contracts, vector IDs, release provenance)
uv run pytest tests -q

# 3. Quét định dạng và quy chuẩn mã nguồn với Ruff
uv run ruff check .

# 4. Kiểm tra tính toàn vẹn của ma trận tích hợp 10 điểm
uv run python scripts/verify_matrix.py

# 5. Kiểm tra tính tương thích đa nền tảng (Windows/macOS/Linux)
uv run python scripts/check_portability.py

# 6. Xác thực cú pháp và schema của các tệp Kubernetes/GitOps manifests
uv run python scripts/validate_manifests.py
```

**Kết quả mong đợi:** Tất cả các lệnh trên phải trả về mã kết thúc `0` (Exit Code 0), không có lỗi linter, không có bài test nào bị fail. **Nếu bước này chưa pass hoàn toàn, tuyệt đối chưa chuyển sang Docker.**

---

### Bước 6: Kiểm tra cấu hình Docker trước khi tải image

Kiểm tra tính hợp lệ của tệp Docker Compose và các cổng dịch vụ:

```bash
docker compose --env-file ports.template config --quiet
docker compose --env-file ports.template --profile full config --quiet
```

Nếu lệnh chạy êm không in ra thông báo lỗi cú pháp YAML nghĩa là cấu hình mạng và biến môi trường hoàn toàn hợp lệ.

> [!TIP]
> Tệp `ports.template` chứa danh sách cổng mặc định của máy chủ. Nếu máy bạn bị trùng một cổng nào đó (ví dụ cổng 3000 của Grafana hoặc cổng 8080 của Envoy), hãy sao chép ra tệp `ports.local.env`, chỉnh sửa số cổng trên máy host, và truyền `--env-file ports.local.env` vào các lệnh Docker Compose sau này.

---

### Bước 7: Chạy hệ thống cơ bản (Core Stack Checkpoint)

Khởi động hệ thống lõi (gồm Kafka, Qdrant, Feast, MLflow, API, Gateway, Jaeger, Prometheus, Grafana):

```bash
# 1. Bật container và chờ các healthcheck hoàn tất
docker compose --env-file ports.template up -d --build --wait

# 2. Kiểm tra trạng thái container (tất cả phải running hoặc healthy)
docker compose --env-file ports.template ps

# 3. Tạo các Kafka topics chuẩn
uv run lab28 topics

# 4. Đánh chỉ mục dữ liệu ban đầu vào Qdrant Vector Store
uv run lab28 index --source file

# 5. Đăng ký cấu hình RAG Release ban đầu vào MLflow Model Registry (gắn alias champion)
uv run lab28 release

# 6. Đẩy dữ liệu mẫu (documents & feedback) qua Envoy Gateway vào hệ thống
uv run lab28 seed --via-gateway

# 7. Thăm dò tình trạng kết nối tới toàn bộ các dịch vụ
uv run lab28 inspect

# 8. Kiểm tra trạng thái sẵn sàng của hệ thống
uv run lab28 ready
```

**Các trang Web UI cần mở trên trình duyệt để kiểm tra:**
- **Envoy Gateway:** `http://localhost:8080/health` (Kiểm tra IP08 định tuyến)
- **FastAPI Docs:** `http://localhost:8000/docs` (Xem tài liệu API)
- **Grafana Dashboard:** `http://localhost:3000` (User: `admin` / Pass: `admin` - Giám sát IP09)
- **Prometheus Targets:** `http://localhost:9090/targets` (Xác nhận các targets đều `UP`)
- **Jaeger Tracing:** `http://localhost:16686` (Tìm kiếm Trace theo service `lab28-gateway` hoặc `lab28-api`)
- **MLflow UI:** `http://localhost:5000` (Xem model version 1 đang mang alias `champion`)
- **Qdrant Dashboard:** `http://localhost:6333/dashboard` (Xem collection `lab28_documents` đã có points)

*Lưu ý:* Nếu chưa kết nối vLLM GPU thật, lệnh `lab28 ready` có thể báo `degraded` ở dịch vụ `vllm`. Đây là trạng thái bình thường đã được thiết kế sẵn.

---

### Bước 8: Chạy toàn bộ luồng Data & ML (Full Stack Checkpoint)

Trên máy có đủ tài nguyên (tối thiểu 12–16 GB RAM, 6 CPU), khởi động thêm **Spark Connect** và **Airflow 3**:

```bash
# 1. Khởi động thêm profile full
docker compose --env-file ports.template --profile full up -d --build --wait

# 2. Nạp thêm dữ liệu qua gateway
uv run lab28 seed --via-gateway

# 3. Chạy kiểm thử luồng hoàng kim Golden Path (J1)
uv run pytest integration-tests/test_j1_golden_path.py -q

# 4. Chạy kiểm thử chống trùng lặp Idempotent Replay (J2)
uv run pytest integration-tests/test_j2_idempotent_replay.py -q
```

Mở giao diện Airflow tại `http://localhost:8082` (Đăng nhập: `airflow` / `admin`). Tìm DAG `lab28_ingestion_pipeline` để quan sát quy trình xử lý dữ liệu tự động từ Kafka sang Delta Lake.

Khi hai hành trình J1 và J2 đã đạt, chạy toàn bộ bộ kiểm thử tích hợp (bỏ qua gate GPU và LangSmith nếu không có credential):

```bash
uv run pytest integration-tests -m "not gpu and not langsmith" -q
```

---

### Bước 9: Tích hợp vLLM thật trên GPU (Local hoặc Kaggle T4)

Để đạt trọn vẹn điểm cho **IP07 (Model Serving vLLM)**, hệ thống yêu cầu kết nối với một engine vLLM thật (v0.26+ hoặc v0.28). Máy chủ phải trả về định danh `/version` thật của vLLM và xuất metric có tiền tố `vllm:`.

#### Lựa chọn A: Nếu máy bạn có card đồ họa NVIDIA (VRAM >= 8GB)
Chạy docker compose đè cấu hình GPU:
```bash
docker compose -f compose.yaml -f compose.gpu.yaml --env-file ports.template up -d vllm
```

#### Lựa chọn B: Sử dụng Kaggle GPU T4 miễn phí
Làm theo hướng dẫn tại [`KAGGLE_GPU_EXTENSION.md`](KAGGLE_GPU_EXTENSION.md):
1. Mở một notebook mới trên Kaggle, chọn Accelerator là **GPU T4 x2**.
2. Cài đặt và khởi chạy vLLM phục vụ mô hình nhỏ (ví dụ `Qwen/Qwen3-1.7B` hoặc `Qwen/Qwen3-4B-Instruct-2507`):
   ```bash
   !pip install -q "vllm==0.26.0"
   !vllm serve Qwen/Qwen3-1.7B --host 0.0.0.0 --port 8000 --dtype half --max-model-len 2048 --gpu-memory-utilization 0.85
   ```
3. Tạo tunnel kết nối (qua ngrok hoặc Cloudflare Tunnel) để lấy public URL.
4. Cấu hình biến môi trường trong file `.env` hoặc truyền vào API:
   ```text
   LAB28_VLLM_BASE_URL=https://<your-tunnel-url>/v1
   LAB28_VLLM_MODEL_ID=Qwen/Qwen3-1.7B
   LAB28_VLLM_REQUIRE_REAL=true
   ```
5. Chạy kiểm thử IP07:
   ```bash
   uv run pytest integration-tests/test_j1_golden_path.py -k vllm -q
   ```

---

### Bước 10: Diễn tập sự cố & Kiểm tra hiệu năng (Failure Injection & Load Testing)

#### 1. Đo kiểm hiệu năng (Load Testing)
Chạy script kiểm tra tải với 8 luồng đồng thời:
```bash
uv run python load-tests/run_profile.py --requests 200 --workers 8
```
Sau đó lặp lại với 16 luồng để tìm giới hạn bão hòa (saturation point):
```bash
uv run python load-tests/run_profile.py --requests 200 --workers 16
```
Ghi nhận các giá trị độ trễ **P50, P95, P99**, tỷ lệ lỗi (Error Rate) và tình trạng tiêu thụ CPU/RAM.

#### 2. Diễn tập kịch bản sự cố (Failure Injection Runbook)
Thực hiện theo bảng hướng dẫn trong [`runbooks/failure-injection.md`](runbooks/failure-injection.md):
- **Kịch bản: Dừng Feature Store (Feast Down):**
  1. *Dự đoán giả thuyết:* Khi tắt Feast, endpoint `/ready` không được trả về 503 mà phải chuyển sang `degraded`. Yêu cầu `/api/v1/ask` vẫn phải thành công với lý do `degraded: feature store unavailable`.
  2. *Bơm sự cố:*
     ```bash
     docker compose stop feast
     ```
  3. *Quan sát:* Chạy `uv run lab28 ready` → Thấy trạng thái `degraded`. Gửi câu hỏi `uv run lab28 ask "Kiểm tra sự cố"` → Vẫn nhận được câu trả lời.
  4. *Khôi phục:*
     ```bash
     docker compose start feast
     ```
  5. *Chứng minh phục hồi:* Chạy `uv run lab28 ready` → Trở lại trạng thái `ready` hoàn toàn.

- **Kịch bản: Replay Kafka (Chứng minh không mất và không lặp dữ liệu):**
  1. Ghi nhận số dòng trong Delta Table: `uv run lab28 inspect`.
  2. Phát lại cùng một tệp dữ liệu mẫu: `uv run lab28 seed --via-gateway`.
  3. Kiểm tra lại lịch sử Delta: Số phiên bản (version) tăng lên do có commit MERGE, nhưng tổng số dòng dữ liệu (rows count) vẫn giữ nguyên không đổi.

---

### Bước 11: Xác thực cấu hình Kubernetes & GitOps

1. Kiểm tra tính hợp lệ của toàn bộ manifests triển khai Kubernetes:
   ```bash
   uv run python scripts/validate_manifests.py
   ```
2. Thực hiện quy trình GitOps Drift & Rollback theo [`runbooks/gitops-rollback.md`](runbooks/gitops-rollback.md):
   - Thay đổi số lượng `replicas` hoặc image tag trong file `deploy/kubernetes/base/api.yaml`.
   - Xem sự khác biệt bằng `git diff`.
   - Giải thích cơ chế tự phục hồi (Self-healing) của Argo CD: Khi có sự sai lệch giữa Git (Desired State) và Kubernetes Cluster (Actual State), Argo CD sẽ tự động đồng bộ để đưa cluster về đúng trạng thái khai báo trong Git.

---

### Bước 12: Thu thập bằng chứng nộp bài & Viết ANSWERS.md

#### 1. Xuất gói bằng chứng (Evidence Pack)
Chạy lệnh xuất bằng chứng tự động:
```bash
uv run lab28 evidence
```
Lệnh này sẽ ghi các file bằng chứng vào thư mục `evidence/`:
- `ip03-delta-history.json`
- `ip05-qdrant-search.json`
- `ip06-mlflow-release.json`
- `ip07-vllm-identity.json`
- `integration-report.json`

Các file bằng chứng còn lại sẽ được tạo ra khi bạn chạy các bài kiểm tra tích hợp tương ứng:
- `ip01-kafka-consume.json`, `ip02-airflow-run.json`, `ip04-feast-online.json`: Được sinh ra từ `test_j1_golden_path.py`.
- `ip08-gateway.json`: Được sinh ra từ `test_gateway_rate_limit.py`.
- `ip09-prometheus-targets.json`: Được sinh ra từ `test_prometheus_targets.py`.
- `ip10-trace.json`: Được sinh ra từ `test_trace_span_coverage.py`.

Chạy kiểm tra điểm số tích hợp cuối cùng:
```bash
uv run lab28 integration
```

#### 2. Tạo và hoàn thiện file `ANSWERS.md`
Tạo file `ANSWERS.md` ở thư mục gốc để trả lời các câu hỏi bắt buộc:
1. **Trade-offs (Đánh đổi kiến trúc):**
   - Tại sao chọn Kafka kết hợp Micro-batching với Airflow thay vì ghi streaming trực tiếp vào Delta Lake? (Đánh đổi giữa độ trễ realtime và chi phí tài nguyên tính toán/tính toàn vẹn transaction log).
   - Tại sao chọn lưu vector embedding dạng UUID tất định từ `doc_id` thay vì tự tăng? (Đánh đổi tính đơn giản lấy tính idempotent khi re-index).
2. **Production Gaps (Khoảng cách với môi trường thực tế):**
   - Cần bổ sung gì khi đưa lên production thực thụ? (Ví dụ: Kafka cluster multi-broker có SASL/SSL, Delta Lake trên S3/GCS có catalog Unity/Glue, Secret Manager thay cho biến môi trường, Distributed Tracing với sampling rate hợp lý, CI/CD pipeline tự động hóa kiểm thử).
3. **Team Contributions:**
   - Liệt kê rõ ràng ai phụ trách vai trò nào, mã nguồn đã viết và bằng chứng đã thu thập.

---

### Bước 13: Kịch bản trình diễn Demo trước hội đồng (Demo Runbook)

Theo đúng trình tự 8 bước chuẩn trong [`docs/demo-runbook.md`](docs/demo-runbook.md):
1. **Kiến trúc (Architecture):** Mở sơ đồ kiến trúc, giải thích 5 tầng, chỉ rõ người phụ trách từng phần và 10 ranh giới tích hợp (IP01–IP10).
2. **Luồng chuẩn (Happy Path):**
   - Gửi một tài liệu mới qua Envoy Gateway (`uv run lab28 seed --limit 1 --via-gateway`).
   - Mở Airflow UI xem DAG trigger và hoàn thành task.
   - Chỉ ra phiên bản Delta mới tăng lên, Feast có online entity, Qdrant có vector point mới.
   - Gọi câu hỏi RAG: `uv run lab28 ask "Câu hỏi kiểm tra" --via-gateway`.
3. **Theo dấu vết (Distributed Trace):**
   - Lấy `trace_id` từ kết quả câu lệnh vừa chạy, dán vào ô tìm kiếm của **Jaeger UI**.
   - Mở chi tiết Trace và chỉ ra cây gồm đầy đủ 11 spans (Gateway → API → Kafka produce/consume → Airflow → Spark MERGE → Ask → Feast → Qdrant → MLflow → vLLM).
4. **Tín hiệu vàng (Golden Signals):**
   - Mở **Grafana Dashboard**: Chỉ rõ 4 tín hiệu vàng (Request Rate, Error Rate, Latency P50/P95, Saturation) và biểu đồ Kafka Consumer Lag.
5. **Diễn tập sự cố (Incident & Recovery):**
   - Tuyên bố kịch bản: Giả lập Feast bị sập.
   - Nêu trước dấu hiệu: API sẽ chuyển sang trạng thái `degraded`, không trả lỗi 503.
   - Dừng Feast (`docker compose stop feast`), gọi thử `/ready` và `/ask`.
   - Bật lại Feast, chứng minh hệ thống tự phục hồi về `ready` mà không mất bất kỳ dữ liệu nào.
6. **Thăng hạng & Quay lui mô hình (Promotion & Rollback):**
   - Đăng ký một prompt template v2 qua MLflow: `uv run lab28 release --prompt-version v2`.
   - Chứng minh câu trả lời từ `/ask` đã áp dụng prompt mới.
   - Thực hiện quay lui ngay lập tức: `uv run lab28 rollback`.
   - Chứng minh alias `champion` đã quay về v1 và hệ thống lập tức trả về câu trả lời theo khuôn mẫu v1.
7. **GitOps & Hạ tầng (GitOps Drift & Rollback):**
   - Trình bày file manifest Kubernetes `deploy/kubernetes/base/api.yaml`.
   - Giải thích cách thức Argo CD phát hiện sự sai khác (drift) và tự động hoàn nguyên về trạng thái mong muốn (desired state).
8. **Hỏi đáp phản biện (Q&A):**
   - Giải thích các quyết định thiết kế kỹ thuật, bài toán chi phí hạ tầng/GPU, cơ chế bảo mật và phân vùng mạng (NetworkPolicy).

---

## 4. SỔ TAY XỬ LÝ SỰ CỐ THƯỜNG GẶP (TROUBLESHOOTING RUNBOOK)

| Triệu chứng lỗi | Nguyên nhân gốc rễ | Cách khắc phục nhanh |
|---|---|---|
| `uv: command not found` | Chưa nạp đường dẫn cài đặt của `uv` vào biến môi trường `PATH`. | Đóng mở lại Terminal, hoặc chạy script cài đặt chính thức của astral `powershell -c "irm https://astral.sh/uv/install.ps1 \| iex"`. |
| Chạy `starter-tests` không đúng 4 lỗi | Chạy sai môi trường Python hệ thống hoặc file test bị sửa đổi. | Sử dụng đúng lệnh `uv run pytest starter-tests -q`. Nếu cần, dùng `git checkout starter-tests/` để hoàn nguyên file test ban đầu. |
| Lỗi cổng `port is already allocated` | Một số cổng mặc định (8080, 5000, 3000, 9092) đang bị dịch vụ khác trên máy chiếm dụng. | Tạo file `ports.local.env` từ `ports.template`, thay đổi cổng host bị trùng (ví dụ đổi 8080 thành 18080), rồi thêm `--env-file ports.local.env` vào các lệnh compose. |
| Container báo trạng thái `unhealthy` | Dịch vụ khởi động chậm do thiếu RAM/CPU hoặc phụ thuộc chưa sẵn sàng. | Xem log chi tiết bằng `docker compose --env-file ports.template logs <tên-container>` để tìm dòng lỗi đầu tiên. Tăng RAM cấp cho Docker Desktop (khuyến nghị tối thiểu 8GB - 12GB). |
| Endpoint `/ready` báo `not_ready` | Có ít nhất một dịch vụ bắt buộc (`mandatory=True`) chưa kết nối được. | Chạy `uv run lab28 inspect` để quét từng dịch vụ. Tìm dịch vụ nào báo lỗi kết nối và khởi động lại container đó. |
| Delta Lake không tăng phiên bản khi replay | Quá trình Spark MERGE không ghi nhận thay đổi, hoặc đường dẫn thư mục Delta bị sai. | Spark Connect phân giải đường dẫn trên container server. Kiểm tra xem biến `LAB28_DELTA_ROOT` đã trỏ đúng vào `/workspace/.lab28/delta` chưa. |
| Qdrant hiển thị 0 points | Chưa thực hiện bước nạp dữ liệu vector ban đầu. | Chạy lệnh `uv run lab28 index --source file` trước khi tiến hành demo câu hỏi `/ask`. |
| Cây Trace trên Jaeger bị đứt đoạn ở Kafka | Hàm `event_headers` chưa mã hóa đúng `traceparent` sang kiểu `bytes` hoặc gửi chuỗi rỗng. | Kiểm tra lại Phần A của `integration_tasks.py`. Đảm bảo `traceparent` được encode utf-8 và chỉ thêm vào headers khi có giá trị hợp lệ. |
| vLLM báo connection timeout | Máy không có GPU hoặc tunnel từ Kaggle bị ngắt kết nối. | Trong chế độ kiểm thử cơ bản, hệ thống cho phép vLLM ở trạng thái `degraded`. Nếu dùng Kaggle, hãy kiểm tra xem cell vLLM serve trên Kaggle còn đang chạy không và URL tunnel có bị đổi không. |
| Lệnh dọn dẹp môi trường an toàn | Cần dừng container mà không làm mất dữ liệu đã làm. | Dùng lệnh: `docker compose --env-file ports.template --profile full down --remove-orphans`. Tuyệt đối không dùng cờ `-v` hoặc `reset --yes` trong quá trình demo. |

---
*Tài liệu này được biên soạn đầy đủ làm kim chỉ nam thực hành và bảo vệ đồ án Lab 28 Track 2.*
