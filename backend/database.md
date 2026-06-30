# Database (Cơ sở dữ liệu quan hệ) — SQL, Thiết kế & Hiệu năng

> Báo cáo tìm hiểu các khái niệm nền tảng về cơ sở dữ liệu quan hệ (relational database) với SQL, lấy MySQL và PostgreSQL làm ví dụ chính.

## 1. SQL Basic

**SQL là gì?**

SQL (Structured Query Language — ngôn ngữ truy vấn có cấu trúc) là ngôn ngữ tiêu chuẩn để **giao tiếp với cơ sở dữ liệu quan hệ**. Dữ liệu trong CSDL quan hệ được tổ chức thành các **bảng (table)** gồm **hàng (row/record)** và **cột (column/field)**. SQL cho phép ta tạo bảng, thêm/sửa/xóa dữ liệu và truy vấn (lấy ra) dữ liệu mình cần.

**SQL dùng để làm gì?**

Người ta thường chia các lệnh SQL thành các nhóm:

- **DDL (Data Definition Language)** — định nghĩa cấu trúc: `CREATE`, `ALTER`, `DROP`.
- **DML (Data Manipulation Language)** — thao tác dữ liệu: `INSERT`, `UPDATE`, `DELETE`.
- **DQL (Data Query Language)** — truy vấn dữ liệu: `SELECT`.
- **DCL / TCL** — phân quyền (`GRANT`, `REVOKE`) và quản lý giao dịch (`COMMIT`, `ROLLBACK`).

**Các câu lệnh cơ bản nhất:**

```sql
-- Tạo bảng
CREATE TABLE users (
    id    INT PRIMARY KEY AUTO_INCREMENT,
    name  VARCHAR(100) NOT NULL,
    email VARCHAR(150),
    age   INT
);

-- Thêm dữ liệu
INSERT INTO users (name, email, age) VALUES ('An', 'an@example.com', 25);

-- Truy vấn dữ liệu
SELECT id, name, age          -- chọn cột
FROM users                    -- từ bảng nào
WHERE age >= 18               -- điều kiện lọc
ORDER BY age DESC             -- sắp xếp
LIMIT 10;                     -- giới hạn số dòng

-- Cập nhật
UPDATE users SET age = 26 WHERE id = 1;

-- Xóa
DELETE FROM users WHERE id = 1;
```

**Thứ tự thực thi (execution order) của một câu `SELECT`** khác với thứ tự ta viết. Ta viết `SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...`, nhưng CSDL xử lý theo thứ tự logic khác. `SELECT` tuy viết đầu tiên nhưng lại được xử lý gần **cuối**:

```
1. FROM      →  Lấy dữ liệu từ bảng nào (vd: users)
2. WHERE     →  Lọc bớt từng dòng (vd: giữ age >= 18)
3. GROUP BY  →  Gom các dòng còn lại thành nhóm
4. HAVING    →  Lọc tiếp trên từng nhóm
5. SELECT    →  Mới chọn ra cột nào để hiển thị (alias ở đây mới "ra đời")
6. ORDER BY  →  Sắp xếp kết quả
7. LIMIT     →  Cắt lấy N dòng đầu
```

**Vì sao điều này quan trọng? — Câu chuyện về alias.** Alias (bí danh) là tên rút gọn ta đặt bằng `AS` trong phần `SELECT`. Vì alias chỉ "ra đời" ở **bước 5 (`SELECT`)**, nên những mệnh đề **chạy trước** `SELECT` sẽ không nhìn thấy alias, còn những mệnh đề **chạy sau** thì thấy:

```sql
-- ❌ LỖI: WHERE (bước 2) chạy TRƯỚC SELECT (bước 5), chưa biết "double_age" là gì
SELECT age * 2 AS double_age
FROM users
WHERE double_age > 30;

-- ✅ ĐÚNG: dùng lại cột gốc "age" (đã có sẵn từ bước FROM)
SELECT age * 2 AS double_age
FROM users
WHERE age * 2 > 30;

-- ✅ ĐÚNG: ORDER BY (bước 6) chạy SAU SELECT nên dùng được alias
SELECT age * 2 AS double_age
FROM users
ORDER BY double_age;
```

