# VnGrocery

**Nền tảng minh bạch nguồn gốc sản phẩm áp dụng công nghệ Blockchain**

VnGrocery là nền tảng truy xuất và minh bạch nguồn gốc sản phẩm, ứng dụng **Blockchain** nhằm lưu trữ, xác thực và chia sẻ thông tin về quá trình hình thành, vận chuyển và phân phối sản phẩm.

## Yêu cầu hệ thống

Trước khi cài đặt, đảm bảo môi trường đã có:

* Git
* Repo tool
* Android SDK / JDK nếu xây dựng thành phần Android
* Docker nếu sử dụng môi trường triển khai bằng container
* Kết nối Internet để đồng bộ mã nguồn và tải các dependency

### Cài đặt Repo

```bash
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
export PATH="$HOME/bin:$PATH"
```

Kiểm tra:

```bash
repo --version
```

## Đồng bộ mã nguồn

Tạo thư mục làm việc:

```bash
mkdir VnGrocery
cd VnGrocery
```

Khởi tạo Repo Manifest:

```bash
repo init -u https://github.com/VnGrocery/manifest.git -b main
```

Sau đó đồng bộ toàn bộ source:

```bash
repo sync -j$(nproc)
```

Trên macOS có thể sử dụng:

```bash
repo sync -j$(sysctl -n hw.ncpu)
```

Nếu muốn đồng bộ lại và tự động xử lý các thay đổi:

```bash
repo sync -c --force-sync
```

Sau khi hoàn tất, cấu trúc source sẽ được quản lý bởi Repo Manifest và các project được khai báo trong manifest.

## Cài đặt

### 1. Clone manifest

```bash
mkdir -p ~/VnGrocery
cd ~/VnGrocery

repo init -u https://github.com/VnGrocery/manifest.git -b main
repo sync
```

### 2. Chạy backend

Toàn bộ hạ tầng (API, MongoDB, Vault, IPFS, Besu) chạy bằng một script trong
`server/`:

```bash
cd server
cp .env.example .env      # điền JWT_SECRET và OPENAI_API_KEY
./scripts/vng up
./scripts/vng health
```

API nằm ở cổng `5050` của máy host. Xem chi tiết trong
[server/docs/HUONG-DAN-TRIEN-KHAI.md](https://github.com/VnGrocery/Server/blob/main/docs/HUONG-DAN-TRIEN-KHAI.md).

Đổ dữ liệu mẫu để có gì mà xem:

```bash
./scripts/vng seed            # cửa hàng, sản phẩm, cam kết, đánh giá
./scripts/vng seed-history    # lịch sử giá 30 ngày cho mọi sản phẩm
```

### 3. Chạy app

```bash
cd mobile
flutter pub get
flutter run
```

Mặc định app trỏ tới `http://10.0.2.2:5050` (địa chỉ của máy host nhìn từ
Android emulator). Máy thật thì trỏ vào IP LAN:

```bash
flutter run --dart-define=API_BASE_URL=http://192.168.1.10:5050
```

### 4. Chạy trang quản trị

```bash
cd admin
npm install
npm run dev
```

## Kiến trúc

VnGrocery được xây dựng theo mô hình gồm các thành phần chính:

```text
VnGrocery/              # thư mục repo sync tạo ra
├── server/             # Go API, worker, hợp đồng IntegrityRegistry
├── mobile/             # ứng dụng Flutter cho người mua và người bán
├── admin/              # trang quản trị React + Vite
└── .repo/              # dữ liệu nội bộ của repo tool
```

Ba project được khai báo trong [`default.xml`](default.xml). Có thể lấy riêng
từng phần bằng group:

```bash
repo init -u https://github.com/VnGrocery/manifest.git -b main -g backend
```

Quy trình truy xuất nguồn gốc:

```text
Nhà sản xuất
      │
      ▼
  Sản phẩm
      │
      ▼
 Backend / API
      │
      ├──────────► Database
      │
      ▼
  Blockchain
      │
      ▼
 Người tiêu dùng
```

Blockchain được sử dụng để tăng tính minh bạch và khả năng kiểm chứng dữ liệu trong quá trình truy xuất nguồn gốc sản phẩm.

## Đóng góp

Repo này chỉ chứa manifest. Thay đổi mã nguồn gửi vào đúng project con
(`Server`, `Mobile`, `website`); chỉ sửa `default.xml` khi thêm/bớt project
hoặc đổi nhánh mặc định.

1. Fork repository.
2. Tạo branch mới:

```bash
git checkout -b feature/<ten-tinh-nang>
```

3. Thực hiện thay đổi.
4. Commit:

```bash
git add .
git commit -m "feat: <mo-ta-thay-doi>"
```

5. Push branch:

```bash
git push origin feature/<ten-tinh-nang>
```

6. Tạo Pull Request.

## Giấy phép

Repo manifest này phát hành theo giấy phép MIT — xem [LICENSE](LICENSE).

Mỗi project con giữ giấy phép riêng của nó; đọc file `LICENSE` trong từng
repository sau khi `repo sync`.

