# Bài 3: Xử Lý Xung Đột Phức Tạp Trong Quá Trình Rebase

## Mục Tiêu

- Hiểu sự khác biệt giữa giải quyết xung đột khi **Merge** và khi **Rebase**.
- Áp dụng quy trình giải quyết xung đột từng bước (**step-by-step**) trong Rebase.
- Đưa lịch sử nhánh tính năng lên trên đầu nhánh chính một cách thẳng hàng.

---

## Tổng Quan Bối Cảnh

| Nhánh | Thay đổi |
|-------|----------|
| `main` | Commit 1: port 8080->8081; Commit 2: thêm "env": "production" |
| `feature-api` | Commit 1: port 8080->9000; Commit 2: debug false->true |

Cả hai nhánh đều phân kỳ từ commit **"init config"** và cùng sửa các dòng trong `config.json`, tạo ra điều kiện xung đột khi rebase.

---

## Các Bước Thực Hiện

### Bước 1: Khởi tạo repository và commit ban đầu trên `main`

```bash
git init
git config user.name "Student"
git config user.email "student@example.com"
```

Tạo file `config.json`:
```json
{
  "port": 8080,
  "debug": false
}
```

```bash
git add config.json
git commit -m "init config"
git branch -M main
```

---

### Bước 2: Tạo nhánh `feature-api` và thực hiện 2 commit

```bash
git checkout -b feature-api
```

**Commit 1** - Đổi port thành 9000:
```json
{
  "port": 9000,
  "debug": false
}
```
```bash
git add config.json && git commit -m "feat: change port"
```

**Commit 2** - Bật debug:
```json
{
  "port": 9000,
  "debug": true
}
```
```bash
git add config.json && git commit -m "feat: enable debug"
```

---

### Bước 3: Quay lại `main` và thực hiện 2 commit mới

```bash
git checkout main
```

**Commit 1** - Đổi port thành 8081:
```json
{
  "port": 8081,
  "debug": false
}
```
```bash
git add config.json && git commit -m "update port on main"
```

**Commit 2** - Thêm trường `env`:
```json
{
  "port": 8081,
  "debug": false,
  "env": "production"
}
```
```bash
git add config.json && git commit -m "add env config"
```

---

### Bước 4: Cấu trúc nhánh TRƯỚC khi rebase

```
* 170f930 add env config          <- main (HEAD cua main)
* 4568cd5 update port on main
| * 72279a1 feat: enable debug    <- feature-api (HEAD)
| * 9b5a869 feat: change port
|/
* e57be2c init config             <- diem phan ky
```

---

### Bước 5: Tiến hành Rebase

```bash
git checkout feature-api
git rebase main
```

---

## Chi Tiết Xử Lý Xung Đột

### CONFLICT LAN 1 - Khi áp dụng commit `feat: change port` (Chặng 1/2)

**Thông báo lỗi:**
```
CONFLICT (content): Merge conflict in config.json
error: could not apply 9b5a869... feat: change port
Rebasing (1/2)
```

**Nội dung file khi xung đột:**
```
{
<<<<<<< HEAD
  "port": 8081,
  "debug": false,
  "env": "production"
=======
  "port": 9000,
  "debug": false
>>>>>>> 9b5a869 (feat: change port)
}
```

**Phân tích:**
- `HEAD` (ngon cua `main`): port = **8081**, co them `"env": "production"`
- Commit cua `feature-api`: port = **9000**, khong co `env`
- Xung dot xay ra do ca hai nhanh deu sua dong `"port"` tu gia tri goc 8080.

**Cách giải quyết:**
> Giữ `port: 9000` theo mục đích của feature-api (đổi port), đồng thời bảo tồn `"env": "production"` từ nhánh main để không làm mất thay đổi của nhánh chính.

```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
```

```bash
git add config.json
git rebase --continue
```

---

### CONFLICT LAN 2 - Khi áp dụng commit `feat: enable debug` (Chặng 2/2)

**Thông báo lỗi:**
```
CONFLICT (content): Merge conflict in config.json
error: could not apply 72279a1... feat: enable debug
Rebasing (2/2)
```

**Nội dung file khi xung đột:**
```
{
  "port": 9000,
<<<<<<< HEAD
  "debug": false,
  "env": "production"
=======
  "debug": true
>>>>>>> 72279a1 (feat: enable debug)
}
```

**Phân tích:**
- `HEAD` (sau rebase commit 1): debug = **false**, co `env: "production"`
- Commit `feat: enable debug`: muon doi `debug` thanh **true**, nhung khong co dong `env`
- Xung dot tai dong `"debug"` va su thieu vang cua `env`.

**Cách giải quyết:**
> Bật `debug: true` theo đúng ý định commit, đồng thời giữ lại `"env": "production"` từ main để code nhánh chính không bị phá vỡ.

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

```bash
git add config.json
git rebase --continue
```

**Ket qua:** `Successfully rebased and updated refs/heads/feature-api.`

---

## Kết Quả Sau Khi Rebase

### git status

```
On branch feature-api
nothing to commit, working tree clean
```

### git log --graph --oneline

```
* 52277c3 feat: enable debug
* 5e73083 feat: change port
* 170f930 add env config
* 4568cd5 update port on main
* e57be2c init config
```

> Lich su Git thang tap, **khong co merge commit** phu.
> Cac commit cua `feature-api` nam noi tiep ngay **sau** cac commit moi nhat cua `main`.

---

## So Sánh: Rebase vs Merge

| Tieu chi | Merge | Rebase |
|----------|-------|--------|
| Lich su commit | Co them merge commit phu | Thang tap, tuyen tinh |
| So lan giai quyet conflict | 1 lan duy nhat | Moi commit mot lan (tung chang) |
| Bao toan lich su goc | Co | Khong (commit duoc viet lai) |
| De doc lich su | Kho hon | De hon, ro rang hon |
| Phu hop khi | Feature branch public | Feature branch local/ca nhan |

---

## Nội Dung `config.json` Cuối Cùng

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

---

## Lưu Ý Quan Trọng

1. **Khong dung `git merge`** - bai yeu cau rebase de giu lich su thang tap.
2. **Moi commit trong rebase co the gay ra conflict rieng** - phai giai quyet tung chang.
3. **Khong duoc xoa thay doi cua nhanh main** (`env: "production"`) khi giai quyet conflict.
4. Sau khi sua file xong, **luon phai chay `git add`** truoc `git rebase --continue`.
