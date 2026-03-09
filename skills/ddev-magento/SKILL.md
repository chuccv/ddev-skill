---
name: ddev-magento
description: "Quản lý và vận hành Magento 2 trên DDEV với các dịch vụ MySQL, Redis, Elasticsearch, RabbitMQ và Varnish."
---

# Workflow

Kỹ năng này giúp bạn tương tác với các dự án Magento 2 chạy trên nền tảng DDEV một cách hiệu quả nhất.

## 1. Khởi động và Thiết lập (Startup & Setup)
*   **Khởi động môi trường:** Chạy `ddev start`.
*   **Cài đặt các gói Composer:** Chạy `ddev composer install`.
*   **Kiểm tra trạng thái các dịch vụ:** Chạy `ddev describe` hoặc `ddev status`.

## 2. Quy trình Nâng cấp và Triển khai (Deployment Workflow)
Khi có sự thay đổi về code hoặc database, thực hiện tuần tự:
1.  **Nâng cấp database:** `ddev magento setup:upgrade` (hoặc `ddev upgrade` nếu project có lệnh custom).
2.  **Biên dịch DI:** `ddev magento setup:di:compile`.
3.  **Triển khai Static Content:** `ddev magento setup:static-content:deploy -f`.
4.  **Xóa cache:** `ddev magento cache:flush`.

## 3. Quản lý Cache (Cache Management)
*   **Xóa toàn bộ cache:** `ddev magento cache:flush`.
*   **Xóa cache cụ thể:** `ddev magento cache:clean [type]`.
*   **Bật/Tắt cache:** `ddev magento cache:enable` hoặc `ddev magento cache:disable`.

## 4. Quản lý Dịch vụ (Services Management)
*   **Redis:** Truy cập redis-cli bằng `ddev redis-cli`.
*   **MySQL:** Truy cập MySQL bằng `ddev mysql`.
*   **RabbitMQ:** Xem giao diện quản lý qua URL được cung cấp bởi `ddev describe`.
*   **Elasticsearch/OpenSearch:** Kiểm tra trạng thái bằng `ddev exec curl -s http://elasticsearch:9200`.
*   **Varnish:** Kiểm tra cấu hình và trạng thái varnish.

## 5. Xử lý lỗi (Troubleshooting)
*   **Xem log web server:** `ddev logs -f web`.
*   **Xem log database:** `ddev logs -f db`.
*   **Khởi động lại môi trường:** `ddev restart`.
*   **Truy cập vào container:** `ddev ssh`.

## 6. Các lệnh tùy chỉnh (Custom Commands)
Nếu project có định nghĩa sẵn, bạn có thể dùng:
*   `ddev reindex`: Để reindex toàn bộ dữ liệu.
*   `ddev setup-install`: Để cài đặt lại Magento từ đầu.
*   `ddev setup-domain`: Để cấu hình domain cho project.
