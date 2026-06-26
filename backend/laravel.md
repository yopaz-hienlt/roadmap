# Laravel — Vòng đời một Request (Request Lifecycle)

> Sơ đồ và ghi chú về cách Laravel xử lý một yêu cầu từ trình duyệt.

## Sơ đồ tuần tự (Sequence Diagram)

```mermaid
sequenceDiagram
    participant B as Trình duyệt
    participant I as public/index.php
    participant K as HTTP Kernel
    participant MW as Middleware
    participant R as Router
    participant C as Controller
    participant Mo as Model (Eloquent)
    participant DB as Database

    B->>I: HTTP request
    Note over I: Nạp autoloader + khởi tạo app (bootstrap)
    I->>K: Chuyển request cho HTTP Kernel
    K->>MW: Đi qua các Middleware (auth, csrf...)
    MW->>R: Request hợp lệ → tới Router
    R->>C: Khớp route → gọi Controller tương ứng
    C->>Mo: Gọi Model để lấy/lưu dữ liệu
    Mo->>DB: Truy vấn SQL (qua Eloquent ORM)
    DB-->>Mo: Trả về dữ liệu
    Mo-->>C: Trả dữ liệu cho Controller
    Note over C: Xử lý logic, dựng View hoặc JSON
    C-->>R: Trả về Response
    R-->>MW: Response đi ngược qua Middleware
    MW-->>K: 
    K-->>I: 
    I-->>B: Gửi HTTP response (HTML/JSON)
    Note over B: Render và hiển thị trang
```

## Giải thích các bước

1. **public/index.php** — điểm vào (entry point) duy nhất của ứng dụng. Mọi request đều đi qua đây. Nó nạp **Composer autoloader** và khởi tạo (bootstrap) ứng dụng Laravel.
2. **HTTP Kernel** — "bộ não" tiếp nhận request. Nó nạp các cấu hình, đăng ký **service provider**, rồi đẩy request qua lớp middleware.
3. **Middleware** — các "lớp lọc" mà request phải đi qua trước khi tới logic chính, ví dụ: kiểm tra đăng nhập (auth), chống CSRF, ghi log... Request không hợp lệ sẽ bị chặn tại đây.
4. **Router** — so khớp URL của request với các route đã định nghĩa (trong `routes/web.php` hoặc `routes/api.php`), rồi chuyển tới Controller (hoặc closure) tương ứng.
5. **Controller** — chứa logic xử lý: nhận dữ liệu, gọi Model, quyết định trả về gì.
6. **Model (Eloquent ORM)** — đại diện cho bảng trong cơ sở dữ liệu. Thay vì viết SQL thủ công, ta thao tác với dữ liệu qua các đối tượng PHP; Eloquent tự sinh câu lệnh SQL.
7. **Database** — nơi lưu trữ dữ liệu thực sự (MySQL, PostgreSQL...).
8. **Response** — Controller dựng kết quả thành **View (HTML)** hoặc **JSON** (cho API), rồi response đi **ngược lại** qua middleware → Kernel → index.php → trả về trình duyệt.

> **Tóm tắt:** Request → index.php → Kernel → Middleware → Router → Controller → Model → Database, rồi đi ngược lại để trả Response về cho trình duyệt. Laravel dùng mô hình **MVC** (Model – View – Controller) để tách bạch dữ liệu, giao diện và logic điều khiển.

---
### Nguồn tham khảo
<!-- Liệt kê các link/tài liệu đã đọc -->
