# Buổi 9: Thao tác với DATABASE

## Mục lục
- [Buổi 9: Thao tác với DATABASE](#buổi-9-thao-tác-với-database)
  - [Mục lục](#mục-lục)
  - [I. Các mối quan hệ(1..N, N..N) trong JPA (@ManyToMany, @OneToMany,...)](#i-các-mối-quan-hệ1n-nn-trong-jpa-manytomany-onetomany)
    - [1. OneToMany, ManyToOne](#1-onetomany-manytoone)
      - [a. Khái niệm](#a-khái-niệm)
      - [b. Cách sử dụng @OneToMany và @ManyToOne](#b-cách-sử-dụng-onetomany-và-manytoone)
    - [2. Tóm tắt các mối quan hệ](#2-tóm-tắt-các-mối-quan-hệ)
    - [3. CascadeTypes](#3-cascadetypes)
  - [II. Derived query methods (truy vấn theo tên hàm)](#ii-derived-query-methods-truy-vấn-theo-tên-hàm)
    - [1. Khái niệm và cơ chế hoạt động](#1-khái-niệm-và-cơ-chế-hoạt-động)
    - [2. Các từ khóa (Keywords) phổ biến](#2-các-từ-khóa-keywords-phổ-biến)
    - [3. Hỗ trợ phân trang (Pageable) và sắp xếp (Sort)](#3-hỗ-trợ-phân-trang-pageable-và-sắp-xếp-sort)
    - [4. Ví dụ thực tế](#4-ví-dụ-thực-tế)
    - [5. Ưu điểm và nhược điểm](#5-ưu-điểm-và-nhược-điểm)
  - [III. JPQL với @Query](#iii-jpql-với-query)
    - [1. Khái niệm JPQL](#1-khái-niệm-jpql)
    - [2. Truyền tham số trong JPQL](#2-truyền-tham-số-trong-jpql)
    - [3. Các kỹ thuật quan trọng trong JPQL](#3-các-kỹ-thuật-quan-trọng-trong-jpql)
    - [4. Cập nhật và xóa dữ liệu với @Modifying](#4-cập-nhật-và-xóa-dữ-liệu-với-modifying)
    - [5. Ví dụ thực tế](#5-ví-dụ-thực-tế-1)
    - [6. Ưu điểm và nhược điểm](#6-ưu-điểm-và-nhược-điểm)
  - [IV. Native query (SQL thuần)](#iv-native-query-sql-thuần)
    - [1. Khái niệm](#1-khái-niệm-1)
    - [2. Cách hứng dữ liệu (Mapping Result)](#2-cách-hứng-dữ-liệu-mapping-result)
    - [3. Khi nào nên dùng Native Query](#3-khi-nào-nên-dùng-native-query)
    - [4. Ví dụ thực tế](#4-ví-dụ-thực-tế-2)
    - [5. Ưu điểm, nhược điểm và lưu ý quan trọng](#5-ưu-điểm-nhược-điểm-và-lưu-ý-quan-trọng)
  - [V. Truy vấn sử dụng Entity manager](#v-truy-vấn-sử-dụng-entity-manager)
    - [1. Khái niệm và vai trò của EntityManager](#1-khái-niệm-và-vai-trò-của-entitymanager)
    - [2. Cách inject EntityManager trong Spring Boot](#2-cách-inject-entitymanager-trong-spring-boot)
    - [3. Các phương thức truy vấn và thao tác cốt lõi](#3-các-phương-thức-truy-vấn-và-thao-tác-cốt-lõi)
    - [4. Ứng dụng thực tế của EntityManager](#4-ứng-dụng-thực-tế-của-entitymanager)
    - [5. Khi nào nên dùng EntityManager](#5-khi-nào-nên-dùng-entitymanager)
  - [VI. Bảng so sánh tổng hợp 4 phương pháp truy vấn](#vi-bảng-so-sánh-tổng-hợp-4-phương-pháp-truy-vấn)

## I. Các mối quan hệ(1..N, N..N) trong JPA (@ManyToMany, @OneToMany,...)

### 1. OneToMany, ManyToOne

#### a. Khái niệm

- Đây là mối quan hệ phổ biến nhất. Một thực thể A có thể liên kết với nhiều thực thể B, nhưng mỗi thực thể B chỉ liên kết với duy nhất một thực thể A

VD với đề tài bán dịch vụ ăn uống trong rạp chiếu phim:

1 order có thể có nhiều dòng line order (chi tiết đơn hàng)

Còn 1 chi tiết đơn hàng thì chỉ thuộc về 1 đơn hàng cụ thể 

#### b. Cách sử dụng @OneToMany và @ManyToOne

Trong JPA, chúng ta sử dụng @OneToMany trên bên “một” và @ManyToOne trên bên “nhiều”. Theo quy ước và thực tế, bên “nhiều” (sở hữu khóa ngoại trong CSDL) thường là bên sở hữu (owning side) trong JPA. Vì vậy, @ManyToOne thường là bên sở hữu.

VD:

Bên nhiều
```java
@Entity
@Data
@Table(name = "order_detail")
public class OrderDetail {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @ManyToOne(fetch = FetchType.EAGER, optional = false)
    @JoinColumn(name = "order_id")
    private Order order;
}
```

Bên ít
```java
@Entity
@Data
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "total_amount", nullable = false)
    private BigDecimal totalAmount;

    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private List<OrderDetail> orderDetails;
}
```

- Giải thích:

***@JoinColumn:*** Annotation chỉ định cột dùng để liên kết, thường là cột khóa ngoại trong database.

***mappedBy:*** Là một thuộc tính của các annotation quan hệ như @OneToMany, @OneToOne, @ManyToMany, dùng để chỉ định phía sở hữu quan hệ đã được khai báo ở thuộc tính nào của Entity bên kia.

### 2. Tóm tắt các mối quan hệ 

| Mối quan hệ                           | Annotation (bên sở hữu)          | Annotation (bên đảo ngược, nếu hai chiều) | Cấu trúc bảng CSDL                                                   | Vai trò khóa ngoại / bảng trung gian                                                                              | Ví dụ phổ biến                             |
| ------------------------------------- | -------------------------------- | ----------------------------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| One-to-One (1:1)                      | `@OneToOne` (với `@JoinColumn`)  | `@OneToOne` (với `mappedBy`)              | Một FK trong bảng của bên sở hữu trỏ đến PK của bảng bên kia.        | Bên sở hữu chứa FK; FK thường có ràng buộc `UNIQUE` để đảm bảo quan hệ 1-1.                                       | Người dùng ↔ Hồ sơ cá nhân                 |
| One-to-Many (1:N) / Many-to-One (N:1) | `@ManyToOne` (với `@JoinColumn`) | `@OneToMany` (với `mappedBy`)             | Một FK trong bảng bên nhiều trỏ đến PK của bảng bên một.             | Bên nhiều (`@ManyToOne`) chứa FK; bên một (`@OneToMany`) dùng `mappedBy` để tham chiếu thuộc tính của bên sở hữu. | Tác giả ↔ Sách, Danh mục ↔ Sản phẩm        |
| Many-to-Many (N:M)                    | `@ManyToMany` (với `@JoinTable`) | `@ManyToMany` (với `mappedBy`)            | Một bảng trung gian chứa hai FK, mỗi FK trỏ đến PK của một bảng gốc. | Bên sở hữu định nghĩa bảng trung gian bằng `@JoinTable`; bên đảo ngược dùng `mappedBy`.                           | Sinh viên ↔ Khóa học, Bài viết ↔ Thẻ (Tag) |


Trong các ứng dụng CRUD thông thường dùng Spring Boot và JPA, @ManyToOne và @OneToMany thường xuất hiện nhiều hơn, còn @OneToOne và @ManyToMany thường ít gặp hơn trong nhiều mô hình nghiệp vụ.

***Khi nào nên dùng @ManyToMany:***

Chỉ nên dùng @ManyToMany mà không tạo Entity trung gian riêng khi bảng trung gian chỉ có nhiệm vụ liên kết hai bảng, không có dữ liệu hay nghiệp vụ riêng cần quản lý.

Với orderDetail của đề tài còn chứa thêm price tại thời điểm bán, số lượng,... nên k dùng ManyToMany

***Khi nào nên dùng @OneToOne:***

Nhóm thông tin có mục đích riêng.

Thông tin ít được sử dụng hoặc cần phân tách quyền truy cập.

Muốn quản lý vòng đời của chúng riêng biệt.

VD: User và UserProfile nhưng với 1 dự án lớn


### 3. CascadeTypes

PERSIST: Thao tác persist (lưu) thực thể chính sẽ cascade đến thực thể liên quan.

MERGE: Thao tác merge (cập nhật) thực thể chính sẽ cascade đến thực thể liên quan.

REMOVE: Thao tác remove (xóa) thực thể chính sẽ cascade đến thực thể liên quan. Cần rất cẩn thận với cái này, đặc biệt trong mối quan hệ Many-to-Many.

ALL: Cascade tất cả các thao tác (PERSIST, MERGE, REMOVE, REFRESH, DETACH).

orphanRemoval = true (chỉ trên @OneToMany, @OneToOne): Nếu một thực thể con bị gỡ bỏ khỏi collection của thực thể cha, nó sẽ bị xóa khỏi CSDL.

> Việc cấu hình cascade đúng cách giúp quản lý vòng đời của các thực thể liên quan một cách tự động, nhưng cấu hình sai có thể dẫn đến mất dữ liệu không mong muốn.

## II. Derived query methods (truy vấn theo tên hàm)

### 1. Khái niệm và cơ chế hoạt động

Derived query methods (hay *Query Creation from Method Names*) là một tính năng cực kỳ mạnh mẽ của Spring Data JPA. Thay vì phải tự viết câu truy vấn SQL hoặc JPQL, bạn chỉ cần đặt tên phương thức trong interface Repository theo đúng quy ước, Spring Data JPA sẽ tự động phân tích (parse) tên hàm thành cây cú pháp và sinh ra câu lệnh SQL tương ứng tại runtime.

- **Cấu trúc đặt tên chuẩn:**
  ```
  [Tiền tố hành động] + [By] + [Tiêu chí lọc kết hợp các từ khóa]
  ```
- **Các tiền tố thường gặp:**
  - `findBy...`, `getBy...`, `readBy...`, `queryBy...`: Dùng để truy vấn lấy danh sách hoặc một thực thể.
  - `countBy...`: Đếm số lượng bản ghi thỏa mãn điều kiện (trả về kiểu `long` hoặc `int`).
  - `existsBy...`: Kiểm tra xem có bản ghi nào tồn tại không (trả về kiểu `boolean`).
  - `deleteBy...`, `removeBy...`: Xóa các bản ghi thỏa mãn điều kiện.

### 2. Các từ khóa (Keywords) phổ biến

Spring Data JPA hỗ trợ rất nhiều từ khóa để kết hợp điều kiện tìm kiếm:

| Từ khóa | Ý nghĩa | Ví dụ trong Repository | SQL tương đương |
| :--- | :--- | :--- | :--- |
| `And` | Kết hợp AND | `findByNameAndIsAvailable(name, isAvailable)` | `WHERE name = ? AND is_available = ?` |
| `Or` | Kết hợp OR | `findByPriceOrIsAvailable(price, isAvailable)` | `WHERE price = ? OR is_available = ?` |
| `Is`, `Equals` | So sánh bằng (mặc định) | `findByName(name)` hoặc `findByNameEquals(name)` | `WHERE name = ?` |
| `Between` | Nằm trong khoảng | `findByPriceBetween(min, max)` | `WHERE price BETWEEN ? AND ?` |
| `LessThan`, `LessThanEqual` | Nhỏ hơn / Nhỏ hơn hoặc bằng | `findByPriceLessThanEqual(maxPrice)` | `WHERE price <= ?` |
| `GreaterThan`, `GreaterThanEqual` | Lớn hơn / Lớn hơn hoặc bằng | `findByPriceGreaterThan(minPrice)` | `WHERE price > ?` |
| `After`, `Before` | Sau / Trước mốc thời gian | `findByCreatedAtAfter(dateTime)` | `WHERE created_at > ?` |
| `IsNull`, `IsNotNull` | Kiểm tra NULL | `findByDescriptionIsNull()` | `WHERE description IS NULL` |
| `Like` | Khớp mẫu với `%` | `findByNameLike(namePattern)` | `WHERE name LIKE ?` |
| `StartingWith` | Bắt đầu với chuỗi | `findByNameStartingWith(prefix)` | `WHERE name LIKE 'prefix%'` |
| `EndingWith` | Kết thúc với chuỗi | `findByNameEndingWith(suffix)` | `WHERE name LIKE '%suffix'` |
| `Containing` | Chứa chuỗi con (tìm kiếm mờ) | `findByNameContaining(keyword)` | `WHERE name LIKE '%keyword%'` |
| `IgnoreCase` | Không phân biệt hoa/thường | `findByNameIgnoreCase(name)` | `WHERE UPPER(name) = UPPER(?)` |
| `OrderBy...Asc/Desc` | Sắp xếp kết quả | `findByIsAvailableTrueOrderByPriceDesc()` | `WHERE is_available = 1 ORDER BY price DESC` |
| `Not`, `NotIn` | Phủ định / Không thuộc tập | `findByCategoryNotIn(categories)` | `WHERE category NOT IN (?, ?)` |
| `True`, `False` | So sánh giá trị boolean | `findByIsAvailableTrue()` | `WHERE is_available = 1` |

### 3. Hỗ trợ phân trang (Pageable) và sắp xếp (Sort)

Ngoài việc nhúng `OrderBy` trực tiếp vào tên hàm, Spring Data JPA cho phép truyền tham số `Pageable` hoặc `Sort` vào tham số cuối cùng của phương thức để linh hoạt điều khiển phân trang và thứ tự sắp xếp từ Controller/Service:

```java
// Sắp xếp động
List<FoodItem> findByIsAvailableTrue(Sort sort);

// Phân trang với Page<T>: Tự động thực hiện 2 câu lệnh (1 câu lấy dữ liệu phân trang, 1 câu count tổng số bản ghi)
Page<FoodItem> findByCategory(String category, Pageable pageable);

// Phân trang với Slice<T>: Chỉ kiểm tra có trang tiếp theo không, không chạy count query (tối ưu cho Infinite Scroll)
Slice<FoodItem> findByNameContaining(String keyword, Pageable pageable);

// Giới hạn số lượng bản ghi bằng từ khóa Top / First
List<FoodItem> findTop5ByOrderByPriceDesc();
Optional<FoodItem> findFirstByOrderByCreatedAtDesc();
```

### 4. Ví dụ thực tế

Áp dụng vào đề tài dịch vụ bắp nước rạp chiếu phim (`FoodItemRepository`):

```java
@Repository
public interface FoodItemRepository extends JpaRepository<FoodItem, Integer> {

    // 1. Tìm món ăn theo tên chính xác
    Optional<FoodItem> findByName(String name);

    // 2. Tìm kiếm món ăn có tên chứa từ khóa (tìm kiếm mờ, không phân biệt chữ hoa/thường)
    List<FoodItem> findByNameContainingIgnoreCase(String keyword);

    // 3. Tìm các món bắp/nước đang mở bán và có giá từ minPrice đến maxPrice
    List<FoodItem> findByIsAvailableTrueAndPriceBetween(BigDecimal minPrice, BigDecimal maxPrice);

    // 4. Lấy danh sách món theo danh mục (Combo, Bắp, Nước), sắp xếp giảm dần theo giá
    List<FoodItem> findByCategoryOrderByPriceDesc(String category);

    // 5. Kiểm tra xem một món có tồn tại và đang mở bán hay không
    boolean existsByNameAndIsAvailableTrue(String name);

    // 6. Đếm số lượng món ăn thuộc một danh mục cụ thể
    long countByCategory(String category);
}
```

### 5. Ưu điểm và nhược điểm

- **Ưu điểm:**
  - Viết code cực nhanh, dễ đọc, không cần viết câu lệnh SQL/JPQL nào.
  - An toàn lúc khởi động (**Fail-fast**): Nếu bạn viết sai tên thuộc tính của Entity (ví dụ: `findByPrise` thay vì `findByPrice`), Spring Boot sẽ ném ngoại lệ và dừng khởi động ứng dụng ngay lập tức, giúp phát hiện lỗi sớm.
- **Nhược điểm:**
  - Tên hàm trở nên dài dòng và khó đọc khi có nhiều điều kiện (`findByIsAvailableTrueAndCategoryAndPriceLessThanEqualOrderByCreatedAtDesc`).
  - Không hỗ trợ các câu truy vấn phức tạp: phép `JOIN` nhiều bảng tùy biến, `GROUP BY`, `HAVING`, hoặc các phép tính toán (`SUM`, `AVG`).

> **Quy tắc thực tế:** Chỉ nên dùng Derived Query cho các thao tác CRUD cơ bản hoặc truy vấn có từ 1 đến 2 điều kiện đơn giản. Khi có từ 3 điều kiện trở lên hoặc cần JOIN bảng, nên chuyển sang dùng `@Query`.

---

## III. JPQL với @Query

### 1. Khái niệm JPQL

**JPQL (Java Persistence Query Language)** là ngôn ngữ truy vấn hướng đối tượng của JPA. 

- **Điểm khác biệt cốt lõi giữa JPQL và SQL thuần:**
  - **SQL:** Thao tác trên **bảng (tables)** và **cột (columns)** của cơ sở dữ liệu.
  - **JPQL:** Thao tác trên **Entity (Class Java)** và **Field (thuộc tính Java)**.
  
Ví dụ so sánh:
- SQL: `SELECT * FROM orders WHERE total_amount > 100000;`
- JPQL: `SELECT o FROM Order o WHERE o.totalAmount > 100000;` *(Trong đó `Order` là tên class Entity, `totalAmount` là tên thuộc tính Java).*

### 2. Truyền tham số trong JPQL

Có 2 cách truyền tham số vào câu lệnh `@Query`:

- **Cách 1: Positional Parameter (Theo vị trí - `?1`, `?2`)**
  ```java
  @Query("SELECT o FROM Order o WHERE o.status = ?1 AND o.totalAmount >= ?2")
  List<Order> findOrders(OrderStatus status, BigDecimal minAmount);
  ```
  *Hạn chế:* Dễ xảy ra lỗi logic nếu bạn đổi thứ tự tham số trong hàm mà quên sửa số thứ tự trong câu query.

- **Cách 2: Named Parameter (Theo tên biến - `:paramName`) - KHUYÊN DÙNG:**
  ```java
  @Query("SELECT o FROM Order o WHERE o.status = :status AND o.totalAmount >= :minAmount")
  List<Order> findOrders(@Param("status") OrderStatus status, @Param("minAmount") BigDecimal minAmount);
  ```
  *Lợi ích:* Code tường minh, rõ ràng, không phụ thuộc vào thứ tự truyền tham số trong phương thức.

### 3. Các kỹ thuật quan trọng trong JPQL

- **JOIN FETCH (Giải quyết triệt để vấn đề N+1 Query):**
  Khi cần lấy `Order` kèm theo toàn bộ `orderDetails`, nếu dùng quan hệ `LAZY`, mỗi lần truy cập `orderDetails` của một order thì Hibernate sẽ bắn thêm 1 câu query con (N+1 query problem). `JOIN FETCH` ép Hibernate thực hiện một câu SQL JOIN duy nhất để tải toàn bộ dữ liệu liên quan ngay lập tức:
  ```java
  @Query("SELECT o FROM Order o LEFT JOIN FETCH o.orderDetails WHERE o.id = :orderId")
  Optional<Order> findOrderWithDetailsById(@Param("orderId") Integer orderId);
  ```

- **Hàm tổng hợp (Aggregate Functions):**
  Hỗ trợ đầy đủ các hàm thống kê: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
  ```java
  @Query("SELECT SUM(o.totalAmount) FROM Order o WHERE o.createdAt BETWEEN :from AND :to")
  BigDecimal calculateTotalRevenue(@Param("from") LocalDateTime from, @Param("to") LocalDateTime to);
  ```

### 4. Cập nhật và xóa dữ liệu với @Modifying

Mặc định, `@Query` chỉ dùng cho các thao tác đọc (`SELECT`). Khi muốn thực hiện `UPDATE` hoặc `DELETE` bằng JPQL, bạn bắt buộc phải thêm annotation `@Modifying` và `@Transactional`:

```java
@Modifying(clearAutomatically = true)
@Transactional
@Query("UPDATE Order o SET o.status = :newStatus WHERE o.id IN :orderIds")
int updateOrderStatusBatch(@Param("orderIds") List<Integer> orderIds, @Param("newStatus") OrderStatus newStatus);
```

> **Tại sao cần `clearAutomatically = true`?**
> Khi chạy câu lệnh UPDATE trực tiếp xuống CSDL thông qua `@Modifying`, bộ nhớ đệm (Persistence Context / First-level cache) của Hibernate sẽ không tự động đồng bộ lại các thay đổi này. Việc đặt `clearAutomatically = true` sẽ tự động xóa sạch cache sau khi thực thi câu query, đảm bảo các lần gọi dữ liệu tiếp theo sẽ đọc giá trị mới nhất từ CSDL.

### 5. Ví dụ thực tế

Áp dụng vào hệ thống quản lý rạp chiếu phim (`OrderRepository`):

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Integer> {

    // 1. Tìm đơn hàng theo trạng thái và có tổng tiền lớn hơn mức chỉ định
    @Query("SELECT o FROM Order o WHERE o.status = :status AND o.totalAmount >= :minTotal")
    List<Order> findLargeOrdersByStatus(
        @Param("status") String status, 
        @Param("minTotal") BigDecimal minTotal
    );

    // 2. Tải toàn bộ thông tin chi tiết đơn hàng kèm món ăn để in hóa đơn (tránh N+1)
    @Query("SELECT o FROM Order o " +
           "LEFT JOIN FETCH o.orderDetails d " +
           "LEFT JOIN FETCH d.foodItem " +
           "WHERE o.id = :orderId")
    Optional<Order> findOrderForReceipt(@Param("orderId") Integer orderId);

    // 3. Đếm số lượng đơn hàng hoàn thành trong ngày
    @Query("SELECT COUNT(o) FROM Order o WHERE o.status = 'COMPLETED' AND o.createdAt >= :startOfDay")
    long countCompletedOrdersToday(@Param("startOfDay") LocalDateTime startOfDay);

    // 4. Hủy các đơn hàng chưa thanh toán quá hạn (UPDATE hàng loạt)
    @Modifying(clearAutomatically = true)
    @Transactional
    @Query("UPDATE Order o SET o.status = 'CANCELLED' WHERE o.status = 'PENDING' AND o.createdAt < :expireTime")
    int cancelExpiredPendingOrders(@Param("expireTime") LocalDateTime expireTime);
}
```

### 6. Ưu điểm và nhược điểm

- **Ưu điểm:**
  - Hoàn toàn độc lập với hệ quản trị CSDL (Database-independent): Bạn có thể đổi từ MySQL sang PostgreSQL hoặc Oracle mà không cần viết lại câu truy vấn JPQL.
  - Hỗ trợ đầy đủ mô hình hướng đối tượng, dễ dàng map kết quả vào các Entity liên kết.
  - Tránh được lỗi SQL Injection vì luôn sử dụng bind parameters.
- **Nhược điểm:**
  - Không tận dụng được các hàm hoặc cú pháp đặc thù riêng biệt của từng hệ quản trị CSDL (ví dụ: các hàm xử lý chuỗi JSON, Window functions của PostgreSQL/MySQL 8).
  - Khó kiểm soát hoàn toàn câu lệnh SQL cuối cùng mà Hibernate sinh ra để tối ưu hóa hiệu năng chi tiết (index hint, execution plan).

---

## IV. Native query (SQL thuần)

### 1. Khái niệm

**Native query** là việc viết câu lệnh SQL thuần túy (SQL dialect) dành riêng cho hệ quản trị CSDL mà ứng dụng đang kết nối (MySQL, MariaDB, PostgreSQL, SQL Server...).

Để kích hoạt chế độ Native query trong Spring Data JPA, ta đặt thuộc tính `nativeQuery = true` bên trong annotation `@Query`:

```java
@Query(value = "SELECT * FROM orders WHERE total_amount > :amount", nativeQuery = true)
List<Order> findOrdersNative(@Param("amount") BigDecimal amount);
```

> **Lưu ý khác biệt lớn:** Trong Native Query, bạn phải dùng **tên bảng thật** và **tên cột thật** trong CSDL (`orders`, `total_amount`, `order_detail`), không được dùng tên class hay thuộc tính Java của Entity.

### 2. Cách hứng dữ liệu (Mapping Result)

Khi viết Native Query, có 2 cách chính để hứng kết quả trả về:

#### a. Trả về Entity
- Chỉ áp dụng khi câu truy vấn `SELECT *` hoặc lấy **đầy đủ tất cả các cột** của bảng tương ứng với Entity.
- Nếu bạn chỉ `SELECT id, total_amount` nhưng Entity có 10 trường, Hibernate sẽ báo lỗi không thể map kết quả.

```java
@Query(value = "SELECT * FROM orders WHERE status = :status ORDER BY created_at DESC LIMIT :limit", nativeQuery = true)
List<Order> findRecentOrdersNative(@Param("status") String status, @Param("limit") int limit);
```

#### b. Trả về Projection (Interface-based Projection) - KHUYÊN DÙNG cho báo cáo / thống kê
Khi câu truy vấn JOIN nhiều bảng hoặc chỉ lấy một vài trường để làm báo cáo, ta định nghĩa một Java Interface chứa các getter tương ứng với tên alias trong câu SQL:

```java
// Interface định nghĩa các trường cần lấy
public interface TopSellingFoodProjection {
    Integer getFoodId();
    String getFoodName();
    Integer getTotalQuantity();
    BigDecimal getTotalRevenue();
}
```

```java
// Trong Repository: Tên cột (alias) trong câu SQL phải khớp với tên hàm getter (foodId -> getFoodId)
@Query(value = """
    SELECT 
        f.id AS foodId, 
        f.name AS foodName, 
        SUM(od.quantity) AS totalQuantity, 
        SUM(od.quantity * od.price) AS totalRevenue
    FROM order_detail od
    INNER JOIN food_items f ON od.food_item_id = f.id
    INNER JOIN orders o ON od.order_id = o.id
    WHERE o.status = 'COMPLETED'
    GROUP BY f.id, f.name
    ORDER BY totalQuantity DESC
    LIMIT :topN
    """, nativeQuery = true)
List<TopSellingFoodProjection> findTopSellingFoods(@Param("topN") int topN);
```

### 3. Khi nào nên dùng Native Query

Bạn nên chuyển sang dùng Native Query trong các trường hợp sau:

1. **Sử dụng các hàm chuyên biệt của hệ quản trị CSDL (Vendor-specific functions):**
   - MySQL: `DATE_FORMAT()`, `JSON_EXTRACT()`, `GROUP_CONCAT()`, `FIND_IN_SET()`.
   - PostgreSQL: Các toán tử JSONB (`->`, `->>`), kiểu dữ liệu mảng, hàm Full-text search (`to_tsvector`).
2. **Cần tối ưu hóa hiệu năng tối đa:**
   - Khi cần can thiệp trực tiếp vào execution plan, thêm chỉ dẫn Index (`USE INDEX`, `FORCE INDEX`), hoặc xử lý các câu truy vấn báo cáo nặng hàng triệu bản ghi.
3. **Các câu truy vấn quá phức tạp:**
   - Sử dụng đệ quy (Recursive CTE - `WITH RECURSIVE`), Window Functions (`ROW_NUMBER()`, `RANK()`, `LEAD()`, `LAG()`) mà JPQL không hỗ trợ hoặc viết rất cồng kềnh.

### 4. Ví dụ thực tế

Áp dụng vào nghiệp vụ rạp chiếu phim: Thống kê doanh thu bắp nước theo từng tháng trong năm:

```java
public interface MonthlyRevenueProjection {
    Integer getMonth();
    BigDecimal getRevenue();
    Long getTotalOrders();
}
```

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Integer> {

    // Thống kê doanh thu theo từng tháng bằng hàm MONTH() của MySQL
    @Query(value = """
        SELECT 
            MONTH(created_at) AS month, 
            SUM(total_amount) AS revenue,
            COUNT(id) AS totalOrders
        FROM orders
        WHERE YEAR(created_at) = :year AND status = 'COMPLETED'
        GROUP BY MONTH(created_at)
        ORDER BY month ASC
        """, nativeQuery = true)
    List<MonthlyRevenueProjection> getMonthlyRevenueStatistics(@Param("year") int year);
}
```

### 5. Ưu điểm, nhược điểm và lưu ý quan trọng

- **Ưu điểm:**
  - Tận dụng 100% sức mạnh và tính năng riêng biệt của cơ sở dữ liệu.
  - Hiệu năng tối đa, câu lệnh được gửi thẳng xuống database mà không cần Hibernate biên dịch từ JPQL sang SQL.
- **Nhược điểm:**
  - **Mất tính linh hoạt CSDL (Vendor lock-in):** Nếu chuyển từ MySQL sang PostgreSQL hoặc Oracle, câu lệnh Native SQL có thể bị lỗi cú pháp và phải viết lại.
  - **Không có Type-safety:** Không kiểm tra lỗi tên bảng, tên cột lúc compile code; chỉ phát hiện lỗi khi câu lệnh được thực thi tại runtime.
  - **Phân trang phức tạp hơn:** Nếu dùng phân trang (`Pageable`) với Native Query phức tạp có `GROUP BY`, bạn phải tự viết thêm câu lệnh đếm tổng bản ghi thông qua thuộc tính `countQuery`:
    ```java
    @Query(value = "SELECT * FROM orders WHERE status = :status",
           countQuery = "SELECT count(*) FROM orders WHERE status = :status",
           nativeQuery = true)
    Page<Order> findOrdersWithPagination(@Param("status") String status, Pageable pageable);
    ```

---

## V. Truy vấn sử dụng Entity manager

### 1. Khái niệm và vai trò của EntityManager

`EntityManager` là giao diện trung tâm (Core Interface) của tiêu chuẩn JPA (gói `jakarta.persistence.*`). 

Trong kiến trúc Spring Boot:
```
Spring Data JPA (Repository)  --->  EntityManager  --->  Hibernate Session  --->  JDBC / Database
```
Thực chất, các `JpaRepository` mà bạn dùng hàng ngày chỉ là một lớp bọc tiện ích bên ngoài. Mọi hành động lưu, sửa, xóa, tìm kiếm của Spring Data JPA bên dưới đều ủy quyền (delegate) cho `EntityManager` thực hiện.

`EntityManager` trực tiếp quản lý **Persistence Context** (Bộ nhớ đệm cấp 1 - First-level Cache) và kiểm soát 4 trạng thái vòng đời của một Entity:
1. **New (Transient):** Đối tượng vừa được tạo bằng từ khóa `new`, chưa liên kết với CSDL, chưa có ID.
2. **Managed:** Đối tượng đang được `EntityManager` theo dõi và quản lý. Mọi thay đổi trên đối tượng sẽ tự động đồng bộ xuống CSDL khi transaction kết thúc (Dirty Checking).
3. **Detached:** Đối tượng đã từng ở trạng thái Managed nhưng Persistence Context đã bị đóng hoặc bị tách ra (`em.detach()`, `em.clear()`).
4. **Removed:** Đối tượng được đánh dấu sẽ bị xóa khỏi CSDL khi transaction commit (`em.remove()`).

### 2. Cách inject EntityManager trong Spring Boot

Trong Spring Boot, bạn có thể inject `EntityManager` vào bất kỳ Service hoặc Custom Repository nào bằng annotation `@PersistenceContext`:

```java
@Repository
public class CustomFoodRepositoryImpl {

    @PersistenceContext
    private EntityManager entityManager;

    // ...
}
```

### 3. Các phương thức truy vấn và thao tác cốt lõi

- **`em.find(Class<T> entityClass, Object primaryKey)`**: Tìm kiếm bản ghi theo khóa chính. Rất nhanh vì nó kiểm tra trong bộ nhớ đệm (First-level Cache) trước, nếu đã có thì trả về ngay mà không cần query lại CSDL.
- **`em.createQuery(String jpql, Class<T> resultClass)`**: Tạo truy vấn JPQL động trả về `TypedQuery<T>` an toàn kiểu.
- **`em.createNativeQuery(String sql)`**: Tạo câu truy vấn SQL thuần tùy biến.
- **`em.persist(entity)`**: Đưa một thực thể mới vào trạng thái Managed để lưu vào CSDL.
- **`em.merge(entity)`**: Đưa một thực thể Detached trở lại trạng thái Managed (thường dùng khi cập nhật).
- **`em.remove(entity)`**: Xóa thực thể khỏi CSDL.
- **`em.flush()`**: Ép đẩy ngay lập tức tất cả các thay đổi đang nằm trong bộ nhớ đệm xuống CSDL (nhưng chưa commit transaction).
- **`em.clear()`**: Xóa sạch toàn bộ thực thể trong Persistence Context, đưa tất cả về trạng thái Detached.

### 4. Ứng dụng thực tế của EntityManager

Có 2 kịch bản thực tế mà `JpaRepository` thông thường không đáp ứng tốt, bắt buộc ta nên dùng `EntityManager`:

#### a. Kịch bản 1: Xây dựng bộ lọc tìm kiếm động (Dynamic Query / Multi-criteria Filter)
Giả sử người dùng trên giao diện rạp phim tìm kiếm món ăn với nhiều bộ lọc tùy chọn: có thể nhập tên, có thể chọn khoảng giá, có thể chọn danh mục hoặc không. Nếu dùng Repository thông thường, bạn sẽ phải viết hàng chục hàm cho mọi tổ hợp điều kiện. Với `EntityManager`, ta có thể ghép điều kiện động cực kỳ linh hoạt:

```java
@Repository
public class FoodItemCustomRepository {

    @PersistenceContext
    private EntityManager em;

    public List<FoodItem> searchFoodItems(String keyword, String category, BigDecimal minPrice, BigDecimal maxPrice) {
        StringBuilder jpql = new StringBuilder("SELECT f FROM FoodItem f WHERE 1=1 ");
        Map<String, Object> params = new HashMap<>();

        if (keyword != null && !keyword.trim().isEmpty()) {
            jpql.append("AND LOWER(f.name) LIKE LOWER(:keyword) ");
            params.put("keyword", "%" + keyword.trim() + "%");
        }

        if (category != null && !category.trim().isEmpty()) {
            jpql.append("AND f.category = :category ");
            params.put("category", category);
        }

        if (minPrice != null) {
            jpql.append("AND f.price >= :minPrice ");
            params.put("minPrice", minPrice);
        }

        if (maxPrice != null) {
            jpql.append("AND f.price <= :maxPrice ");
            params.put("maxPrice", maxPrice);
        }

        TypedQuery<FoodItem> query = em.createQuery(jpql.toString(), FoodItem.class);
        params.forEach(query::setParameter);

        return query.getResultList();
    }
}
```

#### b. Kịch bản 2: Xử lý thêm mới/cập nhật dữ liệu hàng loạt (Batch Processing)
Khi bạn cần import hàng ngàn sản phẩm hoặc cập nhật hàng vạn bản ghi đơn hàng, nếu dùng `repository.saveAll()`, Hibernate sẽ giữ toàn bộ hàng ngàn Entity đó trong bộ nhớ RAM (Persistence Context), rất dễ dẫn đến lỗi **OutOfMemoryError** tràn bộ nhớ.

Giải pháp với `EntityManager`: Định kỳ gọi `em.flush()` để đẩy dữ liệu xuống DB và gọi `em.clear()` để giải phóng RAM:

```java
@Service
@RequiredArgsConstructor
public class FoodBatchService {

    @PersistenceContext
    private final EntityManager em;

    @Transactional
    public void batchInsertFoodItems(List<FoodItem> items) {
        int batchSize = 50; // Kích thước mỗi đợt flush
        
        for (int i = 0; i < items.size(); i++) {
            em.persist(items.get(i));
            
            // Cứ mỗi 50 bản ghi thì đẩy xuống DB và xóa bộ nhớ cache để giải phóng RAM
            if (i > 0 && i % batchSize == 0) {
                em.flush(); // Đồng bộ dữ liệu xuống database
                em.clear(); // Xóa sạch bộ nhớ đệm cấp 1 để tránh tràn RAM
            }
        }
        em.flush();
        em.clear();
    }
}
```

### 5. Khi nào nên dùng EntityManager

- **NÊN DÙNG:**
  - Xây dựng chức năng tìm kiếm động nhiều tiêu chí linh hoạt (Search Filter).
  - Xử lý các tác vụ Batch Processing với dữ liệu lớn để kiểm soát bộ nhớ RAM.
  - Khi cần can thiệp sâu vào vòng đời Entity và cơ chế Dirty Checking của JPA.
- **KHÔNG NÊN DÙNG:**
  - Các thao tác CRUD thông thường: Việc viết code với `EntityManager` tốn nhiều dòng code và thủ công hơn nhiều so với việc chỉ cần khai báo một interface kế thừa `JpaRepository`.

---

## VI. Bảng so sánh tổng hợp 4 phương pháp truy vấn

| Tiêu chí | 1. Derived Query Method | 2. JPQL với `@Query` | 3. Native Query | 4. EntityManager |
| :--- | :--- | :--- | :--- | :--- |
| **Cách định nghĩa** | Đặt tên phương thức theo quy ước trong Interface | Viết chuỗi JPQL trên Entity trong `@Query` | Viết chuỗi SQL thuần với `nativeQuery = true` | Gọi trực tiếp qua code Java bằng `EntityManager` |
| **Đối tượng thao tác** | Tên thuộc tính của Entity | Entity và Field Java | Bảng và Cột thật trong CSDL | Entity, JPQL động hoặc Criteria API |
| **Mức độ tiện lợi** | Cực kỳ nhanh, ngắn gọn nhất | Linh hoạt, cú pháp quen thuộc | Viết SQL tự nhiên theo DBMS | Phải viết code thủ công nhiều nhất |
| **Độc lập CSDL** | Có (100% độc lập) | Có (100% độc lập) | **Không** (Dính chặt vào CSDL đang dùng) | Có (nếu dùng JPQL / Criteria) |
| **Type-safety** | Kiểm tra lúc khởi động ứng dụng | Kiểm tra cú pháp cơ bản lúc khởi động | **Không** (Lỗi chỉ xuất hiện tại runtime) | An toàn nếu dùng `TypedQuery` hoặc Criteria API |
| **Xử lý truy vấn phức tạp** | Kém (chỉ phù hợp 1-2 điều kiện) | Tốt (JOIN, FETCH, Aggregate, Group By) | Rất tốt (Mọi tính năng DBMS, Window Func, CTE) | Tối ưu cho truy vấn động (Dynamic query) |
| **Nguy cơ lỗi N+1** | Dễ gặp nếu quan hệ là LAZY | Khắc phục triệt để bằng `JOIN FETCH` | Không áp dụng trực tiếp cơ chế FETCH | Tự kiểm soát được fetch join trong câu lệnh |
| **Trường hợp khuyên dùng** | CRUD đơn giản, tìm kiếm 1-2 điều kiện cơ bản | Nghiệp vụ trung bình đến phức tạp, cần JOIN bảng, tính toán | Thống kê báo cáo nặng, dùng hàm riêng của CSDL, tối ưu index | Bộ lọc tìm kiếm động nhiều điều kiện, Batch processing lớn |

### Lời khuyên thực chiến cho dự án (Best Practices):
1. **Bước 1:** Bắt đầu bằng **Derived Query** cho tất cả các thao tác CRUD cơ bản hoặc tìm kiếm đơn giản.
2. **Bước 2:** Chuyển sang **JPQL (`@Query`)** khi tên phương thức bắt đầu dài dòng (quá 2 điều kiện) hoặc khi cần `JOIN FETCH` để dập tắt lỗi N+1 Query.
3. **Bước 3:** Sử dụng **Native Query** khi cần viết câu truy vấn thống kê, báo cáo phức tạp hoặc cần dùng các hàm đặc thù của hệ quản trị CSDL mà JPQL không hỗ trợ.
4. **Bước 4:** Sử dụng **EntityManager** cho các màn hình lọc động nhiều tiêu chí tùy chọn và khi xử lý dữ liệu lớn (Batch Processing).