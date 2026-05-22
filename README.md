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

Qua bài tập này giúp sinh viên:

* Làm quen với workflow automation
* Tìm hiểu tích hợp AI API
* Hiểu cơ chế hoạt động của webhook
* Triển khai hệ thống nhiều service bằng Docker Compose
* Kết hợp nhiều nền tảng mã nguồn mở trong cùng hệ thống

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
Đăng nhập vào Cloudflare chọn **Networking -> Tunnels** để cấu hình thêm Route cho **PhpMyAdmin** và **n8n**
## Thêm route cho PhpMyAdmin và n8n
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/647ed627-cff7-4124-9120-8a1874396012" /> <br>

<img width="1616" height="875" alt="image" src="https://github.com/user-attachments/assets/eb65d9dc-4755-4c87-a763-612cd616389f" /> <br>

<img width="1645" height="856" alt="image" src="https://github.com/user-attachments/assets/9ce04ec5-b290-4817-9816-f0e44c97886c" />

## Kết quả truy cập phpmyadmin và n8n bằng tên miền vừa thêm
```
pma.tranthithuha.id.vn
```
<img width="1918" height="1029" alt="image" src="https://github.com/user-attachments/assets/19de03f7-cce0-4be0-8d3c-d6c490ed51d1" /> <br>

```
n8n.tranthithuha.id.vn
```
<img width="1809" height="949" alt="image" src="https://github.com/user-attachments/assets/746de479-c60f-4d5d-97f7-d184a9ca7570" />

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









# VII. CẤU HÌNH CLOUDFLARE TUNNEL

Cloudflare Tunnel được dùng để public các service nội bộ ra internet.

## 1. Truy cập Cloudflare Dashboard

https://dash.cloudflare.com

---

## 2. Tạo Public Hostname

### WordPress

| Field     | Value        |
| --------- | ------------ |
| Subdomain | blog         |
| Type      | HTTP         |
| URL       | wordpress:80 |

---

### phpMyAdmin

| Field     | Value         |
| --------- | ------------- |
| Subdomain | pma           |
| Type      | HTTP          |
| URL       | phpmyadmin:80 |

---

### n8n

| Field     | Value    |
| --------- | -------- |
| Subdomain | n8n      |
| Type      | HTTP     |
| URL       | n8n:5678 |

---

# VIII. CÀI ĐẶT WORDPRESS

## 1. Truy cập website

```text id="2y9l2u"
https://blog.tenmien.com
```

---

## 2. Cài đặt WordPress

Nhập:

* Tên website
* Username admin
* Password
* Email

Sau khi hoàn tất WordPress sẽ:

* Tự tạo bảng dữ liệu trong MariaDB
* Tạo tài khoản quản trị

---

# IX. KIỂM TRA DATABASE

## 1. Truy cập phpMyAdmin

```text id="y1rgcp"
https://pma.tenmien.com
```

---

## 2. Quan sát database

Ban đầu database chưa có bảng.

Sau khi cài WordPress sẽ xuất hiện:

* wp_posts
* wp_users
* wp_options
* wp_comments
* wp_terms

Điều này cho thấy WordPress đã kết nối thành công với MariaDB.

---

# X. TẠO BÀI VIẾT THỦ CÔNG

Sau khi cài WordPress, tiến hành tạo:

* Bài giới thiệu bản thân
* Bài giới thiệu môn học

Mục đích:

* Kiểm tra WordPress hoạt động bình thường
* Làm dữ liệu mẫu cho website

---

# XI. CẤU HÌNH N8N

# 1. Truy cập n8n

```text id="7m6ysp"
https://n8n.tenmien.com
```

---

# 2. Tạo tài khoản admin

Khi truy cập lần đầu:

* nhập email
* nhập mật khẩu

Lưu ý:

* nên sử dụng email thật để nhận license key

---

# 3. Activate License Key

Sau khi đăng ký:

* n8n gửi activation key qua email

Tiến hành:

* Settings
* Usage and Plan
* Enter activation key

Kết quả:

* Community Edition được kích hoạt thành công

---

# XII. TẠO TELEGRAM BOT

## 1. Mở Telegram

Tìm bot:

```text id="brk4q4"
@BotFather
```

---

## 2. Tạo bot mới

Gõ:

