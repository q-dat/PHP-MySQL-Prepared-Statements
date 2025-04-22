# 🔧 1. CÁC CÂU LỆNH SQL THƯỜNG DÙNG

### ✅ SELECT – Truy vấn dữ liệu

```sql
SELECT * FROM tasks;
SELECT id, title FROM tasks WHERE status = 'pending';
```

| Thành phần     | Mô tả                                                       |
|----------------|-------------------------------------------------------------|
| `SELECT`       | Cột cần truy vấn                                            |
| `FROM`         | Tên bảng dữ liệu                                            |
| `WHERE`        | Điều kiện lọc dữ liệu                                       |
| `ORDER BY`     | Sắp xếp theo cột (mặc định ASC hoặc dùng DESC)              |
| `LIMIT`        | Giới hạn số dòng kết quả trả về                             |
| `OFFSET`       | Bỏ qua n dòng đầu tiên                                      |
| `DISTINCT`     | Loại bỏ kết quả trùng                                       |

#### Ví dụ nâng cao:
```sql
SELECT DISTINCT status FROM tasks;
SELECT * FROM tasks WHERE status = 'done' ORDER BY due_date ASC LIMIT 5 OFFSET 10;
```

---

### ✍️ INSERT INTO – Thêm dữ liệu

```sql
INSERT INTO tasks (title, status) VALUES ('Fix login bug', 'pending');
```

| Thành phần     | Mô tả                                                       |
|----------------|-------------------------------------------------------------|
| `INSERT INTO`  | Bảng và cột cần thêm dữ liệu                                |
| `VALUES`       | Danh sách giá trị tương ứng theo thứ tự cột                 |

---

### 🔄 UPDATE – Cập nhật dữ liệu

```sql
UPDATE tasks SET status = 'done', updated_at = NOW() WHERE id = 5;
```

| Thành phần     | Mô tả                                                       |
|----------------|-------------------------------------------------------------|
| `UPDATE`       | Tên bảng                                                    |
| `SET`          | Giá trị mới cần cập nhật cho các cột                        |
| `WHERE`        | Điều kiện lọc dòng để cập nhật                              |

> ⚠️ Không có `WHERE` sẽ cập nhật toàn bộ bảng!

---

### ❌ DELETE – Xoá dữ liệu

```sql
DELETE FROM tasks WHERE id = 3;
```

| Thành phần     | Mô tả                                                       |
|----------------|-------------------------------------------------------------|
| `DELETE FROM`  | Xoá dữ liệu từ bảng                                         |
| `WHERE`        | Điều kiện lọc để xoá                                        |

---

### 🔗 JOIN – Kết hợp bảng

```sql
SELECT t.*, u.name FROM tasks t
JOIN users u ON t.user_id = u.id
WHERE u.role = 'manager';
```

| Loại JOIN      | Ý nghĩa                                                    |
|----------------|------------------------------------------------------------|
| `INNER JOIN`   | Chỉ lấy các dòng có dữ liệu khớp ở cả hai bảng             |
| `LEFT JOIN`    | Lấy tất cả dòng từ bảng bên trái + khớp nếu có từ bảng phải|
| `RIGHT JOIN`   | Ngược lại của LEFT JOIN                                    |

---

### 📊 Hàm tổng hợp (Aggregate)

```sql
SELECT COUNT(*) FROM tasks;
SELECT AVG(salary) FROM employees WHERE department = 'IT';
```

| Hàm      | Mô tả                        |
|----------|-----------------------------|
| `COUNT()`| Đếm số dòng                 |
| `SUM()`  | Tổng giá trị                |
| `AVG()`  | Giá trị trung bình          |
| `MIN()`  | Nhỏ nhất                    |
| `MAX()`  | Lớn nhất                    |

---

### 🔎 Điều kiện nâng cao

```sql
SELECT * FROM tasks WHERE title LIKE '%bug%';
SELECT * FROM tasks WHERE status IN ('pending', 'done');
SELECT * FROM tasks WHERE due_date BETWEEN '2025-04-01' AND '2025-04-30';
```

| Mệnh đề     | Mô tả                                       |
|-------------|----------------------------------------------|
| `LIKE`      | Tìm kiếm gần đúng (sử dụng `%`, `_`)         |
| `IN`        | So sánh với danh sách giá trị                |
| `BETWEEN`   | So sánh nằm trong khoảng                     |
| `IS NULL`   | Kiểm tra giá trị NULL                        |
| `AND`, `OR`, `NOT` | Tổ hợp điều kiện                      |

---

# 🛠️ 2. PREPARED STATEMENTS TRONG PHP (MySQLi)

### ❓ Tại sao nên dùng?
- Tránh **SQL Injection**
- Tự động **escape dữ liệu**
- Tăng hiệu suất với truy vấn lặp lại

---

## ✅ Các hàm chính:

