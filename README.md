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
<img width="1837" height="959" alt="image" src="https://github.com/user-attachments/assets/f729fe05-1984-4b3f-94eb-d34e3e592444" /> <br>

### Chọn Send me a free license key:
 <img width="1826" height="945" alt="image" src="https://github.com/user-attachments/assets/3ca17eaa-0534-4d26-b71d-af7c82b692d4" /> <br>

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
- Đặt tên định danh (Username): Viết liền không dấu, bắt buộc phải kết thúc bằng chữ `bot` hoặc `_bot`. Ví dụ: `thuha_wp_bot` 
- @BotFather sẽ gửi lại một tin nhắn chúc mừng, trong đó có một chuỗi ký tự gọi là HTTP API Token <br>
<img width="1919" height="1021" alt="image" src="https://github.com/user-attachments/assets/17153f85-a811-48e7-8589-9e7796283aaa" />

### Cấu hình Node trên n8n
<img width="1909" height="917" alt="image" src="https://github.com/user-attachments/assets/853b0bd0-7daa-471a-910f-e0297518b7cb" /> <br>

<img width="1862" height="876" alt="image" src="https://github.com/user-attachments/assets/20aafebc-b66f-4b47-abb3-38b4314fb797" /> <br>

<img width="1863" height="863" alt="image" src="https://github.com/user-attachments/assets/9ce7fe66-b2c8-4c7e-b5c3-e3c561a5aa5f" /> <br>

<img width="1879" height="896" alt="image" src="https://github.com/user-attachments/assets/cfd74dd0-37a2-4eee-b1e0-e0636b44212e" /> <br>

Chat với bot mới lần đầu tiên: <br>
<img width="1919" height="1028" alt="image" src="https://github.com/user-attachments/assets/f88510d3-bd86-44e3-b464-1b692bd73e0f" /> <br>

<img width="1535" height="762" alt="image" src="https://github.com/user-attachments/assets/f8edd8b4-7db6-4740-8037-ac5580da128e" />

## BƯỚC 3. THÊM VÀ CẤU HÌNH NODE GOOGLE GEMINI
### Lấy API key từ Google AI Studio
- Truy cập vào Google AI Studio: `https://aistudio.google.com/api-keys?project=gen-lang-client-0148436295`
- Đăng nhập bằng tài khoản Google và chọn **API keys** -> **Create API key** <br>
<img width="1907" height="975" alt="image" src="https://github.com/user-attachments/assets/86c9e95f-d5e9-4db2-bd26-f8c236255e1c" /> <br>

### Trên n8n thêm và cấu hình node Google Gemini
- Trên n8n tại worflow vừa cấu hình node Telegram Trigger, nhấn dấu `+` sau node Telegram Trigger => tìm kiếm và chọn node **Google Gemini** => Chọn **Message a Model** => **Set up Credential** => Dán API vừa cop trên google vào => Save <br>
<img width="1919" height="1023" alt="image" src="https://github.com/user-attachments/assets/e25c3411-9031-497e-a243-08b718ccd4b6" /> <br>

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/88c7235d-b880-4ccd-8d59-82275cb63e60" /> <br>

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/f44b06cf-8ed5-4069-be45-8d47a2b5e9e6" /> <br>

<img width="1919" height="966" alt="image" src="https://github.com/user-attachments/assets/35410604-1c4a-4ef1-81b9-2176ac0dffcb" /> <br>

<img width="1919" height="951" alt="image" src="https://github.com/user-attachments/assets/e839660b-f6cd-42ba-bd29-d1feb637e331" /> <br>

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d156dae1-3187-49fe-9fd5-eb17a5b8806e" /> <br>

<img width="683" height="229" alt="image" src="https://github.com/user-attachments/assets/f9bd1a2d-18a2-48b5-8316-096ef5b521a9" />


## BƯỚC 4. THÊM VÀ CẤU HÌNH NODE CODE IN JAVASCRIPT
- Nhấn dấu + sau node Gemini, tìm kiếm node Code (chọn ngôn ngữ JavaScript).
- Chọn chế độ: Run once for each item.
- Xóa code mặc định và dán đoạn code đã sửa phù hợp vào <br>
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/72d8ddab-d20c-4cff-affe-ca6bc193f9f2" />


## BƯỚC 5. CẤU HÌNH NODE WORDPRESS ĐỂ TẠO BÀI ĐĂNG
### Cấu hình Node trên n8n
- Nhấn dấu + sau node Code JavaScript, tìm kiếm node WordPress -> Chọn hành động Create a Post.
### Set up Credential
- Đăng nhập vào trang quản trị wordpress (qua subdomain 1: `wordpress.tranthithuha.id.vn`)
- Vào mục tài khoản (đã tạo lúc setup wordpress) => Hồ sơ => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential <br>
<img width="1919" height="974" alt="image" src="https://github.com/user-attachments/assets/3186a383-7820-428b-a1b3-98e45f2cd990" /> <br>

<img width="1919" height="986" alt="image" src="https://github.com/user-attachments/assets/a09eb8bb-974a-4a44-aba2-f7aeab3113ba" />


