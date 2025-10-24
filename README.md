# Network Programming Blog

Blog cá nhân về lập trình mạng với Java & JavaScript, được xây dựng bằng Hugo.

## Yêu cầu hệ thống

- Hugo (phiên bản 0.100.0 trở lên)
- Git
- Trình duyệt web hiện đại

## Cài đặt

### 1. Clone repository

```bash
git clone <repository-url>
cd network-programming-blog
```

### 2. Cài đặt Hugo

#### Windows:
```bash
# Sử dụng Chocolatey
choco install hugo

# Hoặc tải từ https://github.com/gohugoio/hugo/releases
```

#### macOS:
```bash
# Sử dụng Homebrew
brew install hugo
```

#### Linux:
```bash
# Ubuntu/Debian
sudo apt-get install hugo

# CentOS/RHEL
sudo yum install hugo
```

### 3. Chạy blog

```bash
# Chạy server development
hugo server -D

# Hoặc chạy với draft content
hugo server --buildDrafts
```

Blog sẽ có sẵn tại: http://localhost:1313

## Cấu trúc dự án

```
network-programming-blog/
├── content/
│   └── posts/                 # Các bài viết blog
├── themes/
│   └── minimal-theme/         # Theme tùy chỉnh
│       ├── layouts/          # Templates
│       ├── static/           # CSS, JS, images
│       └── partials/          # Components
├── hugo.toml                 # Cấu hình Hugo
└── README.md
```

## Tính năng

- ✅ **Trang chủ** với giới thiệu cá nhân
- ✅ **Trang Blog** với danh sách bài viết
- ✅ **9 bài viết** về Java & JavaScript
- ✅ **Thiết kế tối giản** và responsive
- ✅ **Navigation menu** dễ sử dụng
- ✅ **SEO friendly** với meta tags
- ✅ **Mobile responsive**

## Nội dung blog

### Bài viết về Java:
1. **Lập Trình Socket với Java - Cơ Bản**
2. **Hướng Dẫn Sử Dụng HTTP Client trong Java 11+**
3. **Lập Trình Mạng với Java NIO**
4. **Lập Trình Reactive với Spring WebFlux**
5. **Giao Tiếp Giữa Các Microservices với Java**

### Bài viết về JavaScript:
1. **Hướng Dẫn Sử Dụng Fetch API trong JavaScript**
2. **Lập Trình WebSocket với JavaScript**
3. **So Sánh Axios và Fetch API trong JavaScript**
4. **Xây Dựng REST API với Node.js và Express**

## Tùy chỉnh

### Thay đổi thông tin cá nhân

Chỉnh sửa file `hugo.toml`:

```toml
[params]
  author = "Tên của bạn"
  description = "Mô tả blog của bạn"
  keywords = ["từ khóa", "của", "bạn"]
```

### Thêm bài viết mới

```bash
# Tạo bài viết mới
hugo new posts/ten-bai-viet-moi.md
```

### Thay đổi giao diện

Chỉnh sửa các file trong `themes/minimal-theme/static/css/style.css`

## Build và Deploy

### Build cho production

```bash
# Build static site
hugo --minify

# Output sẽ được tạo trong thư mục `public/`
```

### Deploy lên GitHub Pages

```bash
# Build
hugo --minify

# Commit và push
git add .
git commit -m "Update blog"
git push origin main

# Deploy (nếu sử dụng GitHub Actions)
# File .github/workflows/deploy.yml sẽ tự động deploy
```

### Deploy lên Netlify

1. Kết nối repository với Netlify
2. Cấu hình build command: `hugo --minify`
3. Cấu hình publish directory: `public`
4. Deploy tự động khi push code

## Cấu trúc theme

### Layouts
- `baseof.html` - Template chính
- `index.html` - Trang chủ
- `list.html` - Danh sách bài viết
- `single.html` - Chi tiết bài viết

### Partials
- `header.html` - Header với navigation
- `footer.html` - Footer

### Static files
- `css/style.css` - Stylesheet chính

## Troubleshooting

### Lỗi thường gặp

1. **Hugo server không chạy được**
   ```bash
   # Kiểm tra phiên bản Hugo
   hugo version
   
   # Cài đặt lại Hugo
   # Windows: choco install hugo
   # macOS: brew install hugo
   ```

2. **Theme không load được**
   ```bash
   # Kiểm tra cấu hình trong hugo.toml
   theme = 'minimal-theme'
   ```

3. **CSS không hiển thị**
   - Kiểm tra đường dẫn trong `baseof.html`
   - Đảm bảo file CSS tồn tại trong `themes/minimal-theme/static/css/`

## Đóng góp

1. Fork repository
2. Tạo feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'Add amazing feature'`
4. Push to branch: `git push origin feature/amazing-feature`
5. Tạo Pull Request

## License

MIT License - xem file [LICENSE](LICENSE) để biết thêm chi tiết.

## Liên hệ

- Email: contact@example.com
- GitHub: [username](https://github.com/username)
- LinkedIn: [Your Name](https://linkedin.com/in/yourname)

---

**Lưu ý**: Blog này được tạo để đáp ứng yêu cầu đồ án học phần về lập trình mạng. Tất cả nội dung đều được viết bằng tiếng Việt và tập trung vào Java & JavaScript.


