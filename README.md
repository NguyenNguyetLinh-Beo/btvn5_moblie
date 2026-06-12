# BÀI TẬP 5 - HỌC PHẦN PHÁT TRIỂN ỨNG DỤNG MÃ NGUỒN MỞ
# NGUYỄN NGUYỆT LINH - K225480106039

# PHẦN I: LÝ THUYẾT

## 1. Khái niệm Docker là gì?
<img width="300" height="168" alt="image" src="https://github.com/user-attachments/assets/b3044680-db52-44b4-a798-7c9ff69d50f4" />

Docker là một nền tảng mã nguồn mở cho phép tự động đóng gói ứng dụng và tất cả các thành phần phụ thuộc của nó (thư viện, môi trường, cấu hình...) vào trong một đơn vị ảo hóa độc lập gọi là **Container**. 
* **Khác biệt cốt lõi:** Khác với máy ảo truyền thống (VMware, VirtualBox) phải chạy kèm cả một hệ điều hành khách (Guest OS) nặng nề, Docker Container chia sẻ chung nhân hệ điều hành (Kernel) của máy Host. Nhờ đó, Container khởi động chỉ trong vài giây, cực kỳ nhẹ và tiêu tốn rất ít tài nguyên phần cứng (RAM/CPU).

---

## 2. Các từ khóa (Keywords) trong tệp `docker-compose.yml`

| Từ khóa | Ý nghĩa cấu trúc | Ví dụ minh họa |
| :--- | :--- | :--- |
| **`version`** | Định nghĩa phiên bản cấu hình của Docker Compose được sử dụng để đảm bảo tính tương thích với Docker Engine. | `version: '3.8'` |
| **`services`** | Khối khai báo gốc, bắt đầu danh sách các container dịch vụ sẽ được cấu hình và khởi tạo trong hệ thống. | `services:`<br>`  mariadb:` |
| **`image`** | Chỉ định tên và phiên bản bản vá (tag) của Docker Image mẫu tải từ Docker Hub để dựng container. | `image: influxdb:1.8` |
| **`container_name`** | Thiết lập tên cố định cho container khi chạy, giúp quản lý, kiểm tra logs và debug dễ dàng. | `container_name: app_nodered` |
| **`ports`** | Ánh xạ cổng dịch vụ công khai theo cú pháp: `Cổng_Máy_Host:Cổng_Trong_Container`. | `ports:`<br>`  - "80:80"` |
| **`environment`** | Thiết lập các biến môi trường cấu hình động bên trong container (như tài khoản, mật khẩu, tên DB...). | `environment:`<br>`  MYSQL_ROOT_PASSWORD: rootpassword` |
| **`volumes`** | Gắn vùng lưu trữ dữ liệu bền vững (từ thư mục máy host hoặc volume độc lập) vào bên trong container. | `volumes:`<br>`  - ./init.sql:/docker-entrypoint-initdb.d/init.sql` |
| **`networks`** | Định nghĩa mạng nội bộ cô lập để các container tham gia kết nối và giao tiếp trực tiếp với nhau bằng tên service. | `networks:`<br>`  - monitor_net` |
| **`depends_on`** | Thiết lập thứ tự ràng buộc khởi động, ép container này phải đợi container phụ thuộc sẵn sàng trước. | `depends_on:`<br>`  - mariadb` |
| **`command`** | Ghi đè câu lệnh thực thi mặc định bên trong container khi hệ thống bắt đầu khởi tạo (startup). | `command: sh -c "python app.py"` |
| **`restart`** | Cấu hình chính sách tự động khởi động lại container nếu nó gặp lỗi sập hoặc crash đột ngột. | `restart: always` |

---

## 3. Ưu điểm khi triển khai ứng dụng sử dụng Docker
* **Tính nhất quán môi trường (Write Once, Run Anywhere):** Loại bỏ triệt để lỗi *"Chạy trên máy cá nhân mượt mà nhưng lên máy chủ bị lỗi môi trường"*. Container đóng gói cô lập hoàn toàn.
* **Tối ưu hóa tài nguyên phần cứng:** Khởi động siêu tốc trong vài giây, chiếm dụng dung lượng ổ cứng và bộ nhớ RAM cực ít so với ảo hóa máy ảo VM thông thường.
* **Triển khai và mở rộng tự động:** Quản lý toàn bộ hạ tầng phức tạp bao gồm nhiều lớp dịch vụ chỉ thông qua một tệp cấu hình mã nguồn duy nhất, dễ dàng nhân bản và nâng cấp.

---

## 4. Các bước triển khai ứng dụng lên Máy chủ thật KHÔNG CÓ INTERNET (Offline)
* **Bước 1 (Chuẩn bị trên máy mạng):** Xây dựng, cấu hình các tệp mã nguồn và chạy thử nghiệm hệ thống hoàn chỉnh bằng Docker Compose trên máy tính cá nhân có kết nối Internet ổn định.
* **Bước 2 (Xuất đóng gói dữ liệu):** Sử dụng câu lệnh `docker save` để nén toàn bộ các cấu trúc Docker Images cần dùng thành các tệp tin lưu trữ offline có định dạng đuôi `.tar`.
* **Bước 3 (Di chuyển tài nguyên):** Sử dụng các thiết bị ngoại vi độc lập (USB, ổ cứng di động, mạng LAN nội bộ cô lập) để sao chép toàn bộ các tệp `.tar` này cùng với thư mục mã nguồn dự án (chứa file `docker-compose.yml`, code web, code API) sang máy chủ thật.
* **Bước 4 (Kích hoạt tại máy chủ Offline):** Tại máy chủ thật (đã được cài sẵn Docker Engine offline), sử dụng câu lệnh `docker load` để nạp các tệp `.tar` vào kho quản lý image cục bộ, sau đó khởi chạy hệ thống bằng lệnh `docker compose up -d` mà hoàn toàn không cần chạm vào Internet.

