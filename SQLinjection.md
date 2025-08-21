# Tìm hiểu về SQL injection
- Người thực hiện: Lê Thị Mỹ Tiên
- Cập nhật lần cuối: 21/8/2025
## Mục Lục
## Nội dung
### 1. Khái niệm - Tác động
- SQL Injection (SQLi) là một lỗ hổng bảo mật web cho phép kẻ tấn công chèn hoặc can thiệp vào các truy vấn SQL mà ứng dụng gửi đến cơ sở dữ liệu.

- Tác động: 
  + Rò rỉ dữ liệu nhạy cảm (dữ liệu người dùng khác, thông tin hệ thống, v.v.).
  + Sửa đổi hoặc xóa dữ liệu, gây thay đổi nội dung và hành vi ứng dụng.
  + Trong trường hợp nghiêm trọng, có thể chiếm quyền kiểm soát máy chủ hoặc hạ tầng backend.
  + Có thể dẫn đến tấn công từ chối dịch vụ (DoS).
### 2. Cách phát hiện lỗ hổng SQL injection
Có thể phát hiện SQLi theo cách thủ công bằng cách:
- Ký tự dấu nháy đơn 'và tìm kiếm lỗi hoặc các bất thường khác.

- Một số cú pháp SQL cụ thể đánh giá theo giá trị cơ sở (gốc) của điểm vào và theo một giá trị khác, đồng thời tìm kiếm sự khác biệt có hệ thống trong phản hồi của ứng dụng.

- Các điều kiện Boolean như OR 1=1 và OR 1=2, và tìm kiếm sự khác biệt trong phản hồi của ứng dụng.

- Tải trọng được thiết kế để kích hoạt độ trễ thời gian khi thực hiện trong truy vấn SQL và tìm kiếm sự khác biệt về thời gian phản hồi.

- Tải trọng OAST được thiết kế để kích hoạt tương tác mạng ngoài băng tần khi được thực hiện trong truy vấn SQL và giám sát mọi tương tác phát sinh.
#### 2.1 Trong các phần khác nhau của truy vấn
- Hầu hết các lỗ hổng SQL injection đều xảy ra trong WHERE của một SELECT
- Trong UPDATE, trong các giá trị được cập nhật hoặc WHERE.
- Trong INSERT, bên trong các giá trị được chèn vào.
- Trong SELECT, bên trong tên bảng hoặc cột.
- Trong SELECT, trong ORDER BY.
##### 2.1.1 Truy vấn dữ liệu ẩn
📜**LAB 1: SQLI TRONG MỆNH ĐỀ WHERE CHO PHÉP TRUY XUẤT DỮ LIỆU ẨN**

Bài thực hành này chứa lỗ hổng SQL injection trong bộ lọc danh mục sản phẩm. Khi người dùng chọn một danh mục, ứng dụng sẽ thực hiện một truy vấn SQL như sau:
`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

Thực hiện một cuộc tấn công tiêm SQL khiến ứng dụng hiển thị một hoặc nhiều sản phẩm chưa phát hành.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/23e7245a-aec2-4c44-9eb0-99480937f093" />

##### 2.1.2 Phá vỡ logic ứng dụng
Nếu ứng dụng kiểm tra thông tin đăng nhập bằng câu truy vấn sau:<br>
`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'`
Nếu truy vấn trả về thông tin chi tiết của người dùng, thì đăng nhập thành công. Nếu không, đăng nhập sẽ bị từ chối.

Trong trường hợp này, kẻ tấn công có thể đăng nhập bằng bất kỳ người dùng nào mà không cần mật khẩu. Chúng có thể thực hiện việc này bằng cách sử dụng chuỗi chú thích SQL -- để xóa kiểm tra mật khẩu khỏi mệnh đề truy vấn WHERE. 

Ví dụ:<br> `SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`
📜 **LAB 2: SQLI CHO PHÉP BỎ QUA ĐĂNG NHẬP**
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6976fdfb-7c4f-4c36-a8b0-b19acab55f3f" />

