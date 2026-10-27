# BUỔI 8: TỐI ƯU JPA, VALIDATION VÀ EXCEPTION HANDLING

## Nội dung
- [BUỔI 8: TỐI ƯU JPA, VALIDATION VÀ EXCEPTION HANDLING](#buổi-8-tối-ưu-jpa-validation-và-exception-handling)
  - [Nội dung](#nội-dung)
  - [I. Các phương pháp tối ưu JPA](#i-các-phương-pháp-tối-ưu-jpa)
    - [1. Lazy / Eager initialization](#1-lazy--eager-initialization)
      - [a. Lazy initialization: Tải tạm hoãn](#a-lazy-initialization-tải-tạm-hoãn)
      - [b. Eager initialization: Tải nhanh](#b-eager-initialization-tải-nhanh)
      - [c. Khi nào sử dụng lazy / eager](#c-khi-nào-sử-dụng-lazy--eager)
      - [d. Sự khác biệt giữa Lazy loading và Eager loading](#d-sự-khác-biệt-giữa-lazy-loading-và-eager-loading)
    - [2. Pagination](#2-pagination)
    - [3. Caching](#3-caching)
      - [a. Ưu điểm / Nhược điểm](#a-ưu-điểm--nhược-điểm)
      - [b. Cách sử dụng](#b-cách-sử-dụng)
    - [4. Manual / Native query](#4-manual--native-query)
      - [Native query là gì ?](#native-query-là-gì-)
      - [Khi nào nên dùng Native query:](#khi-nào-nên-dùng-native-query)
      - [Hạn chế:](#hạn-chế)
    - [5. Dynamic query: Truy vấn động](#5-dynamic-query-truy-vấn-động)
      - [a. Query by Example (QBE)](#a-query-by-example-qbe)
      - [b. Query by Specification](#b-query-by-specification)
      - [c. Querydsl](#c-querydsl)
  - [II. Phòng tránh N + 1 query](#ii-phòng-tránh-n--1-query)
    - [1. N + 1 query là gì?](#1-n--1-query-là-gì)
    - [2. Cách nhận diện lỗi](#2-cách-nhận-diện-lỗi)
    - [3. Cách giải quyết](#3-cách-giải-quyết)
  - [III. Tìm hiểu về Validation](#iii-tìm-hiểu-về-validation)
    - [1. Khái niệm](#1-khái-niệm)
    - [2. Quy trình](#2-quy-trình)
  - [IV. Tìm hiểu về Exception handling](#iv-tìm-hiểu-về-exception-handling)
    - [1. Khái niệm](#1-khái-niệm-1)
    - [2. Làm nhiệm vụ gì](#2-làm-nhiệm-vụ-gì)
  - [V. Override default fetch plan với join fetch / entity graph](#v-override-default-fetch-plan-với-join-fetch--entity-graph)
    - [1. Cách 1: Join fetch](#1-cách-1-join-fetch)
    - [2. Cách 2: Entity graph](#2-cách-2-entity-graph)

---------------------------------


## I. Các phương pháp tối ưu JPA
### 1. Lazy / Eager initialization
#### a. Lazy initialization: Tải tạm hoãn

- Khái niệm: Nghĩa là các thực thể liên quan chỉ được nạp khi chúng được truy cập lần đầu tiên

> Là mặc định cho các mối quan hệ @OneToMany và @ManyToMany.
> Việc tìm nạp xảy ra theo yêu cầu, giảm bộ nhớ ban đầu và tải cơ sở dữ liệu.

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

    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderDetail> orderDetails;
}
```

>fetch = FetchType.LAZY yêu cầu Hibernate không tải danh sách chi tiết order (orderDetail) cho đến khi getOrderDetail được gọi

#### b. Eager initialization: Tải nhanh 

- Khái niệm: Tải Nhanh là một chiến lược trong đó tất cả dữ liệu cần thiết, bao gồm cả dữ liệu liên quan, được tải ngay từ đầu trong một yêu cầu duy nhất, đảm bảo truy cập tức thì vào tất cả thông tin cần thiết.

> Là mặc định cho các mối quan hệ ManyToOne, OneToOne
> Hibernate thực hiện các truy vấn join hoặc các truy vấn bổ sung để tìm nạp dữ liệu liên kết

```java
@Entity
@Data
@Table(name = "order_detail")
public class OrderDetail {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "order_id")
    private Order order;
}
```
> fetch = FetchType.Eager đảm bảo rằng order được tải ngay lập tức cùng với đối tượng orderDeatil

#### c. Khi nào sử dụng lazy / eager
- Sử dụng lazy initialization khi:

***Bảng tin Mạng xã hội:*** Tải bảng tin của người dùng và chỉ tìm nạp hình ảnh hoặc video khi họ cuộn hoặc nhấp vào bài đăng.

***Trang web nặng nội dung:*** Các trang tin tức hoặc cửa hàng thương mại điện tử nơi các chi tiết sản phẩm, đánh giá hoặc hình ảnh bổ sung được tìm nạp khi người dùng tương tác với nội dung.


- Sử dụng eager initialization khi:

***Bảng điều khiển nhiều dữ liệu (Data-Dense Dashboards):*** Các ứng dụng như hệ thống CRM hoặc bảng điều khiển phân tích cần tất cả dữ liệu có sẵn ngay từ đầu để hiển thị nhiều thực thể liên quan.

***Hệ thống Báo cáo:*** Các báo cáo tài chính hoặc kinh doanh kéo tất cả dữ liệu liên quan để hiển thị ngay lập tức, đảm bảo không cần truy vấn bổ sung nào sau đó.

#### d. Sự khác biệt giữa Lazy loading và Eager loading 

| Tiêu chí so sánh                | Tải Tạm Hoãn (Lazy Loading)                    | Tải Nhanh (Eager Loading)                                                |
| :------------------------------ | :--------------------------------------------- | :----------------------------------------------------------------------- |
| **Cách thức hoạt động**         | Dữ liệu chỉ được tải khi được truy cập.        | Dữ liệu được tải ngay từ đầu với yêu cầu ban đầu.                        |
| **Tốc độ tải ban đầu**          | Nhanh hơn.                                     | Chậm hơn.                                                                |
| **Mức độ sử dụng bộ nhớ**       | Chỉ sử dụng bộ nhớ cho dữ liệu được truy cập.  | Sử dụng bộ nhớ cao hơn do tìm nạp tất cả dữ liệu.                        |
| **Số lượng truy vấn SQL**       | Có thể dẫn đến nhiều truy vấn.                 | Thông thường ít truy vấn hơn.                                            |
| **Độ phức tạp của mã nguồn**    | Yêu cầu mã phức tạp hơn để tải trì hoãn.       | Mã đơn giản hơn vì tất cả dữ liệu có sẵn ngay từ đầu.                    |
| **Mức tiêu thụ băng thông**     | Sử dụng ít băng thông hơn ban đầu.             | Có thể sử dụng nhiều băng thông hơn nếu tìm nạp dữ liệu không cần thiết. |
| **Trường hợp ứng dụng phù hợp** | Phù hợp với việc tìm nạp dữ liệu có điều kiện. | Phù hợp nơi cần tất cả dữ liệu ngay từ đầu.                              |


### 2. Pagination

- Phân trang: là kỹ thuật chia một tập dữ liệu lớn thành các "trang" nhỏ hơn, dễ quản lý hơn, chỉ lấy từng trang một khi cần.

Ví dụ: Thay vì tải 1.000 hồ sơ khách hàng, bạn có thể tải 10 hồ sơ cho mỗi trang.

Lợi ích: Giảm tải bộ nhớ, tăng tốc độ phản hồi API, và cải thiện trải nghiệm người dùng bằng cách hiển thị dữ liệu theo từng phần.

- Sắp xếp (Sorting):

Sắp xếp cho phép bạn sắp xếp các bản ghi theo thứ tự tăng dần (ascending) hoặc giảm dần (descending) dựa trên một hoặc nhiều trường dữ liệu.

Ví dụ: Sắp xếp danh sách sản phẩm theo tên (A-Z) hoặc theo giá (thấp đến cao).
Lợi ích: Cải thiện khả năng trình bày dữ liệu và giúp người dùng tìm kiếm thông tin hiệu quả hơn.

```java
package com.gfg.springsortingpaginationdemo;

import org.springframework.data.domain.*;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping
    public Page<Product> getAllProducts(
            @RequestParam(defaultValue = "0") int page, // Trang mặc định là 0
            @RequestParam(defaultValue = "5") int size, // Kích thước trang mặc định là 5
            @RequestParam(defaultValue = "id") String sortBy, // Sắp xếp mặc định theo trường 'id'
            @RequestParam(defaultValue = "true") boolean ascending) { // Mặc định là tăng dần

        // 1. Tạo đối tượng Sort
        Sort sort = ascending
                    ? Sort.by(sortBy).ascending()
                    : Sort.by(sortBy).descending();

        // 2. Tạo đối tượng Pageable
        // PageRequest.of(...): Là phương thức tĩnh (static method) của Spring Data dùng để khởi tạo một đối tượng yêu cầu phân trang.
        Pageable pageable = PageRequest.of(page, size, sort); 

        // 3. Gọi Service và trả về kết quả
        return productService.findAll(pageable);
    }
}
```

**@RequestParam là một annotation của Spring MVC dùng để lấy các giá trị được truyền lên từ URL (Query Parameters) hoặc từ form gửi lên.**

Trường hợp 1 (Có truyền đủ tham số):

URL: http://localhost:8080/products?page=2&size=10

Kết quả: page = 2, size = 10.

Trường hợp 2 (Không truyền tham số nào - dùng mặc định):

URL: http://localhost:8080/products

Kết quả: Do không thấy page và size, Spring tự áp dụng giá trị mặc định nên page = 0, size = 5.

**@param pageable đối tượng Pageable chứa số trang, kích thước trang và sắp xếp.**

Đối tượng pageable này giống như một "túi hành lý chứa chỉ dẫn" bạn đưa cho tầng Service/Repository. Khi nhìn vào túi này, Spring Data JPA sẽ biết chính xác:

Lấy dữ liệu từ trang mấy?

Mỗi trang lấy bao nhiêu dòng?

Sắp xếp theo cột nào, tăng hay giảm?

### 3. Caching

Caching là một kỹ thuật trong lập trình máy tính được sử dụng để lưu trữ tạm thời các dữ liệu, kết quả của các phép toán, hoặc các tài nguyên có thể được truy cập một cách nhanh chóng. Mục tiêu của caching là giảm thời gian truy cập dữ liệu, tăng cường hiệu suất và giảm gánh nặng cho hệ thống.

Khi một ứng dụng yêu cầu dữ liệu, trước hết nó sẽ kiểm tra xem dữ liệu đã được lưu trong bộ nhớ cache hay chưa.

Nếu dữ liệu đã tồn tại trong cache (Cache Hit), ứng dụng sẽ lấy nhanh chóng từ đó thay vì phải truy cập vào nơi lữu trữ gốc (ví dụ: cơ sở dữ liệu, API, hoặc tệp tin).

Nếu dữ liệu không có trong cache (Cache Miss), ứng dụng sẽ lấy từ nơi lưu trữ gốc và sau đó lưu vào cache để sử dụng cho các lần truy cập sau.

Caching thường được sử dụng để giảm độ trễ, tăng tốc độ xử lý, và giảm tải cho nguồn lực hệ thống. Các kỹ thuật caching có thể áp dụng cho nhiều loại dữ liệu và tài nguyên, bao gồm dữ liệu cơ sở dữ liệu, kết quả của phương thức tính toán, tệp tin tĩnh, và nhiều hơn nữa.

#### a. Ưu điểm / Nhược điểm

| Ưu điểm                                                                                                      | Nhược điểm                                                                                                                   |
| :----------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------- |
| **Tăng tốc độ phản hồi:** Đọc dữ liệu từ RAM siêu nhanh, giảm thiểu tối đa độ trễ cho API.                   | **Dữ liệu lỗi thời (Stale Data):** Dễ trả về thông tin cũ nếu dữ liệu trong Database thay đổi mà quên xóa cache.             |
| **Giảm tải cho Database:** Tránh việc Database phải thực thi lặp đi lặp lại các câu query giống nhau.        | **Tốn bộ nhớ RAM:** Lưu trữ dữ liệu trực tiếp trên RAM, dễ dẫn đến nguy cơ tràn bộ nhớ (OOM) nếu lạm dụng.                   |
| **Cải thiện trải nghiệm người dùng:** Ứng dụng mượt mà, phản hồi tức thì ngay cả khi có lượng truy cập lớn.  | **Phức tạp khi quản lý:** Khó khăn trong việc quyết định thời điểm và cách thức xóa hoặc làm mới cache (Cache Invalidation). |
| **Tiết kiệm tài nguyên hệ thống:** Giảm bớt số lượng kết nối đồng thời và áp lực tính toán cho tầng lưu trữ. | **Khó khăn khi Debug:** Dễ gây nhầm tưởng là lỗi code trong khi thực tế ứng dụng chỉ đang đọc dữ liệu cũ từ cache.           |

#### b. Cách sử dụng

***Bước 1: Bật tính năng Cache cho ứng dụng***

```java
@SpringBootApplication
@EnableCaching // Bật tính năng cache lên ở main class
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

***Bước 2: Sử dụng @Cacheable để lưu dữ liệu vào Cache***

Annotation @Cacheable được sử dụng tại các phương thức có kết quả cần được lưu vào cache.

Ví dụ: Lấy thông tin sản phẩm theo ID trong hệ thống quản lý đồ ăn, thức uống.

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    @Cacheable(value = "products", key = "#id")
    public Product getProductById(Integer id) {
        return productRepository.findById(id)
                .orElseThrow(() ->
                    new RuntimeException("Không tìm thấy sản phẩm"));
    }
}
```

Giải thích:

@Cacheable: kiểm tra cache trước khi thực thi phương thức.

value = "products": tên vùng cache được sử dụng để lưu dữ liệu.

key = "#id": sử dụng giá trị tham số id làm khóa cache.

Nếu dữ liệu đã có trong cache, Spring trả về dữ liệu đó mà không cần thực thi lại phương thức.

Nếu dữ liệu chưa có trong cache, phương thức sẽ truy vấn cơ sở dữ liệu, sau đó kết quả được lưu vào cache để sử dụng cho những lần gọi tiếp theo.

Ví dụ hoạt động:

Giả sử người dùng gọi getProductById(1) ba lần liên tiếp và dữ liệu chưa bị xóa khỏi cache.

Lần gọi	Hoạt động
Lần 1: Cache Miss → truy vấn Database → lưu kết quả vào Cache

Lần 2:	Cache Hit → lấy kết quả từ Cache

Lần 3:	Cache Hit → lấy kết quả từ Cache

Nhờ đó, ứng dụng giảm số lần truy vấn Database khi cùng một dữ liệu được yêu cầu nhiều lần.

***Bước 3: Sử dụng @CachePut để cập nhật Cache***

Khi thông tin sản phẩm thay đổi, dữ liệu trong cache cũng cần được cập nhật để tránh trả về thông tin cũ.

Annotation @CachePut luôn thực thi phương thức và cập nhật kết quả trả về vào cache.

```java
@CachePut(value = "products", key = "#product.id")
public Product updateProduct(Product product) {
    return productRepository.save(product);
}
```

Giải thích:

Phương thức updateProduct() vẫn được thực thi mỗi khi được gọi.

Sản phẩm được cập nhật trong Database.

Kết quả trả về được ghi vào cache với khóa tương ứng với ID sản phẩm.

Khác với @Cacheable, @CachePut không bỏ qua việc thực thi phương thức khi cache đã có dữ liệu.

***Bước 4: Sử dụng @CacheEvict để xóa Cache***

Khi một sản phẩm bị xóa khỏi Database, dữ liệu tương ứng trong cache cũng cần được xóa để tránh trả về thông tin không còn tồn tại.

Annotation @CacheEvict được sử dụng để loại bỏ dữ liệu khỏi cache.

```java
@CacheEvict(value = "products", key = "#id")
public void deleteProduct(Integer id) {
    productRepository.deleteById(id);
}
```

Giải thích:

value = "products": xác định vùng cache cần thao tác.

key = "#id": xác định khóa của dữ liệu cần xóa.

Khi phương thức thực thi thành công, dữ liệu trong cache có khóa tương ứng sẽ bị loại bỏ theo cơ chế mặc định.

Nếu muốn xóa toàn bộ dữ liệu trong vùng cache products, có thể sử dụng:

```java
@CacheEvict(value = "products", allEntries = true)
public void clearProductCache() {
}
```

Phương thức trên xóa toàn bộ các mục trong cache products.

### 4. Manual / Native query

#### Native query là gì ?

Native Query (hay câu lệnh SQL thuần) là câu lệnh truy vấn viết bằng ngôn ngữ SQL chuẩn của cơ sở dữ liệu mà bạn đang sử dụng (như MySQL, PostgreSQL, Oracle, SQL Server...), thay vì sử dụng ngôn ngữ truy vấn hướng đối tượng của Spring Data (như JPQL hay HQL).

```java
public interface ProductRepository extends JpaRepository<Product, Integer> {

    //dùng JPQL
    @Query("select p from product p where p.category = ?1")
    Product myCustomQuery(String productCategory);

    //Native SQL.
    @Query(value = "select * from product p where p.category = ?1", nativeQuery = true)
    Product myCustomQuery2(String productCategory);
}
```

#### Khi nào nên dùng Native query:

Cần viết các câu lệnh SQL phức tạp, tối ưu hóa hiệu năng (tuning query) mà JPQL không hỗ trợ hoặc biên dịch không tốt.

Sử dụng các hàm đặc trưng của riêng hệ quản trị cơ sở dữ liệu đó (ví dụ: hàm xử lý ngày tháng riêng của MySQL hoặc PostgreSQL).

Truy vấn dữ liệu tổng hợp phức tạp (JOIN nhiều bảng phức tạp, subquery lằng nhằng).

#### Hạn chế:

***Không độc lập cơ sở dữ liệu (Thiếu tính Portable)***

Bản chất: Vì sử dụng câu lệnh SQL thuần, nó bị "trói buộc" vào một loại Database cụ thể (ví dụ: câu SQL viết cho MySQL có thể sẽ lỗi khi chạy trên Oracle hay SQL Server).

Khác biệt: Nếu dùng JPQL, Hibernate sẽ tự động dịch sang cú pháp phù hợp với Database bạn đang dùng. Còn với Native Query, nếu đổi Database, bạn phải tự sửa lại câu SQL cho tương thích.

***Bắt buộc phải tự viết countQuery khi phân trang (Pageable)***

Bản chất: Khi dùng tính năng phân trang, Spring Data JPA cần biết tổng số bản ghi để chia trang. Với JPQL, hệ thống sẽ tự động tạo câu lệnh đếm (COUNT). Nhưng với Native Query, nó không tự làm được điều này.

Hạn chế: Bạn buộc phải tự viết thêm câu lệnh đếm thủ công thông qua thuộc tính countQuery (hoặc tạo thêm một @NamedNativeQuery có hậu tố .count), nếu không hàm phân trang sẽ không hoạt động đúng.

***Không hỗ trợ sắp xếp động (Dynamic Sorting)***

Bản chất: Với JPQL, bạn có thể truyền tham số Sort vào hàm Repository để Spring tự động thêm mệnh đề ORDER BY linh hoạt dựa trên lựa chọn của người dùng.

Hạn chế: Native Query không hỗ trợ tính năng tự động sinh ORDER BY này. Nếu muốn sắp xếp kết quả, bạn phải viết cứng (hard-code) mệnh đề sắp xếp ngay trong câu SQL, khiến ứng dụng mất đi sự linh hoạt khi cần đổi cột sắp xếp lúc chạy.


### 5. Dynamic query: Truy vấn động
Khi phát triển ứng dụng với Spring Data, việc xây dựng các truy vấn động dựa trên nhiều tiêu chí lựa chọn là rất phổ biến để lấy dữ liệu từ cơ sở dữ liệu.

Hướng dẫn này sẽ khám phá ba phương pháp tiếp cận chính để tạo truy vấn động trong Spring Data JPA: Query by Example, Query by Specification, và Query by Querydsl.

#### a. Query by Example (QBE)

**Query by Example là phương pháp truy vấn dữ liệu dựa trên một đối tượng mẫu (Example). Lập trình viên tạo một đối tượng Entity chứa các giá trị cần tìm kiếm, sau đó Spring Data JPA tự động xây dựng truy vấn dựa trên những giá trị đó.**

Phương pháp này phù hợp với các truy vấn tìm kiếm đơn giản, không yêu cầu các điều kiện phức tạp như phép toán OR kết hợp nhiều nhóm điều kiện hoặc các biểu thức so sánh nâng cao.

Ví dụ: Tìm kiếm sản phẩm theo tên và danh mục trong hệ thống quản lý đồ ăn, thức uống.

Bước 1: Khai báo Repository

```java
public interface ProductRepository
        extends JpaRepository<Product, Integer>,
                QueryByExampleExecutor<Product> {
}
```

Bước 2: Xây dựng truy vấn trong Service

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<Product> search(Product product) {
        Example<Product> example = Example.of(product);
        return productRepository.findAll(example);
    }
}
```

Bước 3: Sử dụng truy vấn

```java
Product product = new Product();
product.setName("Popcorn");
product.setCategory("Snack");

List<Product> products = productService.search(product);
```

Trong ví dụ trên, Spring Data JPA sử dụng các giá trị có trong đối tượng product làm điều kiện tìm kiếm. Những thuộc tính không được thiết lập thường không được sử dụng làm điều kiện truy vấn.

Ưu điểm:

Dễ triển khai, không cần viết câu lệnh SQL hoặc JPQL.

Phù hợp với chức năng tìm kiếm theo nhiều trường dữ liệu đơn giản.

Có thể tùy chỉnh cách so khớp, chẳng hạn tìm tên chứa từ khóa hoặc không phân biệt chữ hoa, chữ thường.

Hạn chế:

Không phù hợp với các điều kiện truy vấn phức tạp như lớn hơn, nhỏ hơn hoặc các nhóm điều kiện logic phức tạp.

Không hỗ trợ trực tiếp các phép so sánh tùy ý trên giá trị thuộc tính.

#### b. Query by Specification

**Query by Specification cho phép xây dựng truy vấn động bằng cách tạo các điều kiện độc lập, sau đó kết hợp chúng thành một truy vấn hoàn chỉnh.**

Phương pháp này dựa trên Criteria API của JPA, phù hợp khi người dùng có thể lựa chọn nhiều tiêu chí tìm kiếm khác nhau.

Ví dụ, trong hệ thống quản lý sản phẩm, người dùng có thể tìm kiếm theo:

Tên sản phẩm.

Danh mục sản phẩm.

Giá tối thiểu.

Giá tối đa.

Bước 1: Khai báo Repository

```java
public interface ProductRepository
        extends JpaRepository<Product, Integer>,
                JpaSpecificationExecutor<Product> {
}
```

Bước 2: Xây dựng Specification

```java
public class ProductSpecification {

    public static Specification<Product> hasName(String name) {
        return (root, query, cb) ->
                name == null || name.isBlank()
                ? null
                : cb.like(
                    cb.lower(root.get("name")),
                    "%" + name.toLowerCase() + "%"
                );
    }

    public static Specification<Product> hasCategory(String category) {
        return (root, query, cb) ->
                category == null || category.isBlank()
                ? null
                : cb.equal(root.get("category"), category);
    }
}
```

Trong đó:

root: đại diện cho Entity được truy vấn.

query: đại diện cho truy vấn đang được xây dựng.

cb (CriteriaBuilder): cung cấp các phương thức tạo điều kiện truy vấn.

cb.like(): tạo điều kiện tìm kiếm theo mẫu.

cb.equal(): tạo điều kiện so sánh bằng.

Trả về null: không thêm điều kiện khi tiêu chí tương ứng không được cung cấp.

Bước 3: Kết hợp các điều kiện

```java
public List<Product> search(String name, String category) {
    Specification<Product> spec = Specification
            .where(ProductSpecification.hasName(name))
            .and(ProductSpecification.hasCategory(category));

    return productRepository.findAll(spec);
}
```

Khi người dùng chỉ nhập tên sản phẩm, truy vấn chỉ lọc theo tên. Khi nhập cả tên và danh mục, hai điều kiện được kết hợp bằng AND.

Ưu điểm:

Hỗ trợ các điều kiện so sánh như bằng, lớn hơn, nhỏ hơn, chứa từ khóa.

Có thể kết hợp điều kiện bằng AND, OR và xây dựng các bộ lọc phức tạp.

Có thể tái sử dụng từng điều kiện trong nhiều chức năng tìm kiếm khác nhau.

Hạn chế:

Cách viết dài và khó hiểu hơn Query by Example.

Cần hiểu các khái niệm trong Criteria API để xây dựng điều kiện phức tạp.

#### c. Querydsl

**Querydsl là thư viện hỗ trợ xây dựng truy vấn bằng Java với cú pháp có kiểu dữ liệu rõ ràng (type-safe). Thay vì ghép chuỗi SQL hoặc JPQL thủ công, lập trình viên có thể sử dụng các đối tượng được tạo từ Entity để xây dựng điều kiện truy vấn.**

Querydsl phù hợp với những hệ thống có yêu cầu tìm kiếm phức tạp, nhiều điều kiện và cần duy trì khả năng kiểm tra kiểu dữ liệu ngay trong quá trình biên dịch.

Ví dụ: Tìm sản phẩm theo tên, danh mục và khoảng giá.

Sau khi cấu hình Querydsl và tạo các lớp Q tương ứng với Entity, ta có thể viết truy vấn như sau:

```java
QProduct product = QProduct.product;

BooleanBuilder builder = new BooleanBuilder();

if (name != null && !name.isBlank()) {
    builder.and(product.name.containsIgnoreCase(name));
}

if (category != null && !category.isBlank()) {
    builder.and(product.category.eq(category));
}

if (minPrice != null) {
    builder.and(product.price.goe(minPrice));
}

if (maxPrice != null) {
    builder.and(product.price.loe(maxPrice));
}
```

Trong ví dụ trên:

QProduct: lớp được Querydsl tạo ra từ Entity Product.

BooleanBuilder: đối tượng hỗ trợ xây dựng các điều kiện truy vấn động.

containsIgnoreCase(): tìm chuỗi con không phân biệt chữ hoa, chữ thường.

eq(): so sánh bằng.

goe(): lớn hơn hoặc bằng.

loe(): nhỏ hơn hoặc bằng.

Các điều kiện được thêm vào BooleanBuilder khi người dùng cung cấp tiêu chí tương ứng. Sau đó, điều kiện này được truyền vào truy vấn Querydsl để lấy dữ liệu.

Ưu điểm:

Cú pháp truy vấn rõ ràng, hỗ trợ kiểm tra kiểu dữ liệu.

Phù hợp với các truy vấn nhiều điều kiện và có cấu trúc phức tạp.

Giảm lỗi do viết sai tên thuộc tính khi xây dựng truy vấn.

Hạn chế:

Cần cài đặt và cấu hình thư viện, bao gồm bước tạo các lớp Q.

Khó thiết lập hơn Query by Example và Specification.

Cần bổ sung cách tích hợp Querydsl với Spring Data JPA để thực thi truy vấn.


> Ba phương pháp truy vấn động đều giúp giảm việc phải viết nhiều câu truy vấn riêng biệt cho từng trường hợp tìm kiếm.

> Query by Example: lựa chọn phù hợp khi chỉ cần tìm kiếm theo các thuộc tính cơ bản của Entity.

> Query by Specification: phù hợp khi cần kết hợp nhiều điều kiện tìm kiếm có thể thay đổi theo dữ liệu đầu vào.

> Querydsl: phù hợp khi hệ thống có các truy vấn phức tạp và cần cú pháp truy vấn an toàn về kiểu dữ liệu.


## II. Phòng tránh N + 1 query
### 1. N + 1 query là gì?

Vấn đề N+1 (N+1 problem) là một vấn đề về hiệu năng trong Object Relational Mapping (ORM), nó kích hoạt nhiều câu lệnh Select Query (chính xác là N+1, với N = số lượng bản ghi trong bảng) vào database cho một select query duy nhất tại tầng application. Hibernate & Spring Data JPA cung cấp nhiều cách để nhận diện và ngăn chặn vấn đề này.

VD:

Hãy tưởng tượng bạn có 100 người dùng (Users), và mỗi người dùng có một danh sách các Quyền hạn (Roles).

Ý định của bạn: Lấy ra danh sách toàn bộ 100 người dùng kèm theo quyền của họ để in ra màn hình.

Điều gì xảy ra đằng sau (nếu code ngây thơ):

Truy vấn thứ nhất (1): Hibernate chạy 1 câu lệnh SQL để lấy danh sách toàn bộ 100 User (SELECT * FROM t_users).

N truy vấn tiếp theo (N): Vì cấu hình quan hệ là Lazy (chỉ khi nào gọi mới tải), nên khi vòng lặp chạy qua từng người trong 100 người đó để lấy Role, Hibernate lại bắn thêm 100 câu lệnh SQL riêng biệt đi lấy Role cho từng người (SELECT * FROM t_user_roles WHERE userid = ?).

> Tổng cộng: Thay vì chỉ cần 1 câu lệnh gộp chung, hệ thống phải gánh tới 1 + 100 = 101 câu lệnh truy vấn xuống database! Nếu hệ thống có 10.000 user, con số này sẽ là 10.001 câu truy vấn, làm sập hoặc chậm đơ ứng dụng. Đó chính là vấn đề N+1.

### 2. Cách nhận diện lỗi

```java
spring:
  jpa:
    show-sql: true // Hiển thị câu lệnh SQL
```

Khi bạn nhìn vào màn hình console, nếu thấy một câu lệnh SELECT lấy danh sách cha, ngay sau đó lặp lại hàng loạt các câu lệnh SELECT con giống hệt nhau với các ID khác nhau (ví dụ user_id = 1, user_id = 2, user_id = 3...), thì chắc chắn bạn đang dính lỗi N+1.

### 3. Cách giải quyết

***Dùng: join fetch / entity graph***

> Ở phần V
[V. Override default fetch plan với join fetch / entity graph](#v-override-default-fetch-plan-với-join-fetch--entity-graph)


## III. Tìm hiểu về Validation 

### 1. Khái niệm

Kiểm tra hợp lệ/ Xác thực dữ liệu. Là quá trình ***kiểm tra xem dữ liệu do người dùng hoặc hệ thống khác gửi vào có đáp ứng đúng các quy tắc, định dạng và điều kiện cho trước hay không*** trước khi cho phép xử lí tiếp hoặc lưu vào CSDL

### 2. Quy trình

***Bước 1: Đặt luật (Annotations) ở tầng DTOKhi nhận dữ liệu từ người dùng (thường thông qua JSON body gửi lên), bạn sẽ hứng nó vào một lớp DTO (Data Transfer Object). Tại đây, bạn dùng các annotation của thư viện Jakarta để đặt ra các "quy tắc".***

```java
Javapublic class UserRegisterRequest {
    
    @NotBlank(message = "Tên đăng nhập không được để trống")
    @Size(min = 4, max = 20, message = "Tên phải từ 4 đến 20 ký tự")
    private String username;

    @NotBlank(message = "Email không được để trống")
    @Email(message = "Email không đúng định dạng (ví dụ: abc@gmail.com)")
    private String email;

    @Min(value = 18, message = "Bạn phải từ 18 tuổi trở lên")
    private int age;
}
```

Ý nghĩa: Bạn đang tự định nghĩa các tiêu chuẩn: Tên không được trống và phải đủ độ dài, email phải đúng chuẩn, tuổi phải >= 18.

> Vì sao lại sử dụng tầng DTO:

Nếu như chúng ta đặt hết các tiêu chuẩn dữ liệu vào trong entity, bạn chỉ có duy nhất một class Entity. 

Khi đó, việc ép chung một luật cho cả lúc tạo mới lẫn lúc cập nhật sẽ trở thành một thảm họa logic 

Bằng cách dùng DTO, bạn có thể tạo ra nhiều DTO khác nhau (UserCreateDto, UserUpdateDto) với các bộ luật validation riêng biệt tùy ý.

***Bước 2: Kích hoạt "gác cổng" (@Valid) ở ControllerKhi người dùng gửi Request đến API, Spring Boot cần biết lúc nào thì phải chạy kiểm tra các luật ở Bước 1. Ta dùng annotation @Valid đặt ngay trước tham số trong Controller:***

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<String> registerUser(@Valid @RequestBody UserRegisterRequest request) {
        // Nếu dữ liệu hợp lệ, code mới chạy tiếp vào đây để lưu database
        return ResponseEntity.ok("Đăng ký thành công!");
    }
}
```

Điểm mấu chốt: Chỉ cần thêm chữ @Valid, Spring Boot sẽ tự động đóng vai trò là "bảo vệ". 

Nó tự quét các annotation trên DTO, nếu phát hiện dữ liệu vi phạm, nó sẽ chặn lại ngay lập tức và không cho code lọt vào trong hàm.


***Bước 3: Xử lý lỗi trả về cho đẹp (GlobalExceptionHandler)Khi bị chặn lại ở Bước 2, mặc định Spring Boot sẽ ném ra ngoại lệ và trả về một cục lỗi dài dòng, khó hiểu (mã HTTP 400).Để thân thiện hơn với Front-end, bạn viết một bộ "bắt lỗi toàn cục" để gom các lỗi đó lại thành một danh sách gọn gàng:***

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap();
        
        // Lấy ra từng trường bị lỗi kèm theo message bạn đã viết ở DTO
        ex.getBindingResult().getFieldErrors().forEach(error -> {
            String fieldName = error.getField(); // Tên trường (ví dụ: email)
            String errorMessage = error.getDefaultMessage(); // Câu thông báo lỗi
            errors.put(fieldName, errorMessage);
        });
        
        return ResponseEntity.badRequest().body(errors);
    }
}
```

Kết quả khi Client nhập sai: Front-end sẽ nhận được một đoạn JSON cực kỳ trực quan để hiển thị lỗi đỏ lè ngay dưới các ô nhập liệu:

```json
{
  "email": "Email không đúng định dạng (ví dụ: abc@gmail.com)",
  "age": "Bạn phải từ 18 tuổi trở lên"
}
```
>Tóm tắt quy trình:

DTO chứa quy tắc (@NotNull, @Email, @Size...).

Controller dùng @Valid để kích hoạt kiểm tra.

GlobalExceptionHandler bắt lỗi khi vi phạm và trả về JSON báo lỗi đẹp mắt cho người dùng sửa.


## IV. Tìm hiểu về Exception handling

### 1. Khái niệm
Là "trung tâm cứu hộ" của ứng dụng, chịu trách nhiệm bắt tất cả các lỗi xảy ra và biến chúng thành phản hồi gọn gàng, chuyên nghiệp gửi về cho Client.

### 2. Làm nhiệm vụ gì

Không có Exception Handling:
Khi có lỗi xảy ra (ví dụ: người dùng nhập sai dữ liệu, hoặc tìm một ID không tồn tại trong database), ứng dụng sẽ ném ra một đống "stack trace" màu đỏ dài dằng dặc kèm theo mã lỗi HTTP 500 Internal Server Error rất xấu xí và khó hiểu đối với Front-end.

Có Exception Handling (với @RestControllerAdvice):Mọi lỗi được thu gom lại một chỗ, tự động phân loại và trả về một cục JSON rõ ràng, thân thiện. 

Ví dụ:

Lỗi dữ liệu không hợp lệ -> Trả về mã 400 Bad Request kèm chỉ dẫn chi tiết trường nào bị sai.

Lỗi không tìm thấy bản ghi -> Trả về mã 404 Not Found ("Không tìm thấy sản phẩm này").

Lỗi hệ thống bất ngờ -> Trả về mã 500 Internal Server Error nhưng được che đậy lịch sự ("Lỗi hệ thống, vui lòng thử lại sau").

## V. Override default fetch plan với join fetch / entity graph

Khi bạn định nghĩa một Entity, bạn phải chỉ định chiến lược tải dữ liệu (fetch type) cho các mối quan hệ (như @OneToMany, @ManyToOne, @ManyToMany):

FetchType.LAZY (Mặc định cho các quan hệ nhiều - Collection): Khi gọi User, Hibernate không lấy Roles ngay lập tức để tiết kiệm bộ nhớ. Chỉ khi nào bạn gọi user.getRoles() nó mới truy vấn.

FetchType.EAGER (Mặc định cho quan hệ một - ToOne): Khi gọi User, Hibernate tự động JOIN và lấy luôn dữ liệu liên quan.

Vấn đề:

Nếu để LAZY, khi cần dùng dữ liệu liên quan mà quên mất, bạn sẽ gặp lỗi LazyInitializationException hoặc dính lỗi N+1.

Nếu để EAGER, Hibernate sẽ luôn luôn kéo dữ liệu liên quan về ngay cả khi bạn không cần dùng đến nó, gây lãng phí tài nguyên trầm trọng.

> Vì vậy, giữ nguyên mặc định là LAZY (để giữ cho các câu truy vấn thông thường nhẹ nhàng, nhanh chóng), nhưng tại những phương thức/API đặc biệt cần dùng dữ liệu liên quan, ta sẽ dùng JOIN FETCH hoặc @EntityGraph để ghi đè kế hoạch mặc định đó cho riêng câu lệnh đó.

### 1. Cách 1: Join fetch

Khi bạn viết một câu query tùy chỉnh, từ khóa FETCH sẽ ép Hibernate phải tải luôn dữ liệu liên quan ngay trong câu lệnh SQL đó, bất chấp cấu hình gốc trong Entity là LAZY.

```java
@Query("SELECT u FROM User u LEFT JOIN FETCH u.roles")
List<User> findAllWithRoles();
```

```sql
SELECT 
    u.id AS user_id, u.name AS user_name, 
    r.id AS role_id, r.name AS role_name
FROM t_users u
LEFT JOIN t_user_roles ur ON u.id = ur.user_id
LEFT JOIN t_roles r ON ur.role_id = r.id;
```
VD:

Bảng t_users:

| id   | name    |
| :--- | :------ |
| 1    | Alice   |
| 2    | Bob     |
| 3    | Charlie |

Bảng t_roles:

| id   | name  |
| :--- | :---- |
| 10   | ADMIN |
| 20   | USER  |

Bảng t_user_roles:

| user_id | role_id |
| :------ | :------ |
| 1       | 10      |
| 1       | 20      |
| 2       | 20      |

Kết quả sau khi fetch join:

| user_id | user_name | role_id | role_name |
| :------ | :-------- | :------ | :-------- |
| 1       | Alice     | 10      | ADMIN     |
| 1       | Alice     | 20      | USER      |
| 2       | Bob       | 20      | USER      |
| 3       | Charlie   | *Null*  | *Null*    |

### 2. Cách 2: Entity graph

Thay vì phải viết câu lệnh JPQL phức tạp, bạn dùng @EntityGraph để "ra lệnh" cho Spring Data JPA biết trường nào cần chuyển thành EAGER chỉ riêng cho hàm này:

```java
@EntityGraph(attributePaths = {"roles"})
List<User> findAll();
```

Ý nghĩa ghi đè: Phương thức findAll() mặc định của Spring Data JPA thường chỉ lấy bảng User. Nhưng khi gắn @EntityGraph(attributePaths = {"roles"}), bạn đang ghi đè hành vi mặc định đó thành: "Hãy lấy cả roles theo kiểu EAGER (join) cho lần gọi này nhé". Kết quả sẽ y hệt join fetch


> Hãy ưu tiên dùng @EntityGraph cho các trường hợp CRUD thông thường để code sạch sẽ. Chỉ chuyển sang dùng LEFT JOIN FETCH khi bạn bắt đầu cần viết câu query có điều kiện phức tạp hoặc join nhiều tầng bảng.

Cbi thêm
Concurrency & Distributed Systems Issues
Data Consistency & Performance Pitfalls