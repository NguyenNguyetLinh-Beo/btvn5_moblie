# BÀI TẬP 5 - HỌC PHẦN PHÁT TRIỂN ỨNG DỤNG MÃ NGUỒN MỞ
# NGUYỄN NGUYỆT LINH - K225480106039

# PHẦN I: LÝ THUYẾT

## 1. Khái niệm Docker là gì?

Docker là một nền tảng mã nguồn mở cho phép tự động đóng gói ứng dụng và tất cả các thành phần phụ thuộc của nó (thư viện, môi trường, cấu hình...) vào trong một đơn vị ảo hóa độc lập gọi là **Container**. 
* **Khác biệt cốt lõi:** Khác với máy ảo truyền thống (VMware, VirtualBox) phải chạy kèm cả một hệ điều hành khách (Guest OS) nặng nề, Docker Container chia sẻ chung nhân hệ điều hành (Kernel) của máy Host. Nhờ đó, Container khởi động chỉ trong vài giây, cực kỳ nhẹ và tiêu tốn rất ít tài nguyên phần cứng (RAM/CPU).

---

## 2. Các từ khóa (Keywords) trong tệp `docker-compose.yml`

| Từ khóa | Ý nghĩa cấu trúc | Ví dụ minh họa |
| :--- | :--- | :--- |
| **`version`** | Định nghĩa phiên bản cấu hình của Docker Compose được sử dụng để đảm bảo tính tương thích với Docker Engine.
| **`services`** | Khối khai báo gốc, bắt đầu danh sách các container dịch vụ sẽ được cấu hình và khởi tạo trong hệ thống. 
| **`image`** | Chỉ định tên và phiên bản bản vá (tag) của Docker Image mẫu tải từ Docker Hub để dựng container.
| **`container_name`** | Thiết lập tên cố định cho container khi chạy, giúp quản lý, kiểm tra logs và debug dễ dàng. 
| **`ports`** | Ánh xạ cổng dịch vụ công khai theo cú pháp: `Cổng_Máy_Host:Cổng_Trong_Container`. 
| **`environment`** | Thiết lập các biến môi trường cấu hình động bên trong container (như tài khoản, mật khẩu, tên DB...). 
| **`volumes`** | Gắn vùng lưu trữ dữ liệu bền vững (từ thư mục máy host hoặc volume độc lập) vào bên trong container. 
| **`networks`** | Định nghĩa mạng nội bộ cô lập để các container tham gia kết nối và giao tiếp trực tiếp với nhau bằng tên service.
| **`depends_on`** | Thiết lập thứ tự ràng buộc khởi động, ép container này phải đợi container phụ thuộc sẵn sàng trước. 
| **`command`** | Ghi đè câu lệnh thực thi mặc định bên trong container khi hệ thống bắt đầu khởi tạo (startup). 
| **`restart`** | Cấu hình chính sách tự động khởi động lại container nếu nó gặp lỗi sập hoặc crash đột ngột. 

---

## 3. Ưu điểm khi triển khai ứng dụng bằng Docker

*  Việc sử dụng Docker trong triển khai ứng dụng mang lại nhiều lợi ích quan trọng:

✔ 1. Đồng nhất môi trường triển khai

*  Ứng dụng chạy giống nhau trên mọi môi trường (máy cá nhân, server, cloud), hạn chế lỗi “chạy được trên máy tôi nhưng không chạy trên máy khác”.

✔ 2. Tối ưu tài nguyên hệ thống

*  Container nhẹ hơn nhiều so với máy ảo, không cần chạy hệ điều hành riêng, giúp tiết kiệm RAM và CPU.

 ✔ 3. Triển khai nhanh và linh hoạt

* Toàn bộ hệ thống có thể được khởi chạy chỉ bằng một câu lệnh:

docker compose up -d  

✔ 4. Dễ dàng mở rộng hệ thống

* Có thể nhân bản service nhanh chóng để mở rộng theo nhu cầu thực tế (scaling).

✔ 5. Dễ quản lý và bảo trì

* Toàn bộ kiến trúc hệ thống được mô tả bằng file cấu hình duy nhất (docker-compose.yml), giúp dễ theo dõi và chỉnh sửa.
---

## 4. Các bước triển khai ứng dụng lên Máy chủ thật KHÔNG CÓ INTERNET (Offline)
Trong thực tế nhiều máy chủ không được phép kết nối Internet vì lý do bảo mật. Vì vậy Docker cần hỗ trợ triển khai hoàn toàn Offline.

*  Bước 1: Chuẩn bị trên máy có Internet
Xây dựng hệ thống.

Kiểm thử toàn bộ chức năng.

Đảm bảo Docker Compose hoạt động ổn định.

*  Bước 2: Xuất Docker Images

Ví dụ:

docker save -o monitor_images.tar \
mariadb:11 \
influxdb:2.7 \
grafana/grafana:latest \
nodered/node-red:latest \
nginx:latest

*  Bước 3: Sao chép dữ liệu

Sao chép:

Source code
Docker Images
Docker Compose

sang máy chủ đích bằng:

USB
Ổ cứng ngoài
Mạng LAN

* Bước 4: Khôi phục

Nạp lại Image:

docker load -i monitor_images.tar

* Bước 5: Khởi động hệ thống
  
docker compose up -d

Sau khi hoàn thành, hệ thống có thể hoạt động bình thường mà không cần kết nối Internet.

# PHẦN II. XÂY DỰNG HỆ THỐNG GIÁM SÁT THỜI TIẾT REAL-TIME VÀ CẢNH BÁO TELEGRAM

## 1. Giới thiệu hệ thống

Sau khi nghiên cứu các kiến thức cơ bản về Docker và Docker Compose, tiến hành xây dựng hệ thống giám sát thời tiết thời gian thực sử dụng kiến trúc đa dịch vụ.

Hệ thống được thiết kế nhằm mục đích:

* Thu thập dữ liệu thời tiết tự động.
* Lưu trữ dữ liệu vào cơ sở dữ liệu.
* Trực quan hóa dữ liệu bằng Dashboard.
* Gửi cảnh báo Telegram khi vượt ngưỡng.
* Dễ dàng triển khai bằng Docker Compose.

Dữ liệu thời tiết được lấy từ Open-Meteo API và được cập nhật liên tục theo chu kỳ 5 giây.

---

## 2. Kiến trúc hệ thống

Hệ thống bao gồm các thành phần:

```text
Open-Meteo API
       │
       ▼
    Node-RED
   /    |    \
  /     |     \
 ▼      ▼      ▼
MariaDB InfluxDB Telegram

MariaDB
   │
   ▼
Flask API
   │
   ▼
 Nginx
   │
   ▼
Dashboard

InfluxDB
   │
   ▼
Grafana
```

### Mô tả hoạt động

#### Open-Meteo API

Là nguồn dữ liệu thời tiết miễn phí được sử dụng để lấy:

* Nhiệt độ
* Độ ẩm
* Tốc độ gió

#### Node-RED

Đóng vai trò trung tâm xử lý dữ liệu.

Chức năng:

* Gọi API thời tiết.
* Xử lý dữ liệu JSON.
* Ghi dữ liệu vào MariaDB.
* Ghi dữ liệu vào InfluxDB.
* Kiểm tra ngưỡng cảnh báo.
* Gửi Telegram Alert.

## BƯỚC 1: TẠO THƯ MỤC DỰ ÁN

<img width="719" height="418" alt="image" src="https://github.com/user-attachments/assets/36a211c9-e2f5-4148-8aa6-c447ba5936f1" />

## BƯỚC 2: TẠO MARIADB

<img width="1408" height="685" alt="image" src="https://github.com/user-attachments/assets/af62c952-aff7-43ec-ac87-b3d23245335c" />

## BƯỚC 3: TẠO DATABASE

<img width="1231" height="318" alt="image" src="https://github.com/user-attachments/assets/e6f2347f-050b-409f-ae5e-438eb5dc0ae8" />

<img width="745" height="198" alt="image" src="https://github.com/user-attachments/assets/b7dd725a-779a-4dcf-8082-bb65cccb5a48" />
## BƯỚC 4: TẠO INFLUXDB

<img width="1460" height="358" alt="image" src="https://github.com/user-attachments/assets/6399346e-d26c-4b99-9a58-507c84d965d5" />
## BƯỚC 5: KIỂM TRA NODE-RED

<img width="1919" height="885" alt="image" src="https://github.com/user-attachments/assets/1a75481d-b5d8-4457-bedb-2845cc54b2b9" />

## BƯỚC 6: LẤY DỮ LIỆU THỜI TIẾT THỰC TẾ

https://api.openweathermap.org/data/2.5/weather?q=ThaiNguyen&appid=APIKEY&units=metric

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

## BƯỚC 9: CÀI GRAFANA
<img width="889" height="453" alt="Gemini_Generated_Image_776c72776c72776c" src="https://github.com/user-attachments/assets/b395e4d5-cd7b-47ef-84ac-c1e40816562c" />
<img width="1630" height="821" alt="image" src="https://github.com/user-attachments/assets/f68ef089-a248-4366-9917-3ff75d1ddd19" />

## BƯỚC 10: TẠO FLASK API
<img width="250" height="200" alt="image" src="https://github.com/user-attachments/assets/8119ce86-24cf-48fd-b225-1d69901edce5" />

<img width="1908" height="893" alt="image" src="https://github.com/user-attachments/assets/98ac5950-eea7-4d78-b1da-5c66a9ddab82" />

<img width="1312" height="725" alt="image" src="https://github.com/user-attachments/assets/d2db8493-7814-4e2b-ad13-88ae18e1ee21" />

## BƯỚC 11: TEST API
 http://localhost:5000/api/latest