| Mệnh đề | Chạy trước/sau `SELECT` | Dùng được alias? |
|---|---|---|
| `WHERE` | Trước | ❌ Không |
| `GROUP BY` | Trước | ❌ Không (đa số DB) |
| `HAVING` | Trước | ⚠️ Tùy DB (MySQL cho, chuẩn SQL thì không) |
| `ORDER BY` | Sau | ✅ Có |

> **Câu chốt:** alias sinh ra ở `SELECT`, nên chỉ mệnh đề chạy **sau** `SELECT` (như `ORDER BY`) mới dùng được; `WHERE` chạy **trước** nên phải viết lại biểu thức/cột gốc.

## 2. JOIN

**JOIN là gì?**

JOIN là phép **ghép dữ liệu từ hai (hoặc nhiều) bảng** lại với nhau dựa trên một điều kiện liên kết — thường là khóa ngoại (foreign key) khớp với khóa chính (primary key) của bảng kia. Trong CSDL quan hệ, dữ liệu được tách ra nhiều bảng để tránh trùng lặp (chuẩn hóa), nên khi cần thông tin đầy đủ ta phải JOIN chúng lại.

**Các loại JOIN dùng để làm gì?**

Giả sử có bảng `users` và bảng `orders` (mỗi đơn hàng thuộc về một user qua cột `orders.user_id`):

- **INNER JOIN** — chỉ lấy các dòng **khớp ở cả hai bảng**. User không có đơn nào, hoặc đơn không gắn user nào, đều bị loại.
- **LEFT JOIN (LEFT OUTER)** — lấy **tất cả dòng bảng trái** (`users`), cộng thêm dữ liệu khớp ở bảng phải; user nào không có đơn thì cột bên `orders` là `NULL`. Dùng khi muốn "tất cả user, kèm đơn hàng nếu có".
- **RIGHT JOIN (RIGHT OUTER)** — ngược lại LEFT: giữ tất cả dòng bảng phải.
- **FULL OUTER JOIN** — lấy tất cả dòng của **cả hai bảng**, không khớp thì điền `NULL`. (MySQL không hỗ trợ trực tiếp, phải dùng `UNION` của LEFT và RIGHT; PostgreSQL hỗ trợ sẵn.)
- **CROSS JOIN** — tích Descartes: ghép **mọi dòng bảng này với mọi dòng bảng kia**.

```sql
-- Lấy tên user kèm tổng tiền mỗi đơn (chỉ user có đơn)
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON o.user_id = u.id;

-- Lấy tất cả user, kèm đơn nếu có (user chưa mua vẫn hiện)
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON o.user_id = u.id;
```

> **Mẹo:** Một câu hỏi phỏng vấn kinh điển là dùng `LEFT JOIN ... WHERE o.id IS NULL` để tìm các user **chưa từng đặt đơn** (các dòng không khớp ở bảng phải).

## 3. Aggregate (Hàm tổng hợp & gom nhóm)

**Aggregate là gì?**

Hàm tổng hợp (aggregate function) là hàm nhận **nhiều dòng** và trả về **một giá trị duy nhất** tóm tắt chúng. Các hàm phổ biến: `COUNT()` (đếm), `SUM()` (tổng), `AVG()` (trung bình), `MIN()`, `MAX()`.

**Dùng để làm gì?**

Dùng để tính toán thống kê trên dữ liệu — thường kết hợp với `GROUP BY` để tính cho **từng nhóm** thay vì toàn bảng.

```sql
-- Đếm tổng số user
SELECT COUNT(*) FROM users;

-- Tổng doanh thu và số đơn theo từng user
SELECT user_id,
       COUNT(*)   AS so_don,
       SUM(total) AS doanh_thu
FROM orders
GROUP BY user_id
HAVING SUM(total) > 1000000   -- lọc SAU khi gom nhóm
ORDER BY doanh_thu DESC;
```

**`WHERE` và `HAVING` khác nhau ở đâu?** Đây là điểm hay nhầm:

- `WHERE` lọc **từng dòng trước khi gom nhóm** — không dùng được với hàm aggregate.
- `HAVING` lọc **kết quả sau khi đã gom nhóm** — dùng được với hàm aggregate (ví dụ `HAVING SUM(total) > 1000000`).

