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
   - Lợi ích: Bạn chỉ cần dùng một lệnh duy nhất (`docker-compose up`) để khởi động hoặc tắt toàn bộ 6 services. Nó cũng tự động tạo một mạng riêng (`iot_net`) để chúng "nói chuyện" với nhau.
<img width="1920" height="1080" alt="Screenshot (43)" src="https://github.com/user-attachments/assets/c0ccfc1f-bf3c-4a46-b59d-d5883e7c3987" />

<img width="1920" height="1080" alt="Screenshot (44)" src="https://github.com/user-attachments/assets/9fdbc01d-57c8-4947-b611-e964ebbcaf04" />

----
##### Các Services (6 Container)
    - MariaDB (CSDL Quan hệ):
         - Mục đích: Để lưu trữ dữ liệu có cấu trúc (dạng bảng).
         - Trong bài này: Dùng để lưu 2 thứ:
         - Bảng users: Lưu tên đăng nhập, họ tên, và mật khẩu đã mã hóa (hash).
         - Bảng latest_data: Lưu giá trị mới nhất của cảm biến (để hiển thị nhanh lên dashboard).

    - phpMyAdmin (Quản trị CSDL):
         - Mục đích: Cung cấp một giao diện web (giống như "trình duyệt") để nhìn và thao tác với dữ liệu trong MariaDB. Dùng nó để tạo bảng, thêm/sửa/xóa user (admin) mà không cần gõ lệnh SQL phức tạp.
<img width="1407" height="672" alt="image" src="https://github.com/user-attachments/assets/91f5098e-8b2b-42a9-a3a5-174918fc97cb" />
         
    - Node-RED (Backend - "Bộ não"):
         - Mục đích: Đây là "backend" (bộ não) của toàn bộ ứng dụng.
         - Trong bài này: Nó làm 4 nhiệm vụ chính:
         - Tạo API: Tạo ra các đường dẫn (endpoints) như /login, /register, /latest-data.
         - Giả lập Cảm biến: Tự động tạo ra dữ liệu (nhiệt độ, độ ẩm) sau mỗi 5 giây.
         - Xử lý Logic: Nhận yêu cầu từ web (vd: login), kiểm tra CSDL (MariaDB), và trả về kết quả (JSON).
         - Phân phối Dữ liệu: Ghi dữ liệu mới nhất vào MariaDB (Flow 3) VÀ ghi dữ liệu lịch sử vào InfluxDB (Flow 1).
<img width="1322" height="941" alt="image" src="https://github.com/user-attachments/assets/e07d9c6a-db60-4a4a-8015-01e2b1ca4a30" />
### Hiện tại vẫn đang lỗi không thể ghi dữ liệu lịch sử vào InfluxDB.
  
    - InfluxDB (CSDL Chuỗi thời gian):
         - Mục đích: Đây là CSDL chuyên dụng để lưu lịch sử dữ liệu theo thời gian.
         - Khác biệt: MariaDB chỉ lưu giá trị mới nhất (vd: nhiệt độ = 25). InfluxDB lưu tất cả (8:00:00 nhiệt độ 25, 8:00:05 nhiệt độ 25.1, v.v.).
         - Trong bài này: Dùng để lưu toàn bộ lịch sử nhiệt độ, độ ẩm để cho Grafana vẽ biểu đồ.
<img width="1909" height="980" alt="image" src="https://github.com/user-attachments/assets/d973f870-0208-4da2-a699-62a46fc81b0c" />

### Do Node-RED không ghi được vào InfluxDB nên chưa thể lấy lịch sử nhiệt độ, độ ẩm.
    - Tác động đến InfluxDB
        - Vai trò: Trong bài này, Node-RED là "Nhà sản xuất" dữ liệu lịch sử (cứ 5 giây nó tạo ra nhiệt độ/độ ẩm). InfluxDB là "Kho chứa" dữ liệu đó.
        - Hậu quả: Khi Node-RED không ghi vào được, "kho chứa" InfluxDB sẽ hoàn toàn trống rỗng (hoặc chỉ chứa dữ liệu rác từ các bài test curl cũ). Mặc dù bản thân dịch vụ InfluxDB vẫn chạy, nhưng cái "xô" (iotbucket) không nhận được bất kỳ dữ liệu nào.

    - Grafana (Hiển thị Biểu đồ):
         - Mục đích: Công cụ chuyên dụng để vẽ biểu đồ đẹp và phức tạp.
         - Trong bài này: Nó kết nối trực tiếp với InfluxDB (nguồn dữ liệu lịch sử), tạo ra biểu đồ, và cung cấp một link <iframe> để bạn nhúng vào index.html.

    - Nginx (Web Server & Reverse Proxy):
         - Mục đích: Đây là "bộ mặt" và là "cổng gác" của hệ thống.
         - Trong bài này: Nó làm 2 nhiệm vụ:
         - Web Server: Phục vụ file index.html (trang SPA của bạn) khi người dùng truy cập http://haunhatninh.com.
         - Reverse Proxy (Cổng gác): Nhận tất cả truy cập qua cổng 80. Khi thấy /nodered, nó bí mật chuyển hướng (proxy) đến nodered:1880. Khi thấy /grafana, nó chuyển đến grafana:3000.

##### 🖥️ Ứng dụng Web (Frontend)

    - index.html (SPA) và JavaScript:
         - Mục đích: Đây là "frontend" (bộ mặt) mà người dùng nhìn thấy và tương tác.
         - Trong bài này: Nó là một "Ứng dụng Trang đơn" (SPA). Nó dùng JavaScript để "vẽ" 3 màn hình (Login, Register, Dashboard) mà không cần tải lại trang. 
    - Nó chịu trách nhiệm:
         - Mã hóa mật khẩu (bằng CryptoJS) trước khi gửi đi.
         - Gọi API (vd: fetch('/nodered/login')) đến backend (Node-RED).
         - Lưu phiên đăng nhập (dùng localStorage).
         - Hiển thị dữ liệu mới nhất (từ MariaDB) và biểu đồ (từ Grafana <iframe>).