## BƯỚC 12: TẠO FRONTEND

- index.html
- script.js
  
<img width="1847" height="838" alt="Screenshot 2026-06-12 193045" src="https://github.com/user-attachments/assets/ca70f1b5-9a62-4baf-8351-fd1a71cf523e" />

## BƯỚC 13: TELEGRAM ALERT

Thiết lập cảnh báo

Trong Node Function:

```javascript
if(msg.temperature > 35)
{
    msg.payload =
    "🚨 CẢNH BÁO NHIỆT ĐỘ CAO: "
    + msg.temperature + "°C";
    return [msg,null];
}

if(msg.temperature < 20)
{
    msg.payload =
    "⚠ CẢNH BÁO NHIỆT ĐỘ THẤP: "
    + msg.temperature + "°C";
    return [null,msg];
}

return null;
```

Điều kiện:

text
Temperature > 35°C
Temperature < 20°C

Cứ mỗi 5 giây hệ thống sẽ tự động lấy dữ liệu thời tiết mới. 

<img width="941" height="887" alt="image" src="https://github.com/user-attachments/assets/b7963eb2-4b84-44e0-a9cf-51deed488b1c" />

<img width="1847" height="838" alt="Screenshot 2026-06-12 193045" src="https://github.com/user-attachments/assets/2c17fdda-4eba-468e-a0f6-df5e836b0f13" />

## BƯỚC 14: Hệ thống chạy lại thành công

<img width="1880" height="978" alt="Screenshot 2026-06-12 193321" src="https://github.com/user-attachments/assets/761ad346-dba4-482f-a073-68fb86c0928a" />

## BƯỚC 15: Cảnh báo hệ thống

<img width="1351" height="925" alt="Screenshot 2026-06-12 191137" src="https://github.com/user-attachments/assets/71cb9534-d0a0-4755-8e21-bd8bdc892935" />

<img width="1920" height="1020" alt="Screenshot 2026-06-12 191100" src="https://github.com/user-attachments/assets/af518781-11ff-4ca3-bc8d-3c4420e6c41a" />

# 16. Kết quả đạt được

Sau quá trình nghiên cứu và triển khai, hệ thống đã hoàn thành đầy đủ các chức năng đề ra.

### Chức năng đã thực hiện

✔ Thu thập dữ liệu từ Open-Meteo API

✔ Xử lý dữ liệu bằng Node-RED

✔ Lưu dữ liệu MariaDB

✔ Lưu dữ liệu InfluxDB

✔ Cung cấp REST API bằng Flask

✔ Hiển thị Dashboard Web

✔ Trực quan hóa dữ liệu bằng Grafana

✔ Gửi Telegram Alert

✔ Backup và Restore

✔ Triển khai bằng Docker Compose

---

## Ưu điểm

* Hoạt động ổn định.
* Dễ triển khai.
* Dễ mở rộng.
* Hỗ trợ môi trường Offline.
* Quản lý tập trung bằng Docker Compose.

---

## Hạn chế

* Dữ liệu phụ thuộc Open-Meteo API.
* Chưa hỗ trợ nhiều loại cảm biến thực tế.
* Chưa có cơ chế xác thực người dùng.
* Chưa triển khai trên Cloud.

---

# 17. Hướng phát triển

Trong tương lai hệ thống có thể được mở rộng theo các hướng:

### Tích hợp IoT

Thu thập dữ liệu trực tiếp từ:

* ESP32
* Arduino
* Raspberry Pi

### Machine Learning

Phân tích và dự đoán thời tiết.

### Cloud Deployment

Triển khai trên:

* AWS
* Azure
* Google Cloud

### Mobile Application

Xây dựng ứng dụng Android và iOS.

### Authentication

Bổ sung:

* Login
* JWT
* Phân quyền người dùng

---

# 18. Kết luận

Qua quá trình thực hiện đề tài "Hệ thống giám sát thời tiết Real-Time và cảnh báo Telegram sử dụng Docker Compose", nhóm đã tìm hiểu và áp dụng thành công nhiều công nghệ hiện đại trong lĩnh vực phát triển và triển khai ứng dụng.

Hệ thống cho phép thu thập dữ liệu thời tiết tự động từ Open-Meteo API, lưu trữ dữ liệu trên MariaDB và InfluxDB, hiển thị trực quan bằng Dashboard Web và Grafana, đồng thời gửi cảnh báo Telegram khi dữ liệu vượt ngưỡng cấu hình.

Việc sử dụng Docker Compose giúp đơn giản hóa quá trình triển khai và quản lý hệ thống đa dịch vụ. Các thành phần được đóng gói độc lập nhưng vẫn giao tiếp hiệu quả thông qua Docker Network.

Kết quả đạt được đáp ứng đầy đủ các yêu cầu của bài tập lớn, đồng thời tạo nền tảng để mở rộng thành các hệ thống giám sát IoT và Smart Monitoring trong thực tế.
