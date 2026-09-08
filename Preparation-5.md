# Buổi 5: Spring Boot Cơ Bản

## I. Nội dung bài học 
### 1. HTTP là gì ? Các method trong HTTP
### 2. Response là gì, Request là gì ?
### 3. API là gì, RestAPI là gì ?
### 4. Design pattern : DI, IOC

## II. HTTP là gì? Các method trong HTTP

### 1. Khái niệm HTTP

HTTP là giao thức truyền tải siêu văn bản, là ngôn ngữ/quy tắc để trình duyệt (web client) và web server (máy chủ - cung cấp nội dung, dịch vụ web) nói chuyện với nhau, và nó thuộc tầng ứng dụng của TCP/IP.

![alt text](image.png)

### 2. Method trong HTTP

Một yêu cầu xác định một phương thức (đôi khi gọi không chính thức là động từ) để phân loại hành động mong muốn được thực hiện trên một tài nguyên (Request Method), các method dùng để xác định client muốn server làm gì với tài nguyên được yêu cầu?

GET, POST, PUT, PATCH, DELETE là nhóm sẽ gặp thường xuyên nhất.

***GET (Lấy dữ liệu)***

GET được sử dụng để lấy lại thông tin từ máy chủ đã cung cấp bởi sử dụng một URI đã cung cấp. Các yêu cầu sử dụng GET chỉ nhận dữ liệu và không có ảnh hưởng gì tới dữ liệu.

***POST (Thêm dữ liệu)***

Một yêu cầu POST sử dụng các mẫu HTML để gửi dữ liệu tới máy chủ, như thông tin khách hàng, file tải lên,...

POST thường được dùng để gửi dữ liệu lên server để tạo một tài nguyên mới hoặc yêu cầu server thực hiện một xử lý.
VD: đăng kí tài khoản user

***PUT (Cập nhật dữ liệu)***

Thay đổi tất cả các đại diện hiện tại của nguồn mục tiêu với nội dung được tải lên

***PATCH***

Chỉ thay đổi phần tài nguyên mà mình muốn thay đổi

***DELETE (Xóa dữ liệu)***

Gỡ bỏ tất cả các đại diện hiện tại của nguồn mục tiêu bởi URI (Uniform Resource Identifier - 1 chuỗi kí tự để định danh tài nguyên, có thể là một trang web, một file, một API endpoint, một hình ảnh,...)

***HEAD***

Tương tự như GET, nhưng nó truyền tải dòng trạng thái và khu vực header, chỉ lấy thông tin của nội dung
VD: GET /image.jpg -> trả về header (thông tin) + nội dung (hình ảnh)
HEAD /image.jpg -> trả về header (thông tin) -> dùng để lấy thông tin file này có tồn tại k và kích thước là bao nhiêu

***OPTIONS***

Miêu tả các chức năng giao tiếp cho nguồn mục tiêu, có thể hiểu là server sẽ cho tôi biết tài nguyên này sẽ hỗ trợ chức năng gì (VD: get, patch, post,...)

***TRACE***

Trình bày một vòng lặp kiểm tra thông báo song song với path tới nguồn mục tiêu, dùng để debug/kiểm tra


## III. Respone & Request
![alt text](image.png)

### 1. Request (Yêu cầu)

#### a. Chức năng

HTTP request là thông tin được gửi từ client lên server, để yêu cầu server tìm hoặc xử lý một số thông tin, dữ liệu mà client muốn.

#### b. Cấu trúc
```
Request Line (METHOD URI HTTP-VERSION)
Header Request (Request header giúp client có thể gửi yêu cầu lên server. Mỗi yêu cầu sẽ kèm theo các thông số, và các thông số đó được gọi là Header Parameters. Trình duyệt và server sẽ dựa vào các thông số header này để trả dữ liệu và hiển thị dữ liệu cho phù hợp.)

Body Request (Có thể có hoặc k, Request body thường được sử dụng trong các phương thức Post, Put, Patch.)
```

VD:
```
POST /BookStore/v1/Books HTTP/1.1
Host: example.com
Content-Type: application/json
Accept: application/json

{
    "title": "Java cơ bản",
    "author": "Nguyen Van A",
    "price": 50000
}
```

### 2. Respone (Phản hồi)

### a. Chức năng

Response là phần server gửi ngược lại cho client (browser) sau khi server nhận và xử lý request.

