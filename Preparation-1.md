# Buổi 1: Nhập môn cơ sở dữ liệu

## I. Tổng quan buổi học
### 1. Giới thiệu, nói chuyện về lộ trình chi tiết, định hướng của team.
### 2. Thảo luận về các kiến thức mọi người đã chuẩn bị 
- Cơ sở dữ liệu là gì?
- Hệ quản trị CSDL là gì?
- Cài đặt MS SQL Server
- Câu lệnh tạo database, table trong MS SQL Server

## II. Cơ sở dữ liệu (Database)
### 1. Khái niệm, mục đích
- Một CSDL là một bộ tập hợp các dữ liệu liên quan tới nhau
- Khái niệm “dữ liệu” trong một cơ sở dữ liệu có thể bao phủ một phạm vi rất rộng các đối tượng khác nhau từ các số, văn bản, đồ họa, video, v.v...
- Một cơ sở dữ liệu được thiết kế, xây dựng, lớn dần và được sử dụng cho một mục đích có thể nào đó. Nó sẽ có một tập các người sử dụng tiềm năng và được sử dụng cho các ứng dụng cụ thể ngay từ khi thiết kế ban bầu.
### 2. VD
![alt text](image.png)
- VD: 1 CSDL quản lí thông tin các cuốn sách có mục đích là dùng để tìm các cuốn sách theo tên, tác giả, NXB, ...
- Người sử dụng tiềm năng sẽ là chủ tiệm sách, chủ thư viện, ...

## III. Hệ quản trị CSDL (Database Management System)
### 1. Khái niệm
- Là một hệ thống phần mềm cho phép tạo lập CSDL và điều khiển mọi truy nhập đến CSDL đó.
> Một CSDL được quản lý bởi một hệ quản trị CSDL thường được gọi là một hệ cơ sở dữ liệu.
### 2. Đặc tính
- Cho phép người dùng tạo mới CSDL, thông qua ngôn ngữ định nghĩa dữ liệu (DDLs – Data Definition Languages).
- Cho phép người dùng truy vấn cơ sở dữ liệu, thông qua ngôn ngữ thao tác dữ liệu (DMLs – Data Manipulation Languages).
- Hỗ trợ lưu trữ số lượng lớn dữ liệu, thường lên tới hàng Gigabytes hoặc nhiều hơn, trong một thời gian dài. Duy trì tính bảo mật và tính toàn vẹn trong quá trình xử lý.
- Kiểm soát truy nhập dữ liệu từ nhiều người dùng tại cùng một thời điểm.
![alt text](image-1.png)
![alt text](image-2.png)

## IV. Tạo database + table
```sql
// 
CREATE DATABASE TENDATA;
GO

USE TENDATA;
GO 

CREATE TABLE SinhVien (
    TenCot kdl,
    MaSV INT,
    HoTen NVARCHAR(100)
);
GO

CREATE TABLE Data (
    TenCot KDL [Ràng buộc]
);
GO
```
- create database -> tạo CSDL
- use -> chuyển đến database đang sử dụng
- create table -> tạo bảng

![alt text](image-3.png)

- 1 số kiểu dữ liệu thông dụng trong SQL

- INT             → số nguyên
- BIGINT          → số nguyên rất lớn
- DECIMAL         → số thập phân chính xác
VD: Diem DECIMAL(4,2) -> Số thập phân tổng tối đa 4 chữ số, 2 chữ số sau dấu phẩy
- VARCHAR         → chuỗi
- NVARCHAR        → chuỗi Unicode / tiếng Việt
- DATE            → ngày
- DATETIME        → ngày + giờ
- TIME            → giờ
- BIT             → đúng/sai (0/1)