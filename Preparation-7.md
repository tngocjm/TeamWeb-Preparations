# Buổi 7: CRUD cơ bản

## Mục lục

- [Buổi 7: CRUD cơ bản](#buổi-7-crud-cơ-bản)
  - [Mục lục](#mục-lục)
  - [I. Triển khai RestfullAPI trong Spring Boot](#i-triển-khai-restfullapi-trong-spring-boot)
  - [II. JPA là gì? Hibernate là gì?](#ii-jpa-là-gì-hibernate-là-gì)
    - [1. JPA là gì?](#1-jpa-là-gì)
    - [2. Hibernate là gì?](#2-hibernate-là-gì)
  - [III. Kiến trúc 4 lớp Spring Boot](#iii-kiến-trúc-4-lớp-spring-boot)
  - [IV. Request flow (dòng chảy xử lý request)](#iv-request-flow-dòng-chảy-xử-lý-request)
  - [V. Cấu hình DataSource](#v-cấu-hình-datasource)
  - [VI. Tìm hiểu các Annotation : @RestController, @Service, @Repository, @Entity, @Table, @Id, @Transactional, @Query ...](#vi-tìm-hiểu-các-annotation--restcontroller-service-repository-entity-table-id-transactional-query-)
    - [1. @RestController](#1-restcontroller)
    - [2. @Service](#2-service)
    - [3. @Repository](#3-repository)
    - [4. @Entity, @Table, @Id, @GeneratedValue, @Column](#4-entity-table-id-generatedvalue-column)
    - [5. @Transactional](#5-transactional)
    - [6. @Query](#6-query)

---

## I. Triển khai RestfullAPI trong Spring Boot



## II. JPA là gì? Hibernate là gì?

ORM - Object Relational Mapping - Ánh xạ đối tượng trong Java với dữ liệu trong cơ sở dữ liệu quan hệ

- Nếu không dùng ORM: 

Database ta có bảng users
|id|name|age|
|---|---|---|
|1|Mai|20|
|2|Ngọc|17|

Nếu muốn lấy dữ liệu từ database với id = 1, thì ta cần phải viết câu lệnh SQL
```SQL
SELECT * FROM users WHERE id = 1
```

Sau đó tự lấy dữ liệu và chuyển thành Java object

```java
User user = new User;
user.setId(1);
user.setName("Mai");
user.setAge(20);
```

- Nếu dùng ORM:

Bạn có class:
```java
@Entity
@Table (name = users)
public class User{
    @Id
    private Integer id;

    private String name;

    private Integer age;
}
```

Lúc đó ORM sẽ hiểu:

|Java|________|Database|
|---|---|---|
|User||users|
|id||id|
|name||name|
|age||age|

Khi viết:
```java
User user = userRepository.findById(1).get();
```

thì phía dưới ORM sẽ thực hiện tương đương:

```SQL
SELECT * FROM users WHERE id = 1;
```

rồi tự động biến kết quả thành user


### 1. JPA là gì?

***JPA là specification (đặc tả) quy định cách Java ORM nên hoạt động***

Cho phép lập trình viên code dựa trên các annotation tiêu chuẩn như @Entity, @Id, @OneToMany,…


Ưu điểm:

Code tuân theo chuẩn, dễ bảo trì, dễ thay đổi thư viện ORM nếu cần

Tập trung vào logic thay vì viết SQL thủ công


### 2. Hibernate là gì?

***Hibernate là một ORM framework, đồng thời là một implementation phổ biến của JPA.***

> Điều này cho phép các lập trình viên làm việc với cơ sở dữ liệu bằng cách sử dụng các đối tượng Java, thay vì phải viết các câu lệnh SQL thủ công.

Hibernate cung cấp nhiều tính năng mạnh mẽ như:

Hỗ trợ các mối quan hệ phức tạp giữa các thực thể (one-to-one, one-to-many, many-to-many).

Quản lý bộ nhớ đệm (caching) để cải thiện hiệu suất truy xuất dữ liệu.

Sử dụng Hibernate Query Language (HQL) cho phép viết các truy vấn một cách linh hoạt và dễ dàng.

Hỗ trợ cơ chế nạp dữ liệu lười (lazy loading) và nạp dữ liệu tức thì (eager loading) để tối ưu hóa việc truy xuất dữ liệu.

## III. Kiến trúc 4 lớp Spring Boot

Kiến trúc bao gồm 4 lớp:

| Lớp | Thành phần Spring Boot | Nhiệm vụ |
|---|---|---|
| Presentation Layer | Controller | Nhận HTTP Request và trả HTTP Response |
| Business Layer | Service | Xử lý logic/nghiệp vụ của ứng dụng |
| Data Access Layer | Repository | Truy cập và thao tác với dữ liệu trong Database |
| Domain/Model Layer | Entity | Mô hình hóa các đối tượng và dữ liệu của hệ thống |


## IV. Request flow (dòng chảy xử lý request)

![alt text](image.png)

- Client gửi Request

- Controller nhận request, gọi Service để xử lí logic

- Service xử lí nghiệp vụ và logic, nếu muốn sử dụng dữ liệu thì gọi Repository

- Repository lấy dữ liệu bằng cách làm việc với database thông qua JPA/Hybernate

- Sau đó dữ liệu quay trở lại Controller, Spring đọc thấy @RestController, Spring sẽ chuyển object thành dạng JSON

- Trả Respone về cho client

## V. Cấu hình DataSource
Cho Spring Boot biết cách kết nối và lấy connection tới Database.

├── DataSource

│   ├── URL

│   ├── Username

│   ├── Password

│   ├── JDBC Driver

│   └── Connection Pool

Bao gồm các thành phần:

|Thành phần|Ý nghĩa|
|---|---|
|url|Database nằm ở đâu, kết nối bằng cách nào|
|username|Tài khoản database|
|password|Mật khẩu database|
|driver-class-name|JDBC Driver dùng để kết nối|

## VI. Tìm hiểu các Annotation : @RestController, @Service, @Repository, @Entity, @Table, @Id, @Transactional, @Query ...

### 1. @RestController

@RestController = @Controller + @RestsponeBody

```java
@RestController
public class UserController{
}
```

> Tự động serialize object (tuần tự hóa đối tượng - quá trình chuyển 1 object thành 1 dạng dữ liệu có thể lưu trữ hoặc truyền đi) thành JSON

* Khác biệt cốt lõi:

@Controller -> render view

@RestController -> trả dữ liệu

### 2. @Service

Thực chất là một specialization của @Component

Spring không xử lý khác về mặt kỹ thuật.

Nhưng về mặt kiến trúc, đánh dấu một Class là tầng Service, phục vụ các logic nghiệp vụ.

```java
@Service
public class UserService{
}
```

* Service nên chứa:

Chứa business logic → xử lý các quy tắc nghiệp vụ.

Điều phối Repository → gọi một hoặc nhiều Repository để thực hiện nghiệp vụ.

Xử lý Transaction → đảm bảo nhóm thao tác database thành công toàn bộ hoặc rollback khi có lỗi.

* Service không nên:

Xử lí HTTP

Truy vấn trực tiếp database bằng SQL

### 3. @Repository

Đánh dấu một Class Là tầng Repository, phục vụ truy xuất dữ liệu.

Ngoài việc đăng ký bean, @Repository kích hoạt cơ chế:

***Exception Translation***

Spring bắt các exception của JDBC / JPA và chuyển thành DataAccessException.

```java
@Repository
public class UserRepository { }
```

Lợi ích:

Không phụ thuộc vào loại database

Hệ thống exception thống nhất

Repository chỉ nên làm việc với dữ liệu (CRUD)

Không nên chứa business logic (công việc của tầng service)

### 4. @Entity, @Table, @Id, @GeneratedValue, @Column

@Entity

Đánh dấu class là Entity được JPA quản lý và ánh xạ với dữ liệu trong database.

@Table

Định nghĩa tên bảng.

@Id

Khóa chính.

@Column 

Cho phép bạn tùy chỉnh tên cột trong bảng cơ sở dữ liệu và các thuộc tính của cột đó như kiểu dữ liệu, độ dài, v.v.


```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
e
    @Column(name = "ten_sinh_vien", length = 100)
    private String name;

    private String email;
}
```

### 5. @Transactional

Nếu chúng ta không khai báo rollback rules thì transaction sẽ chỉ rool back khi có exception của RuntimeException và Error. Còn các loại Checked Exceptions sẽ không được rollback

```java
@GetMapping("/get2")
@Transactional(rollbackFor = {Exception.class, Throwable.class})
public void get2() throws Exception {
    TestEntity testEntity = new TestEntity();
    testRepository.save(testEntity);
    throw new Exception("test");
}
```

### 6. @Query

Với cách sử dụng @Query, bạn sẽ có thể sử dụng câu truy vấn JPQL (Hibernate) hoặc raw SQL.

```java
public interface UserRepository extends JpaRepository<User, Long> {

    //dùng JPQL
    @Query("select u from User u where u.emailAddress = ?1")
    User myCustomQuery(String emailAddress);

    //Native SQL.
    @Query(value = "select * from User u where u.email_address = ?1", nativeQuery = true)
    User myCustomQuery2(String emailAddress);
}
```

JPQL: làm việc với entity và các thuộc tính của entity chứ không làm việc với các bảng

NativeSQL: nativeQuery = true, Springboot hiểu là đây là sql thật, không phải là JPQL

> Các query phức tạp mình sẽ sử dụng @query, còn không sẽ sử dụng các query method