```text id="mn83h6"
/newbot
```

---

## 3. Đặt tên bot

Ví dụ:

```text id="9qf7h2"
my_ai_blog_bot
```

---

## 4. Nhận Access Token

BotFather sẽ cung cấp:

* HTTP API Token

Token này dùng để kết nối Telegram với n8n.

---

## 5. Chat thử với bot

Phải gửi ít nhất một tin nhắn cho bot để Telegram tạo dữ liệu chat.

Ví dụ:

```text id="w8g9gh"
Xin chào
```

---

# XIII. TẠO GOOGLE GEMINI API KEY

## 1. Truy cập

https://aistudio.google.com/api-keys

---

## 2. Tạo API KEY

* Create API Key
* Copy API Key

API KEY dùng để n8n gửi prompt tới Gemini AI.

---

# XIV. XÂY DỰNG WORKFLOW N8N

# 1. Telegram Trigger

Node đầu tiên dùng để nhận dữ liệu từ Telegram Bot.

Cấu hình:

* Node: Telegram Trigger
* Operation: On Message

Credential:

* nhập Telegram Access Token

Khi người dùng gửi tin nhắn:

* node sẽ tự động kích hoạt workflow

---

# 2. Google Gemini AI

Node này gửi nội dung người dùng tới Gemini AI để sinh bài viết.

Prompt:

```text id="j3t3qy"
{{ $json.message.text }}

Hãy viết bài blog chuyên nghiệp bằng tiếng Việt.

Kết quả trả về đúng JSON format:

{
  "post_title": "...",
  "post_content": "..."
}

post_content phải chứa HTML + CSS đẹp để đăng WordPress.
```

---

## Giải thích

Trong đó:

* {{ $json.message.text }} là nội dung người dùng gửi từ Telegram
* AI sẽ sinh:

  * tiêu đề bài viết
  * nội dung HTML

Bật:

* Output Content as JSON

để dữ liệu trả về đúng định dạng JSON.

---

# 3. Code JavaScript

Node JavaScript dùng để:

* đọc dữ liệu AI trả về
* tách title và content

Code:

```javascript id="pkgnfh"
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. chuyển JSON string thành object
const cleanData = JSON.parse(rawText);

// 3. trả dữ liệu
return [
  {
    json: {
      title: cleanData.post_title,
      content: cleanData.post_content
    }
  }
];
```

---

## Giải thích

* JSON.parse():
  chuyển chuỗi JSON thành object JavaScript

* title:
  tiêu đề bài viết

* content:
  nội dung HTML bài viết

---

# 4. WordPress Create Post

Node này dùng để đăng bài lên WordPress.

Credential:

* URL website
* Username WordPress
* Application Password

---

# XV. TẠO APPLICATION PASSWORD WORDPRESS

## 1. Truy cập WordPress Admin

```text id="c6b7ho"
https://blog.tenmien.com/wp-admin
```

---

## 2. Tạo mật khẩu ứng dụng

Vào:

* Users
* Profile
* Application Passwords

Nhập:

* tên ứng dụng: n8n

Bấm:

* Add New Application Password

WordPress sẽ sinh chuỗi password gồm 24 ký tự.

Password này được dùng để n8n gọi WordPress API.

---

# XVI. CẤU HÌNH WORDPRESS NODE

Map dữ liệu:

* Title ← title
* Content ← content

Add field:

* Status = publish

Ý nghĩa:

* bài viết sẽ được xuất bản ngay lập tức

---

# XVII. PUBLISH WORKFLOW

Sau khi hoàn thành workflow:

* bấm nút Publish

Khi đó workflow sẽ:

* tự động chạy mỗi khi Telegram gửi tin nhắn

---

# XVIII. KIỂM THỬ HỆ THỐNG

## 1. Gửi tin nhắn Telegram

Ví dụ:

```text id="vgt7km"
Viết bài giới thiệu về AI trong giáo dục
```

---

## 2. Workflow hoạt động

```text id="m2fcsv"
Telegram
→ Telegram Trigger
→ Gemini AI
→ Code JavaScript
→ WordPress Create Post
```

---

## 3. Kết quả

Sau vài giây:

* bài viết tự động xuất hiện trên WordPress

Điều này chứng minh:

* hệ thống automation hoạt động thành công

---

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