### b. Cấu trúc
```
Status Line (HTTP-VERSION STATUS REASON)
Header

Body
```

1 số loại status:

1xx → Thông tin

2xx → Thành công

3xx → Chuyển hướng

4xx → Lỗi từ phía client

5xx → Lỗi phía server

VD:
```
HTTP/1.1 200 OK
Content-Type: text/html

<html>
    <body>
        Hello
    </body>
</html>
```

-> Browser (trình duyệt web) sẽ nhận nó rồi xử lí để hiển thị trang web

## IV. API & Rest API

### 1. API (Application Programming Interface)

#### a. Khái niệm

API là 1 tập hợp các commands, functions, objects, protocol,... giúp 2 phần mềm có thể trao đổi dữ liệu, tương tác qua lại với nhau 

Có thể liên tưởng tới:

***User Interface (UI):***

- Giúp user tương tác với application

- user tương tác với application mà không cần quan tâm application đó xây dựng như thế nào

***Application Programming Interface (API):***

- Giúp application A tương tác với application B

- application a tương tác với app b mà không cần quan tâm app b xây dựng và hiện thực như nào

#### b. VD

Hai ứng dụng tương tác với nhau:

- Bạn Tí: Xây dựng ra 1 hàm tính tổng 'function sum(a, b)'
- Bạn Tèo sử dụng hàm tính tổng của bạn Tí -> Bạn Tí đang cung cấp API tính tổng cho bạn Tèo

Backend tương tác với Frontend:

- Backend: Xây dựng API để lấy danh sách các bài viết 
- Frontend: Sử dụng API của backend để lấy dữ liệu về Frontend

> Không cần quan tâm app cung cấp API xây dựng như thế nào ta chỉ cần quan tâm cách sử dụng, input, output nhận được

### 2. Rest API

#### a. Khái niệm

Rest API là 1 kiểu kiến trúc, 1 tập hợp các quy tắc để đảm bảo việc giao tiếp giữa các application diễn ra theo 1 chuẩn mực dễ đoán và hiệu quả

#### b. 6 nguyên tắc cơ bản của REST

***Stateless (Nguyên tắc vàng - Tính chất không lưu trạng thái):*** Mỗi 1 yêu cầu gửi từ client phải chứa đầy đủ tất cả thông tin mà server cần để xử lý mà không cần phải lưu lại trạng thái request trước đó -> Khả năng mở rộng hệ thống đến vô tận, thêm nhiều máy chủ đáp ứng phục vụ yêu cầu của hàng triệu, tỉ người

***Client – Server:*** Client chỉ cần biết cách gọi API, không cần biết server được xây dựng và hiện thực như nào

***Resource (Tài nguyên) được xác định bằng URI***

***Sử dụng HTTP Method đúng ngữ nghĩa***

***Sử dụng HTTP Status Code***

***Representation:*** Resource có thể biểu diễn bằng nhiều định dạng, phổ biến là JSON

## V. Design pattern (Mẫu thiết kế phần mềm): DI, IOC

### 1. DI - Dependency Injection - đưa sự phụ thuộc vào đối tượng

#### a. Khái niệm

Là kỹ thuật đưa dependency từ bên ngoài vào object thay vì để object tự tạo dependency.

VD: Không dùng DI 

```java
class UserService {

    private UserRepository repository;

    public UserService() {
        repository = new UserRepository();
    }
}
```
> Vấn đề của các này đó là: UserService tự tạo dependency của mình bằng new UserRepository(), UserService bị phụ thuộc chặt vào UserRepositor

```java
class MySQLUserRepository {
}
hoặc
class MockUserRepository {
}
```

> thì phải sửa code bên trong UserService

Thay vì tạo new 1 UserRepository ta sẽ sử dụng Constructor Injection 

```java
class UserService {

    private UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

```java
UserRepository repo = new MockUserRepository();

UserService service = new UserService(repo);
```

(MockUserRepository phải implements UserRepository thì mới truyền được vào constructor đang nhận kiểu UserRepository)

### 2. IOC - Inversion of Control - Đảo ngược quyền kiểm soát

#### a. Khái niệm

Là một nguyên lý trong thiết kế phần mềm, trong đó quyền kiểm soát việc tạo và quản lý đối tượng, dependency được chuyển từ chính class sang một thành phần bên ngoài.

#### b. Mối quan hệ giữa IOC và DI

DI là một kỹ thuật phổ biến để thực hiện IoC.