## PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ (N8N TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS)
### HỌ TÊN: NGUYỄN ĐỨC DƯƠNG
### LỚP: K58KTP
### MSSV: K225480106093

---

## 1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 FILE DOCKER-COMPOSE.YML CHỨA:

- Mariadb: sử dụng image: mariadb:latest để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)

- Phpmyadmin: sử dụng image: phpmyadmin:latest để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1

- WordPress: sử dụng image: wordpress:latest, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường: WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)

- Cloudflared: sử dụng image: cloudflare/cloudflared:latest , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose

- N8n : sử dụng image: n8nio/n8n:latest, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )

### BƯỚC 1: CẬP NHẬT DOCKER COMPOSE & KHỞI CHẠY 5 SERVICE

- Vào File `docker-compose.yml` và bổ sung thêm 2 serice sau:

**a) Phpmyadmin**

<img width="1236" height="728" alt="{676DA8FC-76AC-4CC5-9E8E-41A65812BE20}" src="https://github.com/user-attachments/assets/9dd84e8b-d3d1-400c-81cd-588cb6088422" /></p>

**b) n8n**

<img width="1351" height="757" alt="{BECCEA25-9973-4459-845F-B15FC76CFBE0}" src="https://github.com/user-attachments/assets/25bd8776-7899-4230-9d3f-28cd30922a37" /></p>

**c) Bổ sung Cloudfared**

- Truy cập cloudfared zero trust: *Đăng nhập tài khoản* -> *zero trust* -> *networks* -> *connectors*

- Tiến hành tạo tunnel: Chọn loại Cloudflared (mặc định) -> Bấm Next.

<img width="1543" height="728" alt="{ECE3380D-97A5-4215-94C1-763D67154470}" src="https://github.com/user-attachments/assets/e5debe09-1b62-421a-9cc7-287425328c4a" /></p>

- Nhập tên cho dễ nhớ ở ô *Tunnel name* -> Bấm *Save tunnel*

<img width="1920" height="1020" alt="{D17838EE-12D2-4D7D-83AB-322481C8331A}" src="https://github.com/user-attachments/assets/b36a643f-0e4a-4b6a-b1b4-05d55603f816" /></p>

- Chọn hệ điều hành Docker -> Copy token

<img width="1399" height="699" alt="{6A093E63-06AE-4F8A-B634-E131E9AC043C}" src="https://github.com/user-attachments/assets/26dc260d-df15-425f-9b5c-86a8614ab35c" /></p>

👉 Kết quả (full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose)

<img width="1332" height="732" alt="{A5BD46E1-806E-41BB-B49A-C08AF77552D4}" src="https://github.com/user-attachments/assets/966c1256-154d-4446-b1bd-48d784be0bd8" /></p>

---

### BƯỚC 2: TRIỂN KHAI HỆ THỐNG

- Khởi chạy các container `docker compose up -d`

<img width="1788" height="1054" alt="{54A2CEC6-2802-437D-A2D0-80F218CF495B}" src="https://github.com/user-attachments/assets/fbdb3dd8-ea64-48d2-b088-c290342de6b3" /></p>

- Kiểm tra trạng thái hệ thống `docker compose ps`

<img width="1737" height="1055" alt="{52475A5A-E254-439C-980D-3F2BC8460ED4}" src="https://github.com/user-attachments/assets/47fc422f-e221-424c-8691-2a3a0de81f54" /></p>

---

### BƯỚC 3: CẤU HÌNH CLOUDFLARE TUNNEL (ADD ROUTER)

- Truy cập vào trang Cloudflare Zero Trust Dashboard -> Access -> Tunnels, chọn Tunnel của và thêm 3 Public Hostname tương ứng với 3 sub-domain:

**a) Cấu hình cho WordPress (Trang web chính)**

- Subdomain: Điền wp

- Domain: Nhấp vào và chọn tên miền của bạn (ducduong.id.vn)

- Path: Để trống.

- Service - Type: Chọn HTTP

- Service - URL: Điền chính xác: wordpress:80

