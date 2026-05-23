# Môn: Phát triển ứng dụng với mã nguồn mở - TEE0421
-----
## HỌ TÊN: TRẦN THỊ THU HÀ
## MSSV: K225480106009
## LỚP: K58KTP.01
----
## BÀI TẬP 4: KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS
### Deadline: 23h59 ngày 25 tháng 5 năm 2026.
---

# I. GIỚI THIỆU ĐỀ TÀI

Trong thời đại hiện nay, việc tự động hóa các tác vụ bằng AI và các nền tảng workflow automation đang được ứng dụng rộng rãi trong thực tế. Một trong những ứng dụng phổ biến là tự động tạo nội dung và đăng tải bài viết lên website.

Ở bài tập này, hệ thống WordPress đã được triển khai từ bài tập trước bằng Docker Compose. Nhiệm vụ tiếp theo là bổ sung thêm service n8n để xây dựng hệ thống tự động đăng bài lên WordPress thông qua Telegram Bot và Google Gemini AI.

Người dùng chỉ cần gửi nội dung qua Telegram Bot, hệ thống sẽ tự động:

* Nhận tin nhắn từ Telegram
* Gửi prompt tới Google Gemini AI
* Sinh nội dung bài viết dạng HTML
* Tự động đăng bài lên WordPress

---

# II. MỤC TIÊU BÀI TẬP

* Triển khai nhiều service bằng Docker Compose
* Sử dụng MariaDB làm cơ sở dữ liệu cho WordPress
* Public hệ thống bằng Cloudflare Tunnel
* Tích hợp Telegram Bot API
* Tích hợp Google Gemini AI
* Sử dụng n8n để xây dựng workflow automation
* Tự động tạo và đăng bài viết lên WordPress

---

# III. MÔ HÌNH HỆ THỐNG

## 1. Kiến trúc tổng thể

```text id="6r1m2x"
Telegram User
      ↓
Telegram Bot
      ↓
Telegram Trigger (n8n)
      ↓
Google Gemini AI
      ↓
Code JavaScript
      ↓
WordPress API
      ↓
Website WordPress
```

---

## 2. Nguyên lý hoạt động

Hệ thống hoạt động theo quy trình sau:

1. Người dùng gửi nội dung tới Telegram Bot
2. Telegram Trigger trong n8n nhận được tin nhắn
3. Nội dung được gửi tới Google Gemini AI
4. AI sinh tiêu đề và nội dung bài viết dạng HTML
5. Node JavaScript xử lý dữ liệu JSON trả về
6. Node WordPress sử dụng API để đăng bài viết
7. Bài viết xuất hiện trực tiếp trên website WordPress

---

# IV. CÁC CÔNG NGHỆ SỬ DỤNG

| Công nghệ         | Chức năng                  |
| ----------------- | -------------------------- |
| Docker Compose    | Quản lý nhiều container    |
| MariaDB           | Hệ quản trị cơ sở dữ liệu  |
| phpMyAdmin        | Quản lý database           |
| WordPress         | Hệ quản trị nội dung       |
| Cloudflare Tunnel | Public service ra internet |
| n8n               | Workflow automation        |
| Telegram Bot API  | Nhận dữ liệu người dùng    |
| Google Gemini AI  | Sinh nội dung AI           |

---

# V. CẬP NHẬT DOCKER COMPOSE & MẠNG TUNNEL

# 1. Truy cập vào thư mục `thuha_wordpress`  (thư mục bài tập 3)
## Bổ sung service n8n vào `file docker-compose.yml`
Mở file `docker-compose.yml` và thêm n8n xuống cuối cùng của mục services:
```
nano docker-compose.yml
```

```
 n8n:
    image: n8nio/n8n:latest
    container_name: n8n_service
    restart: always
    ports:
      - "5678:5678"
    environment:
      - TZ=Asia/Ho_Chi_Minh
      - WEBHOOK_URL=http://n8n.tranthithuha.id.vn/
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - db 

volumes:
  thuha_db_data:
  thuha_wp_data:
  n8n_data:
```
## Khởi động lại hệ thống
Chạy lệnh sau để Docker nhận diện service mới và tự động tải image n8n về chạy song song với các service cũ. 

```
docker compose up -d
```

# 2. Cấu hình thêm Route trên Cloudflare Dashboard
- Cấu hình Tunel cloudflare để truy cập vào các dịch vụ bằng các subdomain
- Đăng nhập vào Cloudflare chọn **Networking -> Tunnels** để cấu hình thêm Route cho **PhpMyAdmin** và **n8n**
## Thêm route cho PhpMyAdmin và n8n
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/647ed627-cff7-4124-9120-8a1874396012" /> <br>

<img width="1616" height="875" alt="image" src="https://github.com/user-attachments/assets/eb65d9dc-4755-4c87-a763-612cd616389f" /> <br>

<img width="1645" height="856" alt="image" src="https://github.com/user-attachments/assets/9ce04ec5-b290-4817-9816-f0e44c97886c" />

## Kết quả truy cập phpmyadmin và n8n bằng các sub-domain
Kết quả truy cập sub-doamin của phpmyadmin:
```
pma.tranthithuha.id.vn
```
<img width="1918" height="1029" alt="image" src="https://github.com/user-attachments/assets/19de03f7-cce0-4be0-8d3c-d6c490ed51d1" /> <br>

Kết quả truy cập sub-domain của n8n:
```
n8n.tranthithuha.id.vn
```
<img width="1809" height="949" alt="image" src="https://github.com/user-attachments/assets/746de479-c60f-4d5d-97f7-d184a9ca7570" /> <br>

Các bảng dữ liệu có sau khi cài wordpress:<br>
<img width="1919" height="986" alt="image" src="https://github.com/user-attachments/assets/7a7cdedd-0a84-4665-9c57-681b2fa094b5" /> <br>

Tạo bài viết trong wordpress giới thiệu bản thân sinh viên: <br>
<img width="1008" height="879" alt="image" src="https://github.com/user-attachments/assets/1707f22d-c947-49f4-ac71-6e123564c717" /> <br>

Tạo bài viết giới thiệu những kiến thức mà em đã học được trong môn học Phát triển ứng dụng với mã nguồn mở: <br>
<img width="1580" height="863" alt="image" src="https://github.com/user-attachments/assets/0c01d911-03a3-4f1b-97cb-b11193a22924" />

---


# VI. CẤU HÌNH LIÊN KẾT N8N VỚI TELEGRAM VÀ GEMINI
## BƯỚC 1. ĐĂNG KÝ TÀI KHOẢN ADMIN VÀ KÍCH HOẠT LICENSE N8N 
### Truy cập vào trang quản trị n8n qua **n8n.tranthithuha.id.vn** vừa cấu hình. Điền thông tin đăng ký tài khoản Admin <br>
<img width="1706" height="933" alt="image" src="https://github.com/user-attachments/assets/237167f1-02ab-44be-a66a-226e122d728c" /> <br>

### Chọn Send me a free license key:
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/ad1f0ade-3dd1-492e-9d89-6e353d3309b1" /> <br>

### Activate license key: 
<img width="1916" height="946" alt="image" src="https://github.com/user-attachments/assets/7b6218b7-51dc-4672-8fa0-544b06ff7cb8" /> <br>

<img width="1591" height="749" alt="image" src="https://github.com/user-attachments/assets/599364d5-b5d0-4fc0-b683-95146e41a221" /> <br>

## BƯỚC 2. TẠO VÀ CẤU HÌNH NODE TELEGRAM TRIGGER

### Tạo workflow
<img width="1919" height="974" alt="image" src="https://github.com/user-attachments/assets/fc75cbcb-cb79-42e5-9e1c-85ecc90b0fa0" /> <br>

<img width="1910" height="980" alt="image" src="https://github.com/user-attachments/assets/2325805d-162c-4d81-9ec0-fa4ece4774ad" />

### Tạo bot trên ứng dụng Telegram
- Mở Telegram trên điện thoại hoặc máy tính và tìm kiếm bot có tên `@BotFather` có tích xanh chính chủ
- Bấm **Start** rồi gõ lệnh `/newbot` <br>
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1e288793-8c69-4ca9-b0e6-3e25e736e3f1" /> <br>

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/2fffb3e9-7850-41a0-983b-6ff5b45e1075" /> <br>

- Đặt tên hiển thị (Name): Gõ tên bất kỳ. Ví dụ: `Thu Ha AI Automation`
- Đặt tên định danh (Username): Viết liền không dấu, bắt buộc phải kết thúc bằng chữ `bot` hoặc `_bot`. Ví dụ: `thuha_ai_wp_bot` 
- @BotFather sẽ gửi lại một tin nhắn chúc mừng, trong đó có một chuỗi ký tự gọi là HTTP API Token <br>
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/39cc1bfc-4db4-43da-bc7d-7e1c9b2e6bef" />

### Cấu hình Node trên n8n
<img width="1909" height="917" alt="image" src="https://github.com/user-attachments/assets/853b0bd0-7daa-471a-910f-e0297518b7cb" /> <br>

<img width="1862" height="876" alt="image" src="https://github.com/user-attachments/assets/20aafebc-b66f-4b47-abb3-38b4314fb797" /> <br>

<img width="1863" height="863" alt="image" src="https://github.com/user-attachments/assets/9ce7fe66-b2c8-4c7e-b5c3-e3c561a5aa5f" /> <br>

Chat với bot mới lần đầu tiên: <br>
<img width="1919" height="1058" alt="image" src="https://github.com/user-attachments/assets/149dc82a-0b58-4f91-901e-724d6ae5243c" />














# XIX. KẾT QUẢ ĐẠT ĐƯỢC

Sau khi hoàn thành bài tập:

* triển khai thành công nhiều container bằng Docker Compose
* cấu hình WordPress kết nối MariaDB
* public service bằng Cloudflare Tunnel
* tích hợp Telegram Bot API
* tích hợp Google Gemini AI
* sử dụng n8n để tự động hóa workflow
* tự động sinh bài viết HTML bằng AI
* tự động đăng bài lên WordPress

---

# XX. NHẬN XÉT

Ưu điểm:

* Tự động hóa hoàn toàn quy trình đăng bài
* Dễ mở rộng thêm tính năng
* Không cần mở port router
* Dễ triển khai bằng Docker

Khó khăn:

* Cấu hình Cloudflare Tunnel khá phức tạp
* Xử lý JSON AI trả về cần chính xác
* Cần hiểu workflow của n8n

---

# XXI. KẾT LUẬN

Bài tập giúp sinh viên tiếp cận:

* Docker và containerization
* Workflow automation
* AI integration
* API communication
* Hệ thống mã nguồn mở hiện đại

Ngoài ra bài tập còn giúp hiểu cách kết hợp:

* AI
* Automation
* CMS
* API
* Cloud Tunnel

để xây dựng một hệ thống thực tế hoàn chỉnh.
