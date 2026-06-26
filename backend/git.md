# Git — Lệnh thường dùng & Quy trình làm việc (Git Flow)

> Báo cáo tìm hiểu (yêu cầu: 500 chữ trở lên).

## 1. Git là gì

**Git** là hệ thống quản lý phiên bản phân tán (distributed version control system) — công cụ giúp lưu lại lịch sử thay đổi của mã nguồn theo thời gian. Nhờ Git, nhiều người có thể cùng làm việc trên một dự án mà không ghi đè lên nhau, đồng thời có thể quay lại bất kỳ phiên bản nào trong quá khứ khi cần. "Phân tán" nghĩa là mỗi máy đều giữ một bản sao đầy đủ của repo (gồm toàn bộ lịch sử), không phụ thuộc hoàn toàn vào máy chủ trung tâm.

Một dự án Git gồm ba "vùng" chính: **working directory** (thư mục làm việc, nơi ta sửa file), **staging area** (vùng chờ, nơi đánh dấu thay đổi sẽ được lưu) và **repository** (kho commit, nơi lưu các mốc lịch sử). Quy trình cơ bản là: sửa file → `add` vào staging → `commit` thành một mốc → `push` lên remote (GitHub).

## 2. Các nhóm lệnh thường dùng

### Khởi tạo & cấu hình
```bash
git init                 # tạo repo mới trong thư mục hiện tại
git clone <url>          # tải repo từ remote về máy
git config user.name "Ten"
git config user.email "email@example.com"
```

### Làm việc hằng ngày
```bash
git status               # xem file đã đổi / đã stage / chưa theo dõi
git add <file>           # đưa thay đổi vào staging
git add .                # đưa tất cả thay đổi vào staging
git commit -m "message"  # lưu một mốc thay đổi kèm mô tả
git log --oneline        # xem lịch sử commit gọn
git diff                 # xem chi tiết thay đổi chưa stage
```

### Nhánh (branch)
```bash
git branch               # liệt kê nhánh
git checkout -b feat/x   # tạo + chuyển sang nhánh mới
git checkout main        # chuyển về nhánh có sẵn
git checkout <file>      # bỏ thay đổi chưa commit của 1 file
git branch -d feat/x     # xóa nhánh
git merge <branch>       # gộp nhánh khác vào nhánh hiện tại
git rebase main          # đưa commit của mình lên trên main mới nhất (lịch sử thẳng)
```
> Ghi chú: `git checkout` là lệnh đa năng (vừa chuyển nhánh, vừa khôi phục file). Các bản git mới tách thành `git switch` / `git restore` cho rõ nghĩa, nhưng dùng `checkout` vẫn hoàn toàn chuẩn.

### Remote (đồng bộ với GitHub)
```bash
git fetch                # tải cập nhật về (chưa gộp)
git pull                 # fetch + merge nhánh hiện tại
git push                 # đẩy commit lên remote
git push -u origin feat/x  # đẩy nhánh mới + liên kết upstream
```

### Hoàn tác / tạm cất
```bash
git reset --soft <commit>   # lùi commit, GIỮ lại code đã sửa
git reset --hard <commit>   # lùi commit, XÓA luôn thay đổi (cẩn thận)
git revert <commit>         # tạo commit đảo ngược 1 commit cũ (an toàn cho nhánh chung)
git stash                   # cất tạm thay đổi đang dở
git stash pop               # lấy lại thay đổi đã cất
```

### Cherry-pick
```bash
git cherry-pick <hash>      # "nhặt" 1 commit cụ thể từ nhánh khác áp vào nhánh hiện tại
git cherry-pick A^..B       # nhặt một dải commit từ A đến B
```
`cherry-pick` dùng khi chỉ muốn lấy **đúng một (vài) commit** mà không gộp cả nhánh — ví dụ đưa một hotfix từ `main` sang nhánh release. Lệnh này tạo ra commit mới (hash khác) nhưng cùng nội dung.

## 3. Git Flow trong dự án (feature branch + Pull Request)

Trong các dự án thực tế, nhóm thường dùng mô hình **feature branch + Pull Request** (gần với *GitHub Flow*). Nhánh `main` luôn ở trạng thái chạy được; mọi công việc mới đều tách ra nhánh riêng rồi gộp về qua Pull Request sau khi được review.

```
              feat/A:  ●──●──●
             /                \  (PR merge A)
main ──●────●──────────────────●─────────●──   (nhánh chính, luôn deploy được)
             \                          /  (PR merge B)
              feat/B:  ●──●──●─────────/
```

> Lưu ý: `feat/A` và `feat/B` đều tách ra **độc lập từ `main`** (không tách từ nhau), chạy song song rồi từng nhánh merge về `main` qua PR riêng. Chỉ tách nhánh này từ nhánh kia khi nhánh sau **bắt buộc cần code** của nhánh trước mà nó chưa được merge (trường hợp hiếm, nên tránh).

### Quy trình từng bước
1. Tạo nhánh từ `main` cho mỗi task: `git checkout -b feat/ten-tinh-nang`
2. Code và commit nhiều lần trên nhánh đó.
3. Đẩy nhánh lên remote: `git push -u origin feat/ten-tinh-nang`
4. Tạo **Pull Request (PR)** trên GitHub để xin gộp vào `main`.
5. **Review code**: đồng nghiệp/lead xem, góp ý, duyệt.
6. **Merge** PR vào `main` sau khi được duyệt và CI pass.
7. Cập nhật lại máy: `git checkout main && git pull`, rồi xóa nhánh feature.

### Quy ước tên nhánh
- `feat/...` — tính năng mới
- `fix/...` — sửa lỗi
- `refactor/...` — dọn dẹp code
- `hotfix/...` — sửa gấp trên production

### Nguyên tắc tốt
- Không commit thẳng lên `main`.
- Mỗi PR nhỏ, một mục đích → dễ review.
- Commit message rõ ràng: `feat: thêm chức năng login`, `fix: sửa lỗi 500 ở checkout`.
- `main` luôn ở trạng thái chạy được (deploy được bất cứ lúc nào).

## 4. Kết luận

Git là công cụ nền tảng của mọi quy trình phát triển phần mềm hiện đại. Nắm vững nhóm lệnh cơ bản (add, commit, push, pull, branch, checkout) cùng quy trình feature-branch + Pull Request giúp làm việc nhóm an toàn, dễ review và dễ quay lui khi có sự cố. Các lệnh nâng cao như `rebase`, `cherry-pick`, `revert`, `stash` là công cụ bổ trợ để xử lý những tình huống đặc thù trong thực tế.
