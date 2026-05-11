# Kế hoạch Thực hiện Lab Observability (Day 23)

Kế hoạch này được thiết kế để giúp bạn học sâu về hệ thống giám sát (Observability Stack) cho các ứng dụng AI. Mỗi task được chia nhỏ để bạn có thể nắm vững từng khái niệm.

---

## Giai đoạn 0: Thiết lập Hạ tầng & Kiểm tra Sức khỏe (Pre-flight)
**Mục tiêu:** Đảm bảo máy tính của bạn chạy được 7 service trong Docker mà không gặp lỗi tài nguyên.

### Task 0.1: Cấu hình Môi trường
*   **Thao tác:**
    1.  Copy file `.env.example` thành `.env`.
    2.  Mở `.env` và điền `SLACK_WEBHOOK_URL` (nếu bạn có channel Slack để test alert).
*   **DoD Checklist:**
    - [ ] File `.env` tồn tại và có thông tin.
*   **Deliverables:** File `.env`.

### Task 0.2: Khởi động hệ thống
*   **Thao tác:**
    1.  Chạy `make setup` để cài đặt thư viện và kiểm tra Docker.
    2.  Chạy `make up` để kéo 7 image và khởi động các container.
    3.  Đợi khoảng 30s, chạy `make smoke` để xác nhận tất cả các service (App, Prometheus, Grafana, Loki, Jaeger, Alertmanager, OTel Collector) đều đang `Healthy`.
*   **DoD Checklist:**
    - [ ] 7 container đang chạy (`docker ps`).
    - [ ] Lệnh `make smoke` báo success.
    - [ ] File `setup-report.json` xuất hiện trong thư mục `submission/`.
*   **Deliverables:** `setup-report.json`.

---

## Giai đoạn 1: Instrumentation - Gắn thiết bị cho Code (`01-instrument-fastapi`)
**Mục tiêu:** Hiểu cách ứng dụng FastAPI xuất dữ liệu ra ngoài.

### Task 1.1: Kiểm tra Metrics
*   **Thao tác:**
    1.  Mở file `01-instrument-fastapi/app/instrumentation.py` để xem cách định nghĩa `Counter`, `Histogram`, `Gauge`.
    2.  Truy cập `http://localhost:8000/metrics` để xem dữ liệu thô.
*   **DoD Checklist:**
    - [ ] Tìm thấy `inference_requests_total` trong output của endpoint `/metrics`.
*   **Deliverables:** Ảnh chụp màn hình endpoint `/metrics`.

### Task 1.2: Mô phỏng tải (Load Simulation)
*   **Thao tác:**
    1.  Mở một terminal mới, chạy `make load`. Lệnh này sẽ dùng Locust để bắn request liên tục trong 60s.
    2.  Quan sát sự thay đổi của các chỉ số (đặc biệt là `inference_active_gauge`).
*   **DoD Checklist:**
    - [ ] Gauge `inference_active_gauge` tăng lên khi có tải và về 0 khi hết tải.
*   **Deliverables:** Screenshot ghi lại sự thay đổi của Gauge.

---

## Giai đoạn 2: Dashboard & Alerting (`02-prometheus-grafana`)
**Mục tiêu:** Quan sát hệ thống qua biểu đồ và thiết lập cảnh báo.

### Task 2.1: Khám phá Grafana
*   **Thao tác:**
    1.  Truy cập `http://localhost:3000` (User/Pass mặc định thường là admin/admin nếu không cấu hình khác).
    2.  Mở Dashboard `AI Service Overview`.
*   **DoD Checklist:**
    - [ ] Dashboard hiển thị đủ 6 panel: Request Rate, Latency, Error Rate, GPU Util, Token Throughput, In-flight Requests.
*   **Deliverables:** Ảnh chụp màn hình Overview Dashboard khi đang có traffic.

### Task 2.2: Test Cảnh báo (Alerting)
*   **Thao tác:**
    1.  Chạy `make alert`. Lệnh này sẽ làm sập app hoặc giả lập lỗi 503.
    2.  Vào Alertmanager (`http://localhost:9093`) để xem alert `ServiceDown`.
    3.  Kiểm tra Slack để xem tin nhắn "Firing" và sau đó là "Resolved".
*   **DoD Checklist:**
    - [ ] Nhận được thông báo Slack.
*   **Deliverables:** Ảnh chụp màn hình tin nhắn Slack (Fire & Resolve).

---

## Giai đoạn 3: Tracing & Logs - Soi vết Request (`03-tracing-and-logs`)
**Mục tiêu:** Theo dõi hành trình của một request đi qua các hàm bên trong.

### Task 3.1: Phân tích Trace trong Jaeger
*   **Thao tác:**
    1.  Truy cập Jaeger UI (`http://localhost:16686`).
    2.  Tìm trace cho service `inference-api` và operation `predict`.
    3.  Kiểm tra xem một trace có đủ các sub-spans: `embed-text`, `vector-search`, `generate-tokens` không.
*   **DoD Checklist:**
    - [ ] Một trace cho thấy rõ thời gian xử lý của từng bước nhỏ bên trong hàm predict.
*   **Deliverables:** Ảnh chụp màn hình Jaeger Trace UI.

---

## Giai đoạn 4: Drift Detection - Giám sát chất lượng Model (`04-drift-detection`)
**Mục tiêu:** Phát hiện khi dữ liệu thực tế bị lệch so với dữ liệu huấn luyện.

### Task 4.1: Chạy script phát hiện Drift
*   **Thao tác:**
    1.  Chạy `make drift`.
    2.  Kiểm tra file `04-drift-detection/reports/drift-summary.json`.
*   **DoD Checklist:**
    - [ ] File `drift-summary.json` báo "yes" cho ít nhất một feature (thường là prompt_length hoặc response_quality).
*   **Deliverables:** File `drift-summary.json`.

---

## Giai đoạn 5: Tích hợp & Viết Báo cáo (Reflection)
**Mục tiêu:** Tổng hợp kiến thức và hoàn thiện bài lab.

### Task 5.1: Kiểm tra Dashboard Tổng hợp
*   **Thao tác:**
    1.  Mở Dashboard `Cross-Day Stack` trong Grafana.
    2.  Đảm bảo ít nhất một chỉ số từ các ngày trước (như Day 19 Qdrant) được hiển thị (dù là dữ liệu stub).
*   **DoD Checklist:**
    - [ ] Dashboard tích hợp hiển thị được dữ liệu từ >1 nguồn.

### Task 5.2: Hoàn thiện REFLECTION.md
*   **Thao tác:**
    1.  Mở file `submission/REFLECTION.md`.
    2.  Trả lời đầy đủ các câu hỏi trong 5 phần. Đặc biệt là phần rút ra kinh nghiệm sau bài lab.
*   **DoD Checklist:**
    - [ ] File reflection không được để trống, các screenshot được dẫn link đúng.

### Task 5.3: Verify cuối cùng
*   **Thao tác:**
    1.  Chạy `make verify`. Nếu kết quả là `Exit code 0`, bạn đã sẵn sàng nộp bài.
*   **DoD Checklist:**
    - [ ] Lệnh `make verify` trả về thành công.
*   **Deliverables:** Toàn bộ thư mục `submission/`.
