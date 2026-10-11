# Bài 4: Khai báo GitHub Secrets và Cấu hình Đăng nhập Container Registry

## Mục tiêu

- Tạo Repository Secret `REGISTRY_TOKEN` trên GitHub để lưu trữ Personal Access Token an toàn.
- Cấu hình bước xác thực tự động vào **GitHub Container Registry (GHCR)** trong workflow CI.

---

## Cấu trúc thư mục

```
ex4/
├── .github/
│   └── workflows/
│       └── ci.yml              ← File workflow CI chính
├── gradle/wrapper/
│   └── gradle-wrapper.properties
├── src/
│   ├── main/java/com/example/demo/
│   │   └── DemoApplication.java
│   ├── main/resources/
│   │   └── application.properties
│   └── test/java/com/example/demo/
│       └── DemoApplicationTests.java
├── .dockerignore
├── build.gradle
├── Dockerfile
├── gradlew
├── gradlew.bat
└── settings.gradle
```

---

## Hướng dẫn thực hiện

### Bước 1 — Tạo Personal Access Token (classic)

1. Vào GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**.
2. Nhấn **Generate new token (classic)**.
3. Đặt tên token, chọn quyền **`write:packages`** (bao gồm `read:packages`).
4. Nhấn **Generate token** và **sao chép token** (chỉ hiển thị một lần).

### Bước 2 — Tạo Repository Secret

1. Vào Repository GitHub → **Settings** → **Secrets and variables** → **Actions**.
2. Nhấn **New repository secret**.
3. Điền:
   - **Name:** `REGISTRY_TOKEN`
   - **Secret:** dán token vừa sao chép ở Bước 1.
4. Nhấn **Add secret**.

> **Ảnh chụp màn hình cần nộp:** Màn hình Repository Secrets hiển thị biến `REGISTRY_TOKEN` đã khởi tạo.

### Bước 3 — Đẩy code lên GitHub

```bash
git add .
git commit -m "feat: add CI workflow with GHCR login (ex4)"
git push origin main
```

### Bước 4 — Kiểm tra kết quả

1. Vào tab **Actions** của Repository.
2. Mở workflow run mới nhất.
3. Nhấn vào job **"Đăng nhập vào GitHub Container Registry"**.
4. Mở bước **"Đăng nhập vào GitHub Container Registry"** và xác nhận log hiển thị:

```
Login Succeeded
```

---

## Nội dung file ci.yml

```yaml
name: CI - Bài 4 Đăng nhập GitHub Container Registry

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  login-registry:
    name: Đăng nhập vào GitHub Container Registry
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v4

      - name: Thiết lập Java 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cấp quyền thực thi cho gradlew
        run: chmod +x gradlew

      - name: Chạy Gradle Test
        run: ./gradlew test

      - name: Đóng gói ứng dụng JAR
        run: ./gradlew bootJar

      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

---

## Giải thích các thành phần chính

| Thành phần | Ý nghĩa |
|---|---|
| `secrets.REGISTRY_TOKEN` | Token được lưu an toàn trong Repository Secrets, không lộ trong log |
| `github.actor` | Username GitHub của người trigger workflow |
| `registry: ghcr.io` | Đăng nhập vào GitHub Container Registry |
| `docker/login-action@v3` | Action chính thức của Docker để xác thực registry |

---

## Sản phẩm nộp bài

- [x] File `.github/workflows/ci.yml` có bước `docker/login-action@v3`
- [ ] Ảnh chụp màn hình **Repository Secrets** hiển thị `REGISTRY_TOKEN` đã khởi tạo
- [ ] Ảnh chụp màn hình log Actions hiển thị **"Login Succeeded"**