# PHẦN II: QUY TRÌNH THỰC HIỆN ỨNG DỤNG MONITOR & ALERT DATA REAL-TIME

Hệ thống tiến hành giám sát luồng dữ liệu **Giá Bitcoin biến động liên tục từ nguồn thực tế**. Hệ thống sở hữu cơ chế lưu trữ song song (MariaDB lưu tức thời, InfluxDB lưu lịch sử), trực quan hóa biểu đồ (Grafana), phân phối giao diện đích (Nginx làm Webserver + Flask làm API) và tự động gửi cảnh báo vượt ngưỡng an toàn về nhóm Telegram gồm 3 thành viên.

---

## Bước 1: Chuẩn bị cấu trúc thư mục dự án và các tệp mã nguồn

Di chuyển vào thư mục dự án `~/ma` (Monitor - Alert) và thiết lập cấu trúc cây thư mục tĩnh như sau:

```text
~/ma/
├── docker-compose.yml
├── init.sql
├── flask-api/
│   ├── app.py
│   └── requirements.txt
└── nginx/
    ├── default.conf
    └── web/
        └── index.html
```
Các lệnh tạo nhanh file và thư mục: 

## BƯỚC 1: TẠO THƯ MỤC DỰ ÁN
<img width="719" height="418" alt="image" src="https://github.com/user-attachments/assets/36a211c9-e2f5-4148-8aa6-c447ba5936f1" />
## 
<img width="1408" height="685" alt="image" src="https://github.com/user-attachments/assets/af62c952-aff7-43ec-ac87-b3d23245335c" />
## 
<img width="1231" height="318" alt="image" src="https://github.com/user-attachments/assets/e6f2347f-050b-409f-ae5e-438eb5dc0ae8" />
<img width="745" height="198" alt="image" src="https://github.com/user-attachments/assets/b7dd725a-779a-4dcf-8082-bb65cccb5a48" />
## BƯỚC 4: CHẠY HỆ THỐNG
<img width="1460" height="358" alt="image" src="https://github.com/user-attachments/assets/6399346e-d26c-4b99-9a58-507c84d965d5" />
## BƯỚC 5: KIỂM TRA NODE-RED
<img width="1919" height="885" alt="image" src="https://github.com/user-attachments/assets/1a75481d-b5d8-4457-bedb-2845cc54b2b9" />
## BƯỚC 7: KIỂM TRA INFLUXDB
<img width="1884" height="989" alt="image" src="https://github.com/user-attachments/assets/270acdfe-a9e1-4797-87d0-83c272dbe275" />
<img width="1790" height="810" alt="image" src="https://github.com/user-attachments/assets/4f3e7786-2d73-4449-b0fe-5e4f633370da" />
## BƯỚC 8: TẠO DATABASE MARIADB
<img width="764" height="602" alt="image" src="https://github.com/user-attachments/assets/799c454f-cc74-4eac-8e6f-93324a95e186" />
<img width="726" height="290" alt="image" src="https://github.com/user-attachments/assets/fa0b59fb-232b-4d1e-a964-f1b5a45b1d6d" />
<img width="1262" height="653" alt="image" src="https://github.com/user-attachments/assets/44ce48c4-8169-40b6-bb90-66f3194fe3ef" />
<img width="1214" height="611" alt="image" src="https://github.com/user-attachments/assets/07bdb50f-8adb-4bf6-942e-6441ad9171d0" />
<img width="1910" height="906" alt="image" src="https://github.com/user-attachments/assets/5aac09a3-dbf5-445e-a6e4-99a0bdc11620" />
<img width="1219" height="714" alt="image" src="https://github.com/user-attachments/assets/83ba713a-91b9-453f-a270-3a3a97052414" />
<img width="924" height="542" alt="image" src="https://github.com/user-attachments/assets/476d9dd3-42eb-4375-9238-ea6007711d86" />
<img width="1343" height="784" alt="Gemini_Generated_Image_ydgsbnydgsbnydgs" src="https://github.com/user-attachments/assets/e0d9575e-df2a-4ac9-a53a-c9f3272168c2" />
BƯỚC 10: TẠO FLASK API
<img width="250" height="200" alt="image" src="https://github.com/user-attachments/assets/8119ce86-24cf-48fd-b225-1d69901edce5" />
<img width="889" height="453" alt="Gemini_Generated_Image_776c72776c72776c" src="https://github.com/user-attachments/assets/b395e4d5-cd7b-47ef-84ac-c1e40816562c" />
<img width="1630" height="821" alt="image" src="https://github.com/user-attachments/assets/f68ef089-a248-4366-9917-3ff75d1ddd19" />
<img width="1908" height="893" alt="image" src="https://github.com/user-attachments/assets/98ac5950-eea7-4d78-b1da-5c66a9ddab82" />
