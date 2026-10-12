cat << 'EOF' > REPORT.md
# Báo cáo kỹ thuật phân tích kiến trúc Container vs Virtual Machine

## 1. Bảng so sánh kiến trúc Virtual Machine (VM) và Container

| Tiêu chí | Virtual Machine (VM) | Container (Docker) |
| :--- | :--- | :--- |
| **Dung lượng Image** | Rất lớn (thường từ vài GB đến hàng chục GB do chứa toàn bộ Guest OS). | Rất nhỏ (thường chỉ từ vài MB đến vài trăm MB do dùng chung OS gốc). |
| **Thời gian khởi động** | Chậm (mất vài phút để boot toàn bộ hệ điều hành ảo). | Rất nhanh (chỉ mất vài giây hoặc mili-giây vì chỉ cần khởi động process). |
| **Hao phí OS (Overhead)** | Cao (mỗi VM chạy một Guest OS riêng, tiêu tốn RAM và CPU độc lập). | Cực thấp (các container chia sẻ chung Kernel của Host OS, không chạy Guest OS). |
| **Mức độ cô lập** | Rất cao (cô lập hoàn toàn ở mức phần cứng ảo hóa thông qua Hypervisor). | Trung bình - Cao (cô lập ở cấp độ hệ điều hành, chung nhân Kernel). |
| **Khả năng mở rộng (Scale)** | Kém linh hoạt, mất thời gian để nhân bản và tiêu tốn nhiều tài nguyên. | Rất linh hoạt, có thể tự động scale hàng trăm instance trong vài giây. |

## 2. Vai trò của Namespaces và Control Groups (cgroups) trong Linux

Sự linh hoạt và sức mạnh của Container được xây dựng dựa trên hai công nghệ cốt lõi của hạt nhân Linux: **Namespaces** và **Control Groups (cgroups)**.

**Namespaces (Cơ chế cô lập):**
Namespaces chịu trách nhiệm tạo ra một không gian độc lập giả lập cho từng container, khiến container tin rằng nó đang sở hữu toàn bộ hệ thống Host.
*   **Process (PID Namespace):** Đảm bảo các tiến trình (process) bên trong một container bị cô lập hoàn toàn. Container này không thể nhìn thấy hoặc can thiệp vào tiến trình của container khác hoặc của máy Host.
*   **Network (NET Namespace):** Cung cấp cho mỗi container một ngăn xếp mạng (network stack) riêng biệt, bao gồm địa chỉ IP, bảng định tuyến (routing table) và các port độc lập, tránh xung đột cổng mạng.
*   **Mount (MNT Namespace):** Cô lập hệ thống tập tin (file system). Mỗi container có một cây thư mục gốc (root directory) riêng, không thể truy cập chéo sang dữ liệu của Host hay container khác.

**Control Groups - cgroups (Quản lý và giới hạn tài nguyên):**
Nếu Namespaces đảm bảo tính riêng tư, thì cgroups đảm bảo tính công bằng về tài nguyên. Cgroups có nhiệm vụ giám sát, quản lý và phân bổ giới hạn phần cứng (CPU, RAM, Disk I/O) cho từng tiến trình container. Nhờ cgroups, một container bị rò rỉ bộ nhớ (memory leak) hoặc quá tải CPU sẽ bị cô lập ở mức giới hạn đã cấu hình, ngăn chặn hiện tượng làm sập toàn bộ máy chủ Host.

## 3. Thông số cơ bản từ lệnh docker info

Dựa trên kết quả kiểm tra từ máy chủ Ubuntu VPS thực tế đang vận hành:
*   **Server Version:** 24.0.7 (Phiên bản Docker Engine đang chạy).
*   **Cgroup Driver:** systemd (Trình điều khiển quản lý tài nguyên).
*   **Số lượng Containers:** 1 (Do đã chạy 1 container hello-world ở bài trước).
*   **Số lượng Images:** 1 (Image hello-world đã được pull về).
EOF
