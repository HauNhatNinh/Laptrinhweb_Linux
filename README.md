# Laptrinhweb_Linux
## Cài đặt môi trường linux
### Cài đặt Hạ tầng (Infrastructure):
#### Môi trường (WSL/Ubuntu + Docker)
 - WSL / Ubuntu (Linux):
   - Mục đích: Tạo một môi trường máy chủ Linux chuẩn ngay trên máy Windows. Hầu hết các máy chủ web trên thế giới (như Nginx) đều chạy trên Linux. Làm việc trên Linux sẽ có một môi trường phát triển (dev) giống hệt với môi trường triển khai (production) thực tế.
<img width="1920" height="1080" alt="Screenshot (41)" src="https://github.com/user-attachments/assets/133ab467-88f1-4b27-b899-69bdad19c2fd" />

 - Docker Desktop:
   - Mục đích: Để chạy các "container". Thay vì cài đặt 6 phần mềm (Nginx, MariaDB, v.v.) trực tiếp lên máy, "đóng gói" chúng vào 6 "hộp" (container) riêng biệt.
   - Lợi ích: Sạch sẽ, dễ quản lý, dễ xóa, và đảm bảo 6 phần mềm này chạy giống hệt nhau trên mọi máy tính.
<img width="1920" height="1080" alt="Screenshot (42)" src="https://github.com/user-attachments/assets/71f6404b-6880-4bdc-9f78-2622824e3c34" />
<img width="1920" height="1080" alt="Screenshot (48)" src="https://github.com/user-attachments/assets/cd604f03-06ac-4438-8b24-1b8e028299c2" />

 - File docker-compose.yml:
   - Mục đích: Đây là một file "chỉ huy". Thay vì phải gõ 6 lệnh dài dòng để khởi động 6 container, ở đây nó sẽ định nghĩa tất cả chúng trong file này.
   - Lợi ích: Chỉ cần dùng một lệnh duy nhất (`docker-compose up`) để khởi động hoặc tắt toàn bộ 6 services. Nó cũng tự động tạo một mạng riêng (`iot_net`) để chúng "nói chuyện" với nhau.
<img width="1920" height="1080" alt="Screenshot (43)" src="https://github.com/user-attachments/assets/c0ccfc1f-bf3c-4a46-b59d-d5883e7c3987" />

<img width="1920" height="1080" alt="Screenshot (44)" src="https://github.com/user-attachments/assets/9fdbc01d-57c8-4947-b611-e964ebbcaf04" />

----
##### Các Services (6 Container)
###### 1. MariaDB (CSDL Quan hệ):
         - Mục đích: Để lưu trữ dữ liệu có cấu trúc (dạng bảng).
         - Trong bài này: Dùng để lưu 2 thứ:
         - Bảng users: Lưu tên đăng nhập, họ tên, và mật khẩu đã mã hóa (hash).
         - Bảng latest_data: Lưu giá trị mới nhất của cảm biến (để hiển thị nhanh lên dashboard).

###### 2. phpMyAdmin (Quản trị CSDL):
         - Mục đích: Cung cấp một giao diện web (giống như "trình duyệt") để nhìn và thao tác với dữ liệu trong MariaDB. Dùng nó để tạo bảng, thêm/sửa/xóa user (admin) mà không cần gõ lệnh SQL phức tạp.
<img width="1407" height="672" alt="image" src="https://github.com/user-attachments/assets/91f5098e-8b2b-42a9-a3a5-174918fc97cb" />
         
###### 3. Node-RED (Backend - "Bộ não"):
         - Mục đích: Đây là "backend" (bộ não) của toàn bộ ứng dụng.
         - Trong bài này: Nó làm 4 nhiệm vụ chính:
         - Tạo API: Tạo ra các đường dẫn (endpoints) như /login, /register, /latest-data.
         - Giả lập Cảm biến: Tự động tạo ra dữ liệu (nhiệt độ, độ ẩm) sau mỗi 5 giây.
         - Xử lý Logic: Nhận yêu cầu từ web (vd: login), kiểm tra CSDL (MariaDB), và trả về kết quả (JSON).
         - Phân phối Dữ liệu: Ghi dữ liệu mới nhất vào MariaDB (Flow 3) VÀ ghi dữ liệu lịch sử vào InfluxDB (Flow 1).
<img width="1322" height="941" alt="image" src="https://github.com/user-attachments/assets/e07d9c6a-db60-4a4a-8015-01e2b1ca4a30" />

