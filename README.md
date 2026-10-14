# IT209 - Session 11 - Bài tập 4: Khai báo GitHub Secrets và cấu hình Đăng nhập Container Registry

- **Sinh viên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Lớp:** IT209
- **Học phần:** IT209
- **Đường dẫn nộp bài:** `homework/session_11/ex4/`
- **Trạng thái:** Hoàn thành (100%)

---

## 1. Danh mục tệp tin trong thư mục bài tập

| Tên tệp tin | Mô tả chức năng |
| :--- | :--- |
| [.github/workflows/ci.yml](./.github/workflows/ci.yml) | **Tệp kịch bản GitHub Actions CI chính thức:** Đã cấu hình bước xác thực đăng nhập vào GitHub Container Registry bằng `docker/login-action@v3` |
| [screenshots/github-repository-secrets.png](./screenshots/github-repository-secrets.png) | **Ảnh minh chứng nộp bài:** Mục Repository Secrets hiển thị biến bí mật `REGISTRY_TOKEN` đã khởi tạo |
| [screenshots/github-actions-login-succeeded.png](./screenshots/github-actions-login-succeeded.png) | **Ảnh minh chứng nộp bài:** Log chi tiết của bước Đăng nhập Registry trên tab Actions hiển thị `Login Succeeded` |
| [IT209_Session11_BT4_PhamThanhPhong_N24DTCN120.md](./IT209_Session11_BT4_PhamThanhPhong_N24DTCN120.md) | Báo cáo kỹ thuật chi tiết, phân tích cơ chế mã hóa bất đối xứng Secrets, sơ đồ tuần tự và hướng dẫn từng bước |
| [build.gradle](./build.gradle) & [settings.gradle](./settings.gradle) | Tệp cấu hình Spring Boot 3.2.5, Gradle 8.7 và Java 17 |
| [gradlew](./gradlew) & [gradle/](./gradle/) | Bộ script Gradle Wrapper |
| [src/](./src/) | Mã nguồn ứng dụng và bộ unit tests JUnit 5 |

---

## 2. Kịch bản `.github/workflows/ci.yml` (Bổ sung `docker/login-action@v3`)

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests, Build Artifact & Login Registry
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar

      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

---

## 3. Ảnh minh chứng kết quả thực thi

### 3.1. Cấu hình biến bí mật `REGISTRY_TOKEN` trong Repository Secrets
![GitHub Repository Secrets](./screenshots/github-repository-secrets.png)

### 3.2. Log chi tiết bước Đăng nhập Registry hiển thị `Login Succeeded`
![GitHub Actions Login Succeeded](./screenshots/github-actions-login-succeeded.png)

---

## 4. Hướng dẫn thiết lập nhanh
1. Khởi tạo **Personal Access Token (classic)** với quyền hạn `write:packages` (tự động đính kèm `read:packages`).
2. Vào repository GitHub: **Settings $\rightarrow$ Secrets and variables $\rightarrow$ Actions** $\rightarrow$ bấm **New repository secret**.
3. Điền tên `REGISTRY_TOKEN` và dán mã token vừa tạo.
4. Đẩy mã nguồn lên nhánh `main` để kích hoạt workflow tự động đăng nhập.