| Hàm                      | Mô tả                                               |
|--------------------------|----------------------------------------------------|
| `prepare($query)`        | Chuẩn bị truy vấn có chứa `?`                      |
| `bind_param()`           | Gán biến vào từng tham số `?` (theo kiểu dữ liệu) |
| `execute()`              | Thực thi truy vấn                                 |
| `get_result()`           | Lấy kết quả (dùng cho SELECT)                      |
| `fetch_assoc()`          | Lấy một dòng dạng mảng liên kết                    |
| `fetch_all()`            | Lấy toàn bộ kết quả (dạng mảng)                   |

### 📌 Ví dụ: SELECT

```php
$sql = "SELECT * FROM tasks WHERE status = ? ORDER BY id DESC LIMIT ?";
$stmt = $db->prepare($sql);
$stmt->bind_param("si", $status, $limit); // s: string, i: int
$stmt->execute();
$result = $stmt->get_result()->fetch_all(MYSQLI_ASSOC);
```

### 📌 Ví dụ: INSERT

```php
$stmt = $db->prepare("INSERT INTO tasks (title, status) VALUES (?, ?)");
$stmt->bind_param("ss", $title, $status);
$stmt->execute();
```

### 📌 Ví dụ: UPDATE

```php
$stmt = $db->prepare("UPDATE tasks SET title = ?, status = ? WHERE id = ?");
$stmt->bind_param("ssi", $title, $status, $id);
$stmt->execute();
```

---

## 🔁 Kiểu dữ liệu trong `bind_param()`

| Ký hiệu | Kiểu dữ liệu    |
|---------|-----------------|
| `i`     | Integer         |
| `d`     | Double/float    |
| `s`     | String          |
| `b`     | BLOB (binary)   |

---

## 📦 Pagination – Phân trang dữ liệu

```php
$page = 2;
$limit = 10;
$offset = ($page - 1) * $limit;

$sql = "SELECT * FROM tasks ORDER BY id DESC LIMIT ? OFFSET ?";
$stmt = $db->prepare($sql);
$stmt->bind_param("ii", $limit, $offset);
$stmt->execute();
$data = $stmt->get_result()->fetch_all(MYSQLI_ASSOC);
```

---

---

```php
$page = 2;
```
✅ Đây là **số trang hiện tại** mà bạn muốn lấy dữ liệu.  
Ví dụ: Nếu đang ở trang 2 → muốn hiển thị các dòng từ số 11 đến 20 (nếu mỗi trang 10 dòng).

---

```php
$limit = 10;
```
✅ **Số dòng (bản ghi) hiển thị trên mỗi trang**.  
Bạn có thể tùy chỉnh số này tuỳ vào UI hoặc user setting.

---

```php
$offset = ($page - 1) * $limit;
```
✅ **Vị trí bắt đầu lấy dữ liệu** trong bảng.  
- Với `$page = 2`, `$limit = 10` → offset = `(2 - 1) * 10 = 10`  
→ MySQL sẽ **bỏ qua 10 dòng đầu tiên**, và lấy từ dòng thứ 11 trở đi.

| Trang | Offset | Ghi chú                          |
|-------|--------|----------------------------------|
| 1     | 0      | Lấy từ dòng 1 đến dòng 10       |
| 2     | 10     | Dòng 11 đến dòng 20             |
| 3     | 20     | Dòng 21 đến dòng 30             |

---

```php
$sql = "SELECT * FROM tasks ORDER BY id DESC LIMIT ? OFFSET ?";
```
✅ Câu SQL này sẽ:
- **Lấy toàn bộ cột (`*`) từ bảng `tasks`**
- **Sắp xếp giảm dần theo `id`**
- **Giới hạn `$limit` dòng**
- **Bỏ qua `$offset` dòng đầu tiên**

---

```php
$stmt = $db->prepare($sql);
```
✅ Chuẩn bị truy vấn SQL, `$stmt` là **prepared statement object**.

---

```php
$stmt->bind_param("ii", $limit, $offset);
```
✅ Gán giá trị thật vào các `?`:
- `"ii"` nghĩa là:
  - `i` đầu tiên là `$limit` (kiểu `int`)
  - `i` thứ hai là `$offset` (kiểu `int`)

---

```php
$stmt->execute();
```
✅ Thực thi câu SQL sau khi đã bind giá trị.

---

```php
$data = $stmt->get_result()->fetch_all(MYSQLI_ASSOC);
```
✅ Lấy kết quả dưới dạng **mảng các dòng**, mỗi dòng là một mảng kết hợp (key là tên cột).

---

### ✅ Tổng hợp ý nghĩa các biến:

| Biến       | Ý nghĩa                                                         |
|------------|------------------------------------------------------------------|
| `$page`    | Số trang hiện tại mà người dùng yêu cầu                          |
| `$limit`   | Số lượng dòng hiển thị trên mỗi trang                            |
| `$offset`  | Vị trí dòng bắt đầu lấy trong kết quả truy vấn                   |
| `$sql`     | Câu SQL có `LIMIT ? OFFSET ?` để lấy dữ liệu phân trang         |
| `$stmt`    | Statement object để thực thi SQL một cách an toàn (prepared)    |
| `$data`    | Mảng kết quả trả về từ truy vấn (dùng cho frontend/table UI...) |

---
