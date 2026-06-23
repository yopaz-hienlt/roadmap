# LEMP Stack — Hướng dẫn cài đặt & Ghi chú

> Báo cáo các bước cài đặt LEMP stack (yêu cầu: 500 chữ trở lên).
> Tự viết nội dung vào từng phần bên dưới.

## A. Các bước cài đặt LEMP Stack

### 1. Chuẩn bị server / cập nhật hệ thống
<!-- sudo apt update && sudo apt upgrade ... -->


### 2. Cài Nginx
<!-- Lệnh cài, kiểm tra service, mở firewall -->


### 3. Cài MySQL / MariaDB
<!-- Lệnh cài, chạy mysql_secure_installation -->


### 4. Cài PHP (PHP-FPM)
<!-- Cài php-fpm, các extension cần thiết -->


### 5. Cấu hình Nginx chạy PHP
<!-- Tạo server block, trỏ tới php-fpm sock -->


### 6. Kiểm tra hoạt động
<!-- Tạo file info.php, test trên trình duyệt -->


---

## B. Ghi chú giải thích các khái niệm

### LEMP stack là gì?
<!-- Linux + Nginx (Engine-X) + MySQL/MariaDB + PHP -->


### Tại sao cần tới Nginx?
<!-- Vai trò web server / reverse proxy, phục vụ static, load balancing... -->


### Nginx khác gì Apache?
<!-- Kiến trúc event-driven vs process/thread, hiệu năng, cách xử lý static/dynamic, .htaccess -->


### HTTP/1 khác gì HTTP/2?
<!-- Multiplexing, header compression, server push, 1 kết nối vs nhiều kết nối -->


### VPS là gì, khác gì server vật lý?
<!-- VPS = máy ảo chia từ server vật lý; so sánh chi phí, tài nguyên, khả năng mở rộng -->


---
### Nguồn tham khảo
<!-- Liệt kê các link/tài liệu đã đọc -->