### Cấu hình tham số node
- Wordpress URL: điền giá trị https://sub-domain1/ (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n) 
Ignore SSL Issues (Insecure): TURN ON <br>
<img width="1919" height="967" alt="image" src="https://github.com/user-attachments/assets/99999d04-c386-402c-ae0d-d12186849a82" /> <br>

Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content <br>
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/273d56b3-bb60-40e0-922b-58d3d9227453" /> <br>

Kết quả sau khi cấu hình xong chuỗi flow, được một bài đăng trên wordpress:
<img width="1919" height="980" alt="image" src="https://github.com/user-attachments/assets/7425aed7-d4c4-4ecc-bacd-2f64e1e7a0eb" />



## PUBLISH FLOW
PUBLISH flow (góc trên phải). Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger <br>
<img width="1919" height="928" alt="image" src="https://github.com/user-attachments/assets/d7f2c4fa-6075-4fd8-8fae-aa32333303bf" />

# DEMO KẾT QUẢ
Sau khi đã publish flow thì có thể viết bất cứ bài viết gì chỉ cần chat với bot trên telegram chủ đề muốn viết sau đó bài viết sẽ được AI tự động viết và đăng lên wordpress
## Chat với bot trên telegram bằng điện thoại hoặc máy tính nội dung bài đăng bất kỳ 
<img width="1920" height="1080" alt="Screenshot 2026-05-25 141620" src="https://github.com/user-attachments/assets/743bb1e9-a86b-47cf-839f-afbd31c0b57a" /> <br>

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d3fd8403-0d1c-4b04-8e04-58c914c74b5b" /> <br>

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0a3e90b6-d22e-48c4-98a1-669df8fd7ac6" />

---

# VII. NHẬN XÉT VÀ KẾT QUẢ ĐẠT ĐƯỢC
Bài tập đã triển khai thành công mô hình hệ thống tự động hóa đăng bài viết bằng cách kết hợp nhiều công nghệ mã nguồn mở hiện đại gồm Docker, WordPress, MariaDB, phpMyAdmin, Cloudflare Tunnel và n8n.

Thông qua bài tập này, em đã hiểu rõ cách xây dựng và quản lý hệ thống multi-service bằng Docker Compose trên môi trường Ubuntu. Việc cấu hình các container hoạt động đồng thời giúp em nắm được cơ chế networking nội bộ giữa các service, cách truyền biến môi trường, quản lý volume cũng như kiểm tra trạng thái hoạt động của container thông qua các lệnh Docker.

Ngoài ra, em đã thực hiện thành công việc public các dịch vụ nội bộ ra Internet bằng Cloudflare Tunnel mà không cần mở port trực tiếp trên modem/router. Điều này giúp hệ thống có thể truy cập từ bên ngoài thông qua các sub-domain riêng biệt cho WordPress, phpMyAdmin và n8n, đồng thời tăng tính bảo mật và thuận tiện trong quá trình sử dụng.

Trong quá trình thực hiện, em cũng đã cài đặt và cấu hình WordPress hoàn chỉnh, kết nối thành công với cơ sở dữ liệu MariaDB. Qua phpMyAdmin có thể quan sát được sự thay đổi của cơ sở dữ liệu trước và sau khi WordPress khởi tạo, từ đó hiểu rõ hơn về cấu trúc dữ liệu và cách WordPress tự động tạo các bảng phục vụ cho website.

Phần quan trọng nhất của bài tập là xây dựng workflow tự động trên n8n. Em đã cấu hình thành công:

- Telegram Trigger để nhận nội dung tin nhắn từ bot Telegram.
- Google Gemini AI để xử lý Prompt và sinh nội dung bài viết ở định dạng HTML + CSS.
- JavaScript Code Node để tách và xử lý dữ liệu JSON trả về từ AI.
- WordPress Node để tự động đăng bài viết lên website WordPress.

Kết quả cuối cùng đạt được là:

- Người dùng chỉ cần nhắn tin cho Telegram bot bằng điện thoại.
- Nội dung tin nhắn sẽ tự động được gửi tới AI Gemini để sinh bài viết.
-n8n tự động xử lý dữ liệu và đăng bài viết mới lên WordPress ở trạng thái Publish.
- Sau khi F5 website WordPress, bài viết mới xuất hiện hoàn toàn tự động mà không cần thao tác thủ công.

Bài tập giúp em tiếp cận thực tế với mô hình tự động hóa quy trình (Workflow Automation), tích hợp AI vào hệ thống quản trị nội dung, đồng thời nâng cao kỹ năng về:

- Docker và Docker Compose
- Linux Ubuntu
- WordPress CMS
- Cloudflare Tunnel
- API và Webhook
- n8n Automation
- Tích hợp Telegram Bot
- AI Gemini API
- Xử lý dữ liệu JSON bằng JavaScript

Qua bài tập này, em nhận thấy việc kết hợp AI với các công cụ automation có thể giúp tối ưu hóa quy trình tạo nội dung và quản trị website một cách hiệu quả, tiết kiệm thời gian và giảm thao tác thủ công. Đây là một hướng ứng dụng rất tiềm năng trong thực tế đối với các hệ thống quản trị nội dung, marketing tự động và xây dựng các nền tảng AI Automation hiện đại.
















