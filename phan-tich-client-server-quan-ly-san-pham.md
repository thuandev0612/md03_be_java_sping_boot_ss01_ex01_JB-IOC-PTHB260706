# Phân tích mô hình Client – Server cho ứng dụng Quản lý sản phẩm

**Chức năng:** Xem danh sách sản phẩm.

**Phương án lựa chọn:** Server tạo trang HTML chứa danh sách sản phẩm. Client nhận trang HTML và hiển thị cho người dùng. Đây là tài liệu phân tích; chưa cần lập trình ứng dụng.

## 1. Client gửi yêu cầu

Client là trình duyệt web, chẳng hạn Chrome hoặc Edge. Khi người dùng mở trang danh sách sản phẩm, trình duyệt gửi một HTTP Request đến server.

- URL dự kiến: `http://localhost:8080/products`.
- HTTP Method: `GET`.
- Mục đích: yêu cầu lấy danh sách sản phẩm để xem.
- Không cần gửi nội dung sản phẩm trong phần thân yêu cầu này.

Trong URL dự kiến:

| Thành phần | Ý nghĩa |
| --- | --- |
| `http://` | Giao thức trao đổi giữa trình duyệt và server |
| `localhost` | Máy đang chạy trình duyệt; bài giả định server cũng chạy trên máy này |
| `8080` | Cổng được lựa chọn cho ứng dụng trong ví dụ |
| `/products` | Đường dẫn được thiết kế để xem danh sách sản phẩm |

Chọn GET vì thao tác này chỉ yêu cầu đọc dữ liệu. Đây là mục đích của phương thức GET theo [tài liệu HTTP của MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/GET).

Minh họa yêu cầu bằng HTTP/1.1:

```http
GET /products HTTP/1.1
Host: localhost:8080
Accept: text/html
```

URL trên là địa chỉ đề xuất khi xây dựng ứng dụng, chưa phải một website đã có sẵn.

## 2. Server xử lý yêu cầu

Server chạy ứng dụng Spring Boot. Với phương án Spring MVC trả HTML, các bước xử lý dự kiến là:

1. Nhận yêu cầu `GET /products`.
2. Chuyển yêu cầu đến phương thức xử lý tương ứng trong `ProductController`.
3. Đọc danh sách sản phẩm từ nơi lưu dữ liệu giả lập trong RAM.
4. Đưa danh sách vào dữ liệu dành cho giao diện, gọi là `Model`.
5. Dùng mẫu trang, ví dụ `products.html` với Thymeleaf, để tạo HTML chứa dữ liệu sản phẩm.
6. Gửi phản hồi HTTP về trình duyệt.

Nếu danh sách trống, trang có thể hiển thị “Chưa có sản phẩm”. Chức năng xem danh sách không thêm, sửa hoặc xóa sản phẩm.

Controller tiếp nhận yêu cầu và view tạo HTML là cách tổ chức được trình bày trong [hướng dẫn Spring MVC chính thức](https://spring.io/guides/gs/serving-web-content/).

## 3. Vai trò của Database và dữ liệu giả lập trong RAM

Database có vai trò lưu trữ dữ liệu sản phẩm để server tra cứu. Trong bài này, chưa dùng hệ quản trị cơ sở dữ liệu như MySQL; thay vào đó, dùng một danh sách Java như `List<Product>` trong RAM của ứng dụng server.

Dữ liệu minh họa:

| Mã sản phẩm | Tên sản phẩm | Giá bán (VNĐ) |
| --- | --- | ---: |
| SP01 | Bàn phím | 500.000 |
| SP02 | Chuột không dây | 250.000 |

Server đọc danh sách này khi cần hiển thị sản phẩm. Client không truy cập trực tiếp danh sách Java trong RAM.

RAM là bộ nhớ tạm. Nếu chỉ lưu ở đây, những thay đổi dữ liệu sẽ mất khi ứng dụng dừng; dữ liệu mẫu chỉ xuất hiện lại nếu chương trình khởi tạo lại khi chạy. Trong mô hình này, danh sách nằm ngay trong ứng dụng server, không phải một máy chủ database riêng.

## 4. Server trả Response

Trong phương án đã chọn, phản hồi thành công gồm:

- Mã trạng thái HTTP: `200 OK`.
- Kiểu nội dung: `text/html; charset=UTF-8`.
- Nội dung: trang HTML chứa danh sách mã sản phẩm, tên sản phẩm và giá bán.

Trình duyệt đọc HTML rồi hiển thị danh sách thành giao diện cho người dùng. Trình duyệt không nhận trực tiếp đối tượng Java `Product`.

HTML và JSON là hai lựa chọn thiết kế khác nhau. Nếu xây dựng API, server có thể trả JSON và phần giao diện dùng JavaScript để tạo bảng từ dữ liệu đó. Bài làm này chọn HTML để trình duyệt hiển thị trang do server tạo.

GET/POST nói về phương thức yêu cầu; HTML/JSON nói về định dạng dữ liệu. Một yêu cầu GET có thể nhận HTML hoặc JSON, tùy cách thiết kế server.

## 5. Sơ đồ Request – Response

```text
Client (trình duyệt)
    |
    | 1. Request: GET /products
    v
Server (Spring Boot)
    |
    | 2. Đọc danh sách sản phẩm
    v
Dữ liệu giả lập trong RAM của Server
    |
    | 3. Cung cấp danh sách sản phẩm
    v
Server tạo trang HTML chứa danh sách
    |
    | 4. Response: 200 OK + HTML
    v
Client nhận HTML và hiển thị danh sách
```

Các mũi tên giữa server và RAM mô tả việc đọc dữ liệu trong ứng dụng. Request và Response HTTP được trao đổi giữa client và server.
