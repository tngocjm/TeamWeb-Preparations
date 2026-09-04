# BUỔI 4: TỐI ƯU TRUY VẤN, INDEX VÀ TRANSACTION

## I. Tổng quan buổi học

- Tối ưu truy vấn SQL.
- Khái niệm và cách sử dụng index.
- Transaction và các tính chất ACID, dirty read, dirty write

## II. Tối ưu truy vấn

### 1. Tối ưu truy vấn là gì?

Tối ưu truy vấn là quá trình làm cho câu lệnh SQL sử dụng ít tài nguyên hơn và trả kết quả nhanh hơn nhưng vẫn giữ nguyên tính đúng đắn của dữ liệu.

Một truy vấn nhanh trên bảng 100 dòng chưa chắc còn nhanh khi bảng có 10 triệu dòng. Vì vậy, cần đánh giá bằng dữ liệu có quy mô gần với thực tế.

### 2. Đọc kế hoạch thực thi bằng `EXPLAIN`

#### a. `EXPLAIN`

`EXPLAIN` cho biết kế hoạch mà database dự định sử dụng nhưng thường không thực thi truy vấn.
![alt text](image.png)

```sql
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 1001;
```
#### b. `EXPLAIN ANALYZE`

`EXPLAIN ANALYZE` thực sự chạy truy vấn và bổ sung số dòng, thời gian thực tế, từ đó so sánh estimated rows với actual rows.

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 1001;
```

> **Cẩn thận:** Vì truy vấn được thực thi thật, không chạy `EXPLAIN ANALYZE` tùy tiện với `UPDATE`, `DELETE` hoặc truy vấn rất nặng trên môi trường production.

### 3. Quy trình tối ưu một truy vấn chậm

1. Xác định chính xác truy vấn chậm và tham số đầu vào.
2. Đo thời gian hiện tại để làm mốc so sánh.
3. Dùng `EXPLAIN` hoặc `EXPLAIN ANALYZE` xem execution plan.
4. Xác định nút thắt: full scan, join sai, sort lớn, trả quá nhiều dòng, thiếu index,...
5. Kiểm tra cấu trúc bảng, index và statistics.
6. Viết lại truy vấn hoặc bổ sung index phù hợp.
7. Đo lại bằng cùng dữ liệu và điều kiện.
8. Kiểm tra tác động lên các thao tác ghi và các truy vấn khác.

Không nên tạo index theo cảm tính trước khi xem truy vấn thực tế và execution plan.

### 4. Các kỹ thuật tối ưu truy vấn thường gặp

Sử dụng tên cột cụ thể thay vì SELECT *: Khi chỉ cần một vài cột, việc sử dụng SELECT * có thể làm tăng thời gian xử lý do truy vấn lấy toàn bộ dữ liệu của bảng. Thay vào đó, hãy chỉ định các cột cụ thể để giảm tải truy vấn.

Tối ưu hóa bằng chỉ số (Index): Index giúp tăng tốc độ truy vấn, đặc biệt với các câu lệnh WHERE, JOIN và ORDER BY. Tạo chỉ số cho các cột thường được sử dụng trong các mệnh đề này sẽ cải thiện đáng kể hiệu suất.

Hạn chế sử dụng hàm trong điều kiện WHERE: Các hàm như UPPER() hay LOWER() trong WHERE có thể làm chậm quá trình xử lý. Thay vào đó, nên sử dụng các hàm này ngoài câu truy vấn.

Un-nest các truy vấn con (Sub-query): Các truy vấn con phức tạp thường gây giảm hiệu suất. Thay vì sử dụng sub-query, có thể dùng JOIN hoặc WITH để tái cấu trúc truy vấn và tăng hiệu quả.

Tránh sử dụng DISTINCT nếu không cần thiết: Sử dụng DISTINCT để loại bỏ các giá trị trùng lặp sẽ làm tăng thời gian xử lý của truy vấn. Nếu có thể, hãy xem xét loại bỏ mệnh đề này.

Giảm thiểu sử dụng HAVING: Mệnh đề HAVING lọc kết quả sau khi GROUP BY, điều này làm tăng thời gian xử lý. Hãy sử dụng WHERE để lọc trước khi nhóm dữ liệu.

Tránh các mệnh đề OR phức tạp: Thay vì sử dụng OR trong điều kiện, hãy cân nhắc sử dụng IN hoặc viết lại truy vấn sao cho dễ xử lý hơn.

Sử dụng UNION ALL thay cho UNION: UNION loại bỏ các kết quả trùng lặp, gây thêm một bước xử lý. Nếu không cần loại bỏ trùng lặp, hãy sử dụng UNION ALL để cải thiện tốc độ.

Sử dụng công cụ phân tích EXPLAIN: Dùng lệnh EXPLAIN để phân tích kế hoạch thực thi truy vấn và điều chỉnh phù hợp. Đây là công cụ mạnh mẽ giúp bạn hiểu cách CSDL thực hiện truy vấn của bạn.

## III. Index

### 1. Index là gì?

Index là một cấu trúc dữ liệu bổ sung giúp database tìm dòng nhanh hơn mà không phải đọc lần lượt toàn bộ bảng.

Có thể hình dung:

- Bảng dữ liệu giống một cuốn sách.
- Table scan giống đọc từ trang đầu để tìm một chủ đề.
- Index giống mục lục, giúp xác định nhanh vị trí cần đọc.

Index lưu giá trị của một hoặc nhiều cột cùng thông tin để xác định dòng dữ liệu tương ứng. Vì là cấu trúc bổ sung, index chiếm dung lượng và phải được cập nhật khi dữ liệu thay đổi.

VD: 

```sql
CREATE TABLE Customers (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    phone VARCHAR(20)
);
```
Truy vấn theo email
SELECT * FROM Customers WHERE email = 'abc@example.com';
→ Tạo chỉ mục:

CREATE INDEX idx_email ON Customers(email);
COPY
Lần sau truy vấn, MySQL sẽ tìm nhanh hơn vì không phải quét toàn bộ bảng.

### 2. Khi nào nên tạo index?

- Bảng có dữ liệu lớn
- Truy vấn hay dùng WHERE, JOIN, ORDER BY, GROUP BY trên một số cột nhất định
- Tăng hiệu suất tìm kiếm, lọc, liên kết dữ liệu
- Truy vấn có độ chọn lọc đủ tốt và chạy thường xuyên.

### 3. Tạo, xem và xóa index

#### a. Index một cột

CREATE INDEX ten_index ON ten_bang(cot1 [, cot2, ...]);


```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

Index này có thể hỗ trợ:

```sql
SELECT order_id, order_date, total_amount
FROM orders
WHERE customer_id = 1001;
```

#### b. Index duy nhất

```sql
CREATE UNIQUE INDEX idx_customers_email
ON customers(email);
```

Unique index vừa hỗ trợ truy vấn vừa ngăn hai dòng có cùng email. Trong nhiều DBMS, khai báo `UNIQUE` constraint sẽ tự tạo cấu trúc index tương ứng, vì vậy cần kiểm tra trước để tránh tạo index trùng.

#### c. Xóa index

Cú pháp phụ thuộc DBMS:

```sql
-- PostgreSQL
DROP INDEX idx_orders_customer_id;

-- MySQL
DROP INDEX idx_orders_customer_id ON orders;
```

### 4. Index ghép (composite index)

Index ghép chứa nhiều cột:

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

Index trên phù hợp với các truy vấn như:

```sql
WHERE customer_id = 1001;
```

```sql
WHERE customer_id = 1001
  AND order_date >= '2026-01-01';
```

Nhưng thường không tối ưu cho truy vấn chỉ lọc theo:

```sql
WHERE order_date >= '2026-01-01';
```

### 5. Quy tắc tiền tố trái

Với index `(A, B, C)`, database thường có thể tận dụng tốt các tiền tố bắt đầu từ bên trái:

- `(A)`.
- `(A, B)`.
- `(A, B, C)`.

Nếu bỏ qua `A` và chỉ tìm theo `B` hoặc `C`, index thường không hỗ trợ tìm kiếm trực tiếp hiệu quả như một index bắt đầu bằng cột đó.

Ví dụ với index:

```sql
CREATE INDEX idx_orders_status_customer_date
ON orders(status, customer_id, order_date);
```

Truy vấn phù hợp:

```sql
WHERE status = 'PAID'
  AND customer_id = 1001
  AND order_date >= '2026-01-01';
```

> Quy tắc tiền tố trái là mô hình tư duy quan trọng, nhưng khả năng sử dụng index chính xác còn phụ thuộc DBMS, loại điều kiện và kế hoạch thực thi.

### 6. Chọn thứ tự cột trong index ghép

Không có quy tắc đơn giản “cột có độ chọn lọc cao nhất luôn đứng đầu”. Cần dựa vào các truy vấn quan trọng.

Các yếu tố cần xem xét:

1. Cột xuất hiện trong điều kiện lọc bằng `=`.
2. Cột xuất hiện trong điều kiện phạm vi.
3. Cột dùng trong `ORDER BY` và `GROUP BY`.
4. Tần suất của từng mẫu truy vấn.
5. Độ chọn lọc, tức tỉ lệ số dòng còn lại sau khi lọc.

Ví dụ truy vấn phổ biến:

```sql
SELECT order_id, order_date, total_amount
FROM orders
WHERE customer_id = :customer_id
  AND status = 'PAID'
  AND order_date >= :from_date
ORDER BY order_date DESC;
```

Một index ứng viên:

```sql
CREATE INDEX idx_orders_customer_status_date
ON orders(customer_id, status, order_date DESC);
```

Sau khi tạo vẫn phải dùng `EXPLAIN` để xác nhận.

### 7. Equality trước, range sau

Khi thiết kế index cho một query cụ thể, các cột lọc bằng thường được đặt trước cột lọc theo khoảng:

```sql
WHERE customer_id = 1001
  AND order_date >= '2026-01-01';
```

Index ứng viên:

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

Sau khi database đi vào một khoảng rộng trên cột nào đó, khả năng sử dụng các cột đứng sau để thu hẹp vùng tìm kiếm có thể giảm. Đây là nguyên tắc định hướng chứ không thay thế việc đọc execution plan.

### 8. Khi nào không nên hoặc cần cân nhắc kỹ?

- Bảng rất nhỏ.
- Cột hiếm khi được truy vấn.
- Cột có ít giá trị phân biệt và truy vấn thường lấy phần lớn bảng.
- Bảng có tần suất ghi rất cao nhưng ít đọc.
- Index mới trùng hoặc là tiền tố không cần thiết của index hiện có.
- Cột rất lớn khiến index phình to.

## V. Transaction

### 1. Transaction là gì?

Transaction là một nhóm thao tác đọc/ghi được database xử lý như một đơn vị công việc logic. Một transaction hoặc hoàn thành toàn bộ, hoặc các thay đổi của nó không được công nhận.

Ví dụ chuyển 500.000 đồng từ tài khoản A sang B gồm hai thao tác:

1. Trừ tiền tài khoản A.
2. Cộng tiền tài khoản B.

Nếu chỉ bước 1 thành công rồi hệ thống lỗi, dữ liệu bị sai. Hai thao tác phải được đặt trong cùng transaction.

### 2. Các lệnh cơ bản

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 500000
WHERE account_id = 1;

UPDATE accounts
SET balance = balance + 500000
WHERE account_id = 2;

COMMIT;
```

Nếu có lỗi hoặc điều kiện nghiệp vụ không thỏa mãn:

```sql
ROLLBACK;
```

Tên lệnh bắt đầu transaction có thể là `BEGIN`, `BEGIN TRANSACTION` hoặc `START TRANSACTION` tùy DBMS.


### 3. `SAVEPOINT`

`SAVEPOINT` cho phép quay lại một mốc bên trong transaction mà không hủy toàn bộ transaction.

```sql
BEGIN;

INSERT INTO orders(order_id, customer_id, status, total_amount)
VALUES (5001, 1001, 'PENDING', 1200000);

SAVEPOINT after_order_created;

INSERT INTO order_items(order_id, product_id, quantity, unit_price)
VALUES (5001, 20, 2, 600000);

-- Nếu phần chi tiết bị lỗi:
ROLLBACK TO SAVEPOINT after_order_created;

COMMIT;
```

### 4. Transaction trong nghiệp vụ đặt hàng

Khi khách đặt sản phẩm, hệ thống thường cần:

1. Kiểm tra và trừ tồn kho.
2. Tạo đơn hàng.
3. Tạo các dòng chi tiết đơn hàng.
4. Ghi nhận thanh toán hoặc trạng thái thanh toán.

Các bước làm thay đổi dữ liệu cốt lõi nên nằm trong cùng một transaction hoặc một cơ chế nhất quán thích hợp.

Ví dụ rút gọn:

```sql
BEGIN;

UPDATE products
SET stock = stock - 2
WHERE product_id = 20
  AND stock >= 2;

-- Ứng dụng kiểm tra affected_rows

-- Nếu affected_rows = 0:
ROLLBACK;

-- Nếu affected_rows = 1:
-- tiếp tục 2 câu INSERT

INSERT INTO orders(order_id, customer_id, status, total_amount)
VALUES (5001, 1001, 'PAID', 1200000);

INSERT INTO order_items(order_id, product_id, quantity, unit_price)
VALUES (5001, 20, 2, 600000);

COMMIT;
```
---

## VI. ACID

ACID là bốn tính chất quan trọng của transaction: Atomicity, Consistency, Isolation và Durability.

### 1. Atomicity — Tính nguyên tử

Tất cả thao tác trong transaction được xem như một đơn vị không thể chia nhỏ:

- Hoặc tất cả thành công và `COMMIT`.
- Hoặc tất cả bị hủy bằng `ROLLBACK`.

Ví dụ chuyển tiền: không được phép trừ A mà chưa cộng B.

### 2. Consistency — Tính nhất quán

Transaction phải đưa database từ một trạng thái hợp lệ sang một trạng thái hợp lệ khác, tuân thủ các quy tắc như:

- Primary key, foreign key.
- `UNIQUE`, `NOT NULL`, `CHECK`.
- Kiểu dữ liệu.
- Quy tắc nghiệp vụ do ứng dụng hoặc database đảm nhận.

Ví dụ:

- Số dư không được âm nếu nghiệp vụ cấm thấu chi.
- `order_items.order_id` phải tham chiếu đến một đơn hàng tồn tại.
- Tổng số tiền chuyển đi phải bằng tổng số tiền nhận.

> Database chỉ tự đảm bảo các ràng buộc đã được khai báo hoặc lập trình. ACID không tự biết mọi quy tắc nghiệp vụ của hệ thống.

### 3. Isolation — Tính cô lập

Các transaction chạy đồng thời không nên gây ra kết quả sai do nhìn thấy trạng thái trung gian của nhau. Kết quả quan sát được phụ thuộc vào isolation level.

Isolation càng mạnh thường càng dễ lập luận về tính đúng, nhưng có thể:

- Tăng thời gian chờ khóa.
- Tăng khả năng transaction phải chạy lại.
- Giảm mức độ xử lý đồng thời.

### 4. Durability — Tính bền vững

Sau khi database báo `COMMIT` thành công, dữ liệu phải tồn tại ngay cả khi tiến trình hoặc máy chủ gặp sự cố, theo cam kết durability của cấu hình hệ thống.

DBMS thường dùng:

- Write-ahead log/transaction log.
- Ghi log xuống thiết bị lưu trữ bền vững.
- Cơ chế phục hồi sau sự cố.
- Replication và backup để tăng khả năng phục hồi, dù chúng không đồng nghĩa hoàn toàn với durability của một transaction đơn lẻ.

---

## VII. Các vấn đề khi chạy transaction đồng thời

Giả sử có hai transaction `T1` và `T2` cùng thao tác trên một dữ liệu.

### 1. Dirty read — Đọc dữ liệu chưa commit

Dirty read xảy ra khi một transaction đọc thay đổi chưa được commit của transaction khác. Nếu transaction kia rollback, giá trị đã đọc chưa bao giờ thực sự tồn tại trong trạng thái đã commit của database.

Ví dụ số dư ban đầu là 10 triệu:

| Thời điểm | Transaction T1 | Transaction T2 |
|---|---|---|
| 1 | `UPDATE balance = 5 triệu` | |
| 2 | Chưa commit | Đọc `balance = 5 triệu` |
| 3 | `ROLLBACK` | Đã tính toán dựa trên 5 triệu |
| 4 | Số dư trở lại 10 triệu | Kết quả T2 có thể sai |

Giá trị 5 triệu mà T2 đọc ở bước 2 là dữ liệu “bẩn”.

### 2. Dirty write — Ghi đè dữ liệu chưa commit

Dirty write xảy ra khi một transaction ghi đè lên giá trị đang được transaction khác thay đổi nhưng chưa commit.

| Thời điểm | Transaction T1 | Transaction T2 |
|---|---|---|
| 1 | Ghi `balance = 8 triệu`, chưa commit | |
| 2 | | Ghi đè `balance = 7 triệu`, trong khi T1 chưa kết thúc |
| 3 | `ROLLBACK` hoặc `COMMIT` | `COMMIT` |

Việc phục hồi đúng trở nên khó xác định vì thay đổi của hai transaction chồng lên nhau. Các DBMS quan hệ phổ biến ngăn dirty write bằng cơ chế khóa hoặc kiểm soát đồng thời: T2 thường phải chờ T1 kết thúc.