<img width="1918" height="1025" alt="{6ADD232D-DF2B-43A7-A4F7-F4F8D14E0CF7}" src="https://github.com/user-attachments/assets/f0146ec5-98ba-4800-912e-c180ac2b001d" /></p>

**b) Cấu hình cho phpMyAdmin**

- Subdomain: Điền pma

- Domain: Chọn tên miền của bạn (ducduong.id.vn)

- Path: Để trống.

- Service - Type: Chọn HTTP

- Service - URL: Điền chính xác: phpmyadmin:80

<img width="1920" height="1023" alt="{060A52D8-8C95-4161-B8B0-12B6BDC6D1C7}" src="https://github.com/user-attachments/assets/388cb789-d4ab-4026-997a-1617c9be3441" /></p>

**c) Cấu hình cho n8n (Tự động hóa)**

- Subdomain: Điền n8n (Bắt buộc điền đúng từ này vì nó phải khớp với biến WEBHOOK_URL trong file yml lúc nãy)

- Domain: Chọn domain của bạn (ducduong.id.vn)

- Path: Để trống.

- Service - Type: Chọn HTTP

- Service - URL: Điền tên service của n8n kèm cổng mặc định của nó: n8n:5678

<img width="1920" height="1024" alt="{E16A9D24-8254-4AC5-9F82-64D43FC4F564}" src="https://github.com/user-attachments/assets/be3bae34-b9cb-41a7-b757-d24bf1691d24" /></p>

👉 Kết quả

<img width="1404" height="736" alt="image" src="https://github.com/user-attachments/assets/74ba796f-4a5f-42ee-8dd5-4a8f65dec620" /></p>

---

### BƯỚC 4: KIỂM TRA DATABASE & CÀI ĐẶT WORDPRESS

**a) Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!**

- Truy cập `http://pma.ducduong.id.vn`

- Bước kiểm tra đầu tiên: Truy cập sub-domain2 (phpMyAdmin). Đăng nhập bằng tài khoản wp_user và mật khẩu đã đặt. Sẽ thấy database wordpress_db hoàn toàn trống rỗng (chưa có bảng nào).

<img width="1920" height="1024" alt="{2C643BAB-F8F3-4DCA-8D29-EA0A78943628}" src="https://github.com/user-attachments/assets/c51672ce-ab71-4ec4-8a3e-885677cf1b1e" /></p>

<img width="1920" height="1025" alt="{699BE811-B752-4D6A-A2D1-B961E760E4C1}" src="https://github.com/user-attachments/assets/113c1d08-01d7-4c83-8285-08f0087e2066" /></p>

**b) Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)**

- Truy cập `wp.ducduong.id.vn`

<img width="1920" height="1025" alt="{47BA03D5-B00A-438E-B10A-B918931638D3}" src="https://github.com/user-attachments/assets/07f22b63-ed31-4ff9-afa4-ce468e78835f" /></p>

<img width="1920" height="1028" alt="{F7ED01D2-D026-461F-AC68-BC52DEEFE07A}" src="https://github.com/user-attachments/assets/a8dd7600-5693-4f24-a630-810ce6cb73e3" /></p>

👉 Sau khi cài đặt xong sẽ có thông báo 'Thành công' -> Tiến hành đăng nhập

<img width="1920" height="1026" alt="image" src="https://github.com/user-attachments/assets/c91735b6-ecee-4748-833a-c463616d3181" /></p>

<img width="1920" height="1031" alt="{02456A23-9A57-49EE-AB43-5C776078859A}" src="https://github.com/user-attachments/assets/74d5ac60-1429-43a2-a79d-133204bf5106" /></p>

**c) Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp** 

<img width="1399" height="731" alt="{46305C7C-B22E-4E9B-9574-02E846925D2C}" src="https://github.com/user-attachments/assets/fa8fd7b1-2d41-4c8c-9dc1-2fa587ac7e3e" />

**c) Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ...**

- Ở trang chủ Wordpress -> *Post* -> *Add New*

<img width="1403" height="749" alt="{EB5DF9DB-3BC1-44FC-BF18-2CE1DDB48D5A}" src="https://github.com/user-attachments/assets/0cca9a53-dbd0-40cb-b411-5013f3ca02c2" />

