Lưu ý: mọi cột xuất hiện ở `SELECT` mà không nằm trong hàm aggregate thì **bắt buộc phải có trong `GROUP BY`** (PostgreSQL yêu cầu chặt; MySQL có thể nới lỏng nhưng nên tuân thủ để tránh kết quả khó đoán).

## 4. Database Design (Thiết kế cơ sở dữ liệu)

**Database Design là gì?**

Là quá trình **quyết định cấu trúc** của CSDL: có những bảng nào, mỗi bảng có cột gì, kiểu dữ liệu ra sao, và các bảng liên kết với nhau như thế nào. Thiết kế tốt giúp dữ liệu **nhất quán, ít trùng lặp, dễ mở rộng và truy vấn hiệu quả**.

**Các khái niệm cốt lõi:**

- **Khóa chính (Primary Key):** cột (hoặc tổ hợp cột) định danh **duy nhất** mỗi dòng. Không trùng, không NULL.
- **Khóa ngoại (Foreign Key):** cột trỏ tới khóa chính của bảng khác, tạo nên **quan hệ** giữa các bảng và đảm bảo toàn vẹn dữ liệu.
- **Quan hệ (relationship):**
  - **1–1 (one-to-one):** ví dụ `users` ↔ `user_profiles`.
  - **1–N (one-to-many):** một user có nhiều đơn hàng — phổ biến nhất.
  - **N–N (many-to-many):** một sinh viên học nhiều môn, một môn có nhiều sinh viên — cần **bảng trung gian (pivot/junction table)** như `enrollments(student_id, course_id)`.

**Chuẩn hóa (Normalization) dùng để làm gì?**

Chuẩn hóa là quá trình **tách dữ liệu thành nhiều bảng để loại bỏ trùng lặp** và tránh các bất thường khi thêm/sửa/xóa. Các dạng chuẩn thường gặp:

- **1NF:** mỗi ô chỉ chứa một giá trị nguyên tử (không nhồi danh sách vào một cột).
- **2NF:** mọi cột phụ thuộc vào **toàn bộ** khóa chính (loại bỏ phụ thuộc một phần).
- **3NF:** loại bỏ phụ thuộc bắc cầu (cột không khóa không phụ thuộc vào cột không khóa khác).

> **Đánh đổi:** Chuẩn hóa giảm trùng lặp nhưng làm tăng số JOIN khi truy vấn. Trong các hệ thống đọc nhiều / cần tốc độ (như báo cáo, analytics), đôi khi người ta cố ý **phi chuẩn hóa (denormalization)** — chấp nhận lặp dữ liệu để giảm JOIN, đổi tốc độ đọc lấy độ phức tạp khi ghi.

## 5. Migrations (Di trú cấu trúc CSDL)

**Migration là gì?**

Migration là cách **quản lý phiên bản (version control) cho cấu trúc cơ sở dữ liệu** bằng code. Thay vì sửa schema thủ công bằng tay trên từng máy, ta viết các "file migration" mô tả thay đổi (tạo bảng, thêm cột, thêm index...). Mỗi migration giống như một "commit" cho database.

**Dùng để làm gì?**

- **Đồng bộ schema** giữa các thành viên trong nhóm và giữa các môi trường (dev / staging / production): ai cũng chạy cùng bộ migration thì có cùng cấu trúc DB.
- **Lịch sử thay đổi rõ ràng:** biết schema đã tiến hóa qua các bước nào.
- **Quay lui (rollback):** mỗi migration thường có hàm `up` (áp dụng) và `down` (hoàn tác) để lùi lại khi cần.

Ví dụ migration trong Laravel (PHP):

```php
public function up(): void {
    Schema::create('orders', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained();
        $table->decimal('total', 10, 2);
        $table->timestamps();
    });
}

public function down(): void {
    Schema::dropIfExists('orders');
}
```

Chạy `php artisan migrate` để áp dụng, `php artisan migrate:rollback` để lùi. (Các framework khác có công cụ tương tự: Rails có Active Record Migrations, Django có `makemigrations`/`migrate`, Node có Knex/Sequelize...). Hệ thống lưu lại các migration đã chạy trong một bảng riêng (ví dụ `migrations`) để không chạy lặp.