### => Hiện tại vẫn đang lỗi chưa rõ lý do không thể ghi dữ liệu lịch sử vào InfluxDB.
---  
###### 4. InfluxDB (CSDL Chuỗi thời gian):
         - Mục đích: Đây là CSDL chuyên dụng để lưu lịch sử dữ liệu theo thời gian.
         - Khác biệt: MariaDB chỉ lưu giá trị mới nhất (vd: nhiệt độ = 25). InfluxDB lưu tất cả (8:00:00 nhiệt độ 25, 8:00:05 nhiệt độ 25.1, v.v.).
         - Trong bài này: Dùng để lưu toàn bộ lịch sử nhiệt độ, độ ẩm để cho Grafana vẽ biểu đồ.
<img width="1909" height="980" alt="image" src="https://github.com/user-attachments/assets/d973f870-0208-4da2-a699-62a46fc81b0c" />

### => Do Node-RED không ghi được vào InfluxDB nên chưa thể lấy lịch sử nhiệt độ, độ ẩm.
    - Tác động đến InfluxDB
        - Vai trò: Trong bài này, Node-RED là "Nhà sản xuất" dữ liệu lịch sử (cứ 5 giây nó tạo ra nhiệt độ/độ ẩm). InfluxDB là "Kho chứa" dữ liệu đó.
        - Hậu quả: Khi Node-RED không ghi vào được, "kho chứa" InfluxDB sẽ hoàn toàn trống rỗng (hoặc chỉ chứa dữ liệu rác từ các bài test curl cũ). Mặc dù bản thân dịch vụ InfluxDB vẫn chạy, nhưng cái "xô" (iotbucket) không nhận được bất kỳ dữ liệu nào.
---
###### 5. Grafana (Hiển thị Biểu đồ):
         - Mục đích: Công cụ chuyên dụng để vẽ biểu đồ đẹp và phức tạp.
         - Trong bài này: Nó kết nối trực tiếp với InfluxDB (nguồn dữ liệu lịch sử), tạo ra biểu đồ, và cung cấp một link <iframe> để  nhúng vào index.html.
<img width="1912" height="977" alt="image" src="https://github.com/user-attachments/assets/66418a7c-adbe-44eb-a42f-bd92d2c7c2f1" />

### => Vì Node-RED không ghi được vào InfluxDB nên chưa thể lấy lịch sử nhiệt độ, độ ẩm nên dẫn đến Grafana cũng không có dữ liệu để sinh biểu đồ
    - Tác động đến Grafana
        -Vai trò: Grafana là "Người tiêu thụ" dữ liệu. Nó kết nối với "kho chứa" InfluxDB để lấy dữ liệu và vẽ biểu đồ.
        - Hậu quả: Khi Grafana thực hiện câu truy vấn (query) mà bạn đã viết (from(bucket: "iotbucket")...), nó hỏi InfluxDB: "Cho tôi xin dữ liệu temperature và humidity trong 5 phút qua".
        - InfluxDB (vì kho đang trống) sẽ trả lời: "Không có gì cả."
        - Kết quả là Grafana hiển thị thông báo "No data" (Không có dữ liệu) — chính là lỗi bạn đã thấy trong ảnh
---
###### 6. Nginx (Web Server & Reverse Proxy):
         - Mục đích: Đây là "bộ mặt" và là "cổng gác" của hệ thống.
         - Trong bài này: Nó làm 2 nhiệm vụ:
         - Web Server: Phục vụ file index.html (trang SPA của bạn) khi người dùng truy cập http://haunhatninh.com.
         - Reverse Proxy (Cổng gác): Nhận tất cả truy cập qua cổng 80. Khi thấy /nodered, nó bí mật chuyển hướng (proxy) đến nodered:1880. Khi thấy /grafana, nó chuyển đến grafana:3000.
 - Cấu hình nginx để chạy website qua url http://haunhatninh.com  
<img width="1881" height="791" alt="image" src="https://github.com/user-attachments/assets/c06b6443-9db4-4124-af1b-c77d44fe914b" />

 - Cấu hình nginx để http://haunhatninh.com/nodered truy cập vào nodered qua cổng 80, (dù nodered đang chạy ở port 1880)
<img width="1615" height="807" alt="image" src="https://github.com/user-attachments/assets/bfa401f5-a4dc-4875-8692-dbb4a41d3f39" />
   
 - Cấu hình nginx để http://fullname.com/grafana truy cập vào grafana qua cổng 80, (dù grafana đang chạy ở port 3000)
<img width="1893" height="716" alt="image" src="https://github.com/user-attachments/assets/490abe65-5861-41a8-8a01-3bdfcdf086dd" />
   
### 🖥️ Ứng dụng Web (Frontend)
    - index.html (SPA) và JavaScript:
         - Mục đích: Đây là "frontend" (bộ mặt) mà người dùng nhìn thấy và tương tác.
         - Trong bài này: Nó là một "Ứng dụng Trang đơn" (SPA). Nó dùng JavaScript để "vẽ" 3 màn hình (Login, Register, Dashboard) mà không cần tải lại trang. 
    - Nó chịu trách nhiệm:
         - Mã hóa mật khẩu (bằng CryptoJS) trước khi gửi đi.
         - Gọi API (vd: fetch('/nodered/login')) đến backend (Node-RED).
         - Lưu phiên đăng nhập (dùng localStorage).
         - Hiển thị dữ liệu mới nhất (từ MariaDB) và biểu đồ (từ Grafana <iframe>).
<img width="1809" height="487" alt="image" src="https://github.com/user-attachments/assets/9c7506e3-3d43-41ee-b332-0066827a4d8d" />

## Kết luận
1. Có tính năng login: Đã có. File index.html có một div (view) là #login-view và hàm JavaScript handleLogin() để xử lý.
<img width="940" height="258" alt="image" src="https://github.com/user-attachments/assets/8f7ecfd8-4628-4e71-abef-56e171fd39ee" />

---
2. Yêu cầu sử dụng mã hoá khi gửi login: Đã có.
  - Trong <head>, chúng ta đã nhúng thư viện CryptoJS.
  - Trong hàm handleLogin(), chúng ta dùng CryptoJS.SHA256(password).toString(CryptoJS.enc.Hex) để mã hóa mật khẩu ngay trên trình duyệt trước khi gửi đi.
<img width="794" height="249" alt="image" src="https://github.com/user-attachments/assets/1622c6b4-6887-497b-90a0-e307eb418201" />

---
3. Thông tin login lưu trong cơ sở dữ liệu của mariadb": Đã có. Flow 2 (API /login) nhận password_hash đã mã hóa và truy vấn (SELECT) bảng users trong MariaDB để so sánh.
<img width="1581" height="402" alt="image" src="https://github.com/user-attachments/assets/e0570c5c-3a36-4730-8d6d-e675ccdcbbb2" />

---
4. Được dev quản trị bằng phpmyadmin: Đã có. Dùng http://localhost:8080 (phpMyAdmin) để chạy SQL, tạo bảng users và INSERT tài khoản admin vào.
<img width="1005" height="497" alt="image" src="https://github.com/user-attachments/assets/5fb07740-9308-4708-83d0-08358e4b351b" />

---
5. Lưu phiên đăng nhập vào cookie và session và Chỉ cần login 1 lần...: Đã có.
   - Đề bài dùng từ "cookie" và "session", nhưng em đã dùng một kỹ thuật hiện đại và phù hợp hơn với SPA là localStorage.
   - Khi login thành công, JavaScript chạy lệnh localStorage.setItem('iot_user', ...) để lưu phiên đăng nhập.
   - Khi trang web được tải, hàm checkLoginState() sẽ kiểm tra localStorage. Nếu thấy có iot_user, nó sẽ tự động chuyển vào Dashboard mà không cần login lại.
   - Nút "Đăng xuất" sẽ gọi localStorage.removeItem('iot_user'), xóa phiên đăng nhập và đưa về trang login.
<img width="502" height="256" alt="image" src="https://github.com/user-attachments/assets/53eee9cc-e592-434f-9883-5409d40ac417" />

---
6. Nginx làm web-server
   - Đã cấu hình nginx để chạy được website qua url http://haunhatninh.com 
   - Đã cấu hình nginx để http://haunhatninh.com/nodered truy cập vào nodered qua cổng 80, (dù nodered đang chạy ở port 1880)
   - Đã cấu hình nginx để http://haunhatninh.com/grafana truy cập vào grafana qua cổng 80, (dù grafana đang chạy ở port 3000)

#### => Sẽ cố gắng sửa và hoàn thiện các cấu hình còn thiếu trong thời gian sớm nhất      