## 6. Constraint (Ràng buộc)

**Constraint là gì?**

Constraint là các **quy tắc ràng buộc** áp lên cột/bảng để đảm bảo dữ liệu luôn **hợp lệ và toàn vẹn**. CSDL sẽ **từ chối** mọi thao tác làm vi phạm ràng buộc — đây là tuyến phòng thủ ở tầng dữ liệu, không phụ thuộc vào code ứng dụng.

**Các loại constraint và để làm gì:**

| Constraint | Ý nghĩa |
|---|---|
| `PRIMARY KEY` | Định danh duy nhất mỗi dòng (vừa `UNIQUE` vừa `NOT NULL`). |
| `FOREIGN KEY` | Giá trị phải tồn tại ở bảng được tham chiếu → đảm bảo toàn vẹn tham chiếu (referential integrity). |
| `UNIQUE` | Giá trị không được trùng (ví dụ `email`). |
| `NOT NULL` | Cột bắt buộc có giá trị. |
| `CHECK` | Giá trị phải thỏa điều kiện (ví dụ `age >= 0`). |
| `DEFAULT` | Gán giá trị mặc định khi không truyền vào. |

```sql
CREATE TABLE orders (
    id      INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    total   DECIMAL(10,2) CHECK (total >= 0),
    status  VARCHAR(20) DEFAULT 'pending',
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

> Với `FOREIGN KEY` có thể đặt hành vi khi bản ghi cha bị xóa/sửa: `ON DELETE CASCADE` (xóa luôn con), `ON DELETE SET NULL`, hoặc `RESTRICT` (chặn xóa nếu còn con). Đặt ràng buộc ngay ở DB an toàn hơn là chỉ kiểm tra ở code, vì code có thể có bug nhưng DB luôn chặn.

## 7. Index (Chỉ mục) ⭐

**Index là gì?**

Index (chỉ mục) là một **cấu trúc dữ liệu phụ** mà CSDL xây dựng trên một (hoặc vài) cột, giúp **tìm kiếm dữ liệu nhanh hơn** mà không phải quét toàn bộ bảng. Hình dung giống như **mục lục của một cuốn sách**: thay vì lật từng trang để tìm một từ, ta tra mục lục (đã sắp xếp) để nhảy thẳng tới trang chứa từ đó.

### Tại sao index làm query nhanh hơn?

Không có index, để tìm `WHERE email = 'an@example.com'` CSDL phải làm **full table scan** — duyệt **từng dòng** trong bảng để so khớp, độ phức tạp **O(n)**. Với bảng hàng triệu dòng, điều này rất chậm.

Index phổ biến nhất dùng cấu trúc **B-Tree (cụ thể là B+Tree)** — một cây cân bằng, dữ liệu được **sắp xếp sẵn**. Nhờ đó việc tìm kiếm chỉ mất khoảng **O(log n)** bước (đi từ gốc xuống lá), thay vì quét hết. Ví dụ: bảng 1 triệu dòng, full scan duyệt ~1.000.000 dòng, còn tra B-Tree chỉ mất ~20 bước. Đó là lý do index làm query nhanh hơn **rất nhiều bậc**.

Quan trọng: index chỉ phát huy với cấu trúc truy vấn phù hợp. B-Tree giúp được cho `=`, khoảng (`<`, `>`, `BETWEEN`), `ORDER BY`, và tiền tố `LIKE 'abc%'`. Nhưng **không** giúp được với `LIKE '%abc'` (bắt đầu bằng ký tự đại diện) hay khi ta bọc cột trong hàm như `WHERE YEAR(created_at) = 2024` (làm mất tính sắp xếp của index).

### Có những loại index nào? (MySQL & PostgreSQL)

**MySQL (InnoDB):**

- **B-Tree index** — loại mặc định, dùng cho hầu hết các trường hợp (so sánh bằng, khoảng, sắp xếp). Gần như mọi index trong InnoDB đều là B+Tree.
- **Clustered index (chỉ mục gom cụm)** — chính là **PRIMARY KEY**. InnoDB lưu **dữ liệu thật của dòng ngay tại lá của cây khóa chính**, nên tra theo khóa chính dẫn thẳng tới dữ liệu (rất nhanh). Mỗi bảng chỉ có **một** clustered index.
- **Secondary index (chỉ mục phụ)** — mọi index khác ngoài clustered. Lá của nó chứa giá trị **khóa chính**, nên tìm qua secondary index rồi phải tra tiếp clustered index để lấy dòng (gọi là "bookmark lookup").
- **Full-text index** — cho tìm kiếm văn bản tự nhiên với `MATCH() AGAINST()`.
- **Spatial index (R-Tree)** — cho dữ liệu không gian/địa lý (tọa độ, bản đồ).
- **Hash index** — InnoDB có "adaptive hash index" tự động bên trong; engine `MEMORY` cho phép chỉ định hash tường minh (chỉ phục vụ so sánh `=`).

**PostgreSQL** đa dạng hơn:

- **B-Tree** — mặc định; cho `=`, khoảng, `ORDER BY`, `IS NULL`.
- **Hash** — chỉ cho so sánh bằng (`=`); tra nhanh nhưng không hỗ trợ khoảng.
- **GiST** — dữ liệu hình học, không gian, và tìm "lân cận gần nhất" (nearest-neighbor).
- **SP-GiST** — cho dữ liệu phân vùng tự nhiên, cây không cân bằng (quadtree, k-d tree, định tuyến IP...).
- **GIN (inverted index)** — cho cột chứa **nhiều giá trị**: mảng (array), `jsonb`, full-text search.
- **BRIN (Block Range Index)** — cho bảng **rất lớn** mà dữ liệu sắp theo thứ tự vật lý (ví dụ log theo thời gian). BRIN cực nhỏ và rẻ vì chỉ lưu tóm tắt min/max theo từng khối.

### Khi nào dùng index nào?

- **B-Tree** — mặc định, dùng cho cột hay xuất hiện trong `WHERE`, `JOIN`, `ORDER BY` với so sánh bằng hoặc khoảng. ~90% trường hợp.
- **Hash (Postgres)** — chỉ khi truy vấn **luôn là `=`** và không cần khoảng/sắp xếp.
- **GIN (Postgres)** — khi index cột `jsonb`, mảng, hoặc full-text.
- **BRIN (Postgres)** — bảng khổng lồ ghi tuần tự theo thời gian (logs, sự kiện) mà B-Tree sẽ quá to.
- **Full-text / Spatial** — khi tìm kiếm văn bản hoặc dữ liệu địa lý.
- **Composite index (index nhiều cột)** — khi truy vấn lọc theo nhiều cột cùng lúc, ví dụ `INDEX(user_id, status)`. Lưu ý **thứ tự cột quan trọng** (quy tắc "leftmost prefix"): index `(a, b)` giúp được cho `WHERE a=...` và `WHERE a=... AND b=...`, nhưng **không** giúp cho `WHERE b=...` đơn lẻ.
- **Unique index** — vừa tăng tốc tra cứu vừa đảm bảo không trùng (ràng buộc `UNIQUE` thường tự tạo index).
- **Covering index** — index chứa luôn mọi cột mà query cần, nên DB lấy dữ liệu thẳng từ index, **không cần đọc bảng** (index-only scan) → rất nhanh.

### Đánh index nhiều có ảnh hưởng gì không?

**Có — index không miễn phí.** Đánh quá nhiều index gây tác hại:

1. **Ghi chậm hơn (INSERT/UPDATE/DELETE):** mỗi khi dữ liệu thay đổi, **tất cả index liên quan đều phải cập nhật theo**. Bảng càng nhiều index, thao tác ghi càng tốn kém.
2. **Tốn dung lượng đĩa:** mỗi index là một cấu trúc dữ liệu riêng được lưu trên đĩa; nhiều index = chiếm thêm nhiều dung lượng (và RAM khi nạp vào bộ nhớ đệm).
3. **Bộ tối ưu (query planner) phải làm việc nhiều hơn:** có quá nhiều index khiến CSDL mất công cân nhắc chọn index nào, đôi khi chọn nhầm index kém tối ưu.
4. **Index thừa/trùng lặp:** ví dụ đã có `(a, b)` thì thêm `(a)` là dư (vì leftmost prefix đã bao); những index không bao giờ được dùng chỉ tổ tốn chi phí bảo trì.

> **Nguyên tắc:** Index là sự đánh đổi **đọc nhanh hơn ↔ ghi chậm hơn + tốn chỗ**. Hãy đánh index cho các cột **thật sự hay được lọc/join/sắp xếp**, đo bằng `EXPLAIN` / `EXPLAIN ANALYZE` xem query có dùng index không, và **xóa các index không dùng tới**. Đừng đánh index theo kiểu "cho chắc" trên mọi cột.

## 8. N+1 Problem (Vấn đề N+1)

**N+1 Problem là gì?**

N+1 là một **lỗi hiệu năng phổ biến** khi dùng ORM (Object-Relational Mapping như Eloquent của Laravel, Active Record của Rails, Hibernate...). Nó xảy ra khi để lấy một danh sách và dữ liệu liên quan, chương trình chạy **1 truy vấn lấy danh sách (N bản ghi), rồi chạy thêm N truy vấn — mỗi bản ghi một truy vấn** để lấy dữ liệu liên quan. Tổng cộng **1 + N** câu query, thay vì chỉ vài câu.

**Ví dụ (lazy loading gây N+1):**

```php
$posts = Post::all();           // 1 query: SELECT * FROM posts  (giả sử 100 bài)
foreach ($posts as $post) {
    echo $post->author->name;   // mỗi vòng lặp +1 query lấy author → 100 query
}
// Tổng cộng: 1 + 100 = 101 query  ❌
```

Với 100 bài viết, ta vô tình bắn **101 câu query** xuống DB — chậm và quá tải, dù người viết tưởng chỉ "lấy danh sách bài viết".

**Cách khắc phục: Eager Loading**

Nạp sẵn dữ liệu liên quan bằng **một** (hoặc vài) truy vấn dùng `IN (...)` thay vì N truy vấn lẻ:

```php
$posts = Post::with('author')->get();
// 2 query:
//   SELECT * FROM posts
//   SELECT * FROM authors WHERE id IN (1, 2, 3, ...)   ✅
```

Từ **101 query xuống còn 2 query**. Các ORM khác đều có cơ chế tương tự (Rails: `includes`, Django: `select_related` / `prefetch_related`, Hibernate: `JOIN FETCH`). Bài học: cẩn thận khi truy cập quan hệ **bên trong vòng lặp**; hãy eager-load trước. Dùng công cụ như Laravel Debugbar / query log để phát hiện N+1.

---

## Tóm tắt nhanh

| Chủ đề | Một câu cốt lõi |
|---|---|
| SQL Basic | Ngôn ngữ truy vấn CSDL quan hệ: CRUD + truy vấn `SELECT`. |
| JOIN | Ghép dữ liệu nhiều bảng theo khóa liên kết (INNER/LEFT/RIGHT/FULL). |
| Aggregate | Tổng hợp nhiều dòng thành một giá trị (`COUNT/SUM/AVG`), kết hợp `GROUP BY`/`HAVING`. |
| Database Design | Thiết kế bảng, quan hệ, chuẩn hóa để dữ liệu nhất quán, ít trùng. |
| Migrations | Quản lý phiên bản schema bằng code, đồng bộ & rollback được. |
| Constraint | Ràng buộc tầng DB đảm bảo toàn vẹn (PK, FK, UNIQUE, NOT NULL, CHECK). |
| Index | Cấu trúc phụ (B-Tree...) giúp tìm O(log n) thay vì quét O(n); đánh đổi ghi chậm hơn. |
| N+1 Problem | Lỗi 1+N query của ORM; khắc phục bằng eager loading. |

---
### Nguồn tham khảo
<!-- Liệt kê các link/tài liệu đã đọc -->
- [PostgreSQL Docs — 11.2. Index Types](https://www.postgresql.org/docs/current/indexes-types.html)
- [MySQL Reference Manual — Clustered and Secondary Indexes](https://dev.mysql.com/doc/refman/9.7/en/innodb-index-types.html)
- [Percona — Understanding MySQL Indexes: Types, Benefits, and Best Practices](https://www.percona.com/blog/understanding-mysql-indexes-types-best-practices/)
- roadmap.sh — Backend Developer Roadmap
