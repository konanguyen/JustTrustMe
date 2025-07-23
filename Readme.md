JustTrustMe - Bypass SSL Pinning Nâng Cao
==========================================

JustTrustMe là một module Xposed/LSPosed tiên tiến giúp bypass toàn diện SSL pinning trong các ứng dụng Android. Phiên bản nâng cấp này đã được cập nhật cho **JDK 17** và **các phiên bản Android hiện đại (API 21-34)** với hỗ trợ cho các triển khai SSL pinning mới nhất.

## ✨ Tính Năng Mới (Cập Nhật 2025)

### 🔧 Hỗ Trợ Nền Tảng Hiện Đại
- **Tương thích JDK 17** cho hiệu suất tốt hơn
- **Hỗ trợ Android API 21-34** (Android 5.0 - Android 14+)
- **Cập nhật Android Gradle Plugin 8.2.2** với Gradle 8.5
- **Quản lý dependency hiện đại** (thay thế jcenter bằng mavenCentral)

### 🛡️ Khả Năng Bypass SSL Nâng Cao

#### **Bypass Network Security Config**
- `android.security.net.config.NetworkSecurityTrustManager`
- `android.security.net.config.RootTrustManager` 
- `android.security.net.config.ConfigAwareConnectionStateSSLContext`
- Bypass áp đặt Certificate Transparency (CT)

#### **Hỗ Trợ Conscrypt Hiện Đại**
- Hook nâng cao `com.android.org.conscrypt.TrustManagerImpl`
- Phương thức xác minh trust đặc biệt cho Android 14+
- Bypass `verifyChain` và `checkTrustedRecursive`

#### **Hỗ Trợ Thư Viện HTTP Client**
- **OkHttp 5.x** với hỗ trợ Kotlin coroutines
- **Retrofit 2.x** bypass xác minh SSL
- **Volley** bypass SSL HurlStack
- **Apache HttpClient 5.x** bypass xác minh hostname
- **Cronet** (Chrome Network Stack) bypass lỗi SSL

#### **Hỗ Trợ WebView Nâng Cao**
- Bypass yêu cầu chứng chỉ client WebView hiện đại
- Xử lý lỗi SSL nâng cao cho các phiên bản WebView mới hơn
- Bypass certificate transparency trong WebView

## 🚀 Build Module

### Yêu Cầu
- **Android Studio** với Android SDK
- **JDK 17** hoặc cao hơn
- **Android SDK API 34**

### Lệnh Build

**Windows:**
```batch
gradlew assembleRelease
```

**Linux/macOS:**
```bash
./gradlew assembleRelease
```

File APK được build sẽ nằm tại: `app/build/outputs/apk/release/app-release.apk`

## 📋 Thư Viện & Framework Được Hỗ Trợ

### SSL Android Cốt Lõi
- `javax.net.ssl.HttpsURLConnection`
- `javax.net.ssl.SSLContext`
- `javax.net.ssl.TrustManagerFactory`
- `android.webkit.WebViewClient`

### Thư Viện HTTP Bên Thứ Ba
- **OkHttp**: 2.5.x, 3.x, 4.x, 5.x
- **Retrofit**: 2.x
- **Apache HttpClient**: Legacy và 5.x
- **Volley**: Thư viện HTTP của Google
- **XUtils**: Tiện ích HTTP của Trung Quốc
- **HttpClientAndroidLib**: Apache client thay thế
- **Cronet**: Network stack của Chrome

### Bảo Mật Android Hiện Đại
- Android Network Security Config
- Certificate Transparency
- Conscrypt SSL provider
- Khả năng bypass App Attestation

## 🔍 Chi Tiết Triển Khai Kỹ Thuật

### Các Hook Bypass SSL Cốt Lõi

#### 1. Bypass Certificate Trust Manager
- **`X509TrustManagerExtensions.checkServerTrusted()`** - Trả về chứng chỉ mà không xác thực
- **`SSLContext.init()`** - Thay thế TrustManager bằng implementation cho phép tất cả
- **`TrustManagerFactory.getTrustManagers()`** - Trả về custom trust manager

#### 2. Android Network Security Config
- **`NetworkSecurityTrustManager.checkPins()`** - Vô hiệu hóa certificate pinning
- **`RootTrustManager.checkServerTrusted()`** - Bypass xác thực root certificate
- **`CertificateTransparencyPolicy.shouldEnforceCertificateTransparency()`** - Vô hiệu hóa áp đặt CT

#### 3. Thư Viện HTTP Client
- **DefaultHttpClient**: Thay thế connection manager bằng SSL factory cho phép tất cả
- **OkHttp CertificatePinner**: Vô hiệu hóa kiểm tra certificate pinning trên tất cả phiên bản
- **Retrofit**: Bypass xác minh SSL trong thực thi OkHttpCall
- **Volley**: Inject SSL factory cho phép tất cả vào kết nối HurlStack

#### 4. Xử Lý SSL WebView
- **`WebViewClient.onReceivedSslError()`** - Tự động gọi `proceed()` khi có lỗi SSL
- **`WebViewClient.onReceivedClientCertRequest()`** - Bỏ qua yêu cầu chứng chỉ client

### Tính Năng Nâng Cao Cho Android Hiện Đại

#### Hook Conscrypt Provider (Android 14+)
```java
// Bypass xác minh trust hiện đại
findAndHookMethod("com.android.org.conscrypt.TrustManagerImpl", 
    "verifyChain", X509Certificate[].class, String.class, String.class, boolean.class);

// Bypass kiểm tra trust đệ quy
findAndHookMethod("com.android.org.conscrypt.TrustManagerImpl",
    "checkTrustedRecursive", X509Certificate[].class, byte[].class, byte[].class);
```

#### Bypass Certificate Transparency
```java
// Vô hiệu hóa áp đặt CT
findAndHookMethod("android.security.net.config.CertificateTransparencyPolicy",
    "shouldEnforceCertificateTransparency", String.class);
```

#### Xử Lý Lỗi SSL Cronet
```java
// Bypass lỗi SSL trong network stack của Chrome
findAndHookMethod("org.chromium.net.impl.CronetUrlRequest", 
    "onReceivedError", int.class, String.class);
```

## ⚠️ Tác Động Bảo Mật

Module này hoàn toàn vô hiệu hóa xác thực chứng chỉ SSL/TLS, khiến ứng dụng dễ bị tấn công:
- **Tấn công Man-in-the-middle**
- **Giả mạo chứng chỉ**  
- **Tấn công hạ cấp**
- **Nghe lén giao tiếp mã hóa**

**Sử dụng có trách nhiệm và chỉ cho mục đích kiểm tra bảo mật hợp pháp.**

## 🛡️ Phòng Vệ Chống Bypass SSL Pinning

### Bảo Vệ Cấp Ứng Dụng

#### 1. Phát Hiện Runtime
- Kiểm tra sự hiện diện của framework Xposed/LSPosed
- Xác minh tính toàn vẹn method bằng reflection
- Triển khai kỹ thuật anti-hooking

#### 2. Triển Khai Cấp Native
- Chuyển xác thực SSL sang native code (JNI/C++)
- Sử dụng Android NDK cho certificate pinning
- Triển khai xác minh SSL tùy chỉnh trong thư viện native

#### 3. Xác Thực Phía Server
- Triển khai xác thực mutual TLS (mTLS)
- Sử dụng Google Play Integrity API / SafetyNet
- Xác thực chứng chỉ phía server
- Cơ chế challenge-response

#### 4. Obfuscation Nâng Cao
- Obfuscation code và anti-debugging
- Control flow flattening
- Mã hóa string
- Cơ chế anti-tampering

### Bảo Vệ Cấp Network
- Giám sát Certificate Transparency
- Header HPKP (HTTP Public Key Pinning)
- HSTS (HTTP Strict Transport Security)
- Logic xác thực chứng chỉ tùy chỉnh

## 📱 Môi Trường Đã Kiểm Tra

- **Phiên Bản Android**: 5.0 - 14+ (API 21-34)
- **Xposed Framework**: LSPosed (khuyến nghị), EdXposed
- **Kiến Trúc**: ARM64, ARM, x86, x86_64
- **Phương Thức Root**: Magisk, SuperSU (legacy)

## 🔧 Cài Đặt

1. Cài đặt LSPosed hoặc framework Xposed tương thích
2. Cài đặt APK JustTrustMe
3. Kích hoạt module trong LSPosed Manager
4. Khởi động lại ứng dụng mục tiêu hoặc reboot thiết bị
5. Theo dõi log để xác nhận bypass

## 📜 Nhật Ký Thay Đổi

### Phiên Bản 3.0 (2025)
- **Hỗ trợ JDK 17** với hệ thống build hiện đại
- **Tương thích Android 14+** với bypass bảo mật mới nhất
- **Hỗ trợ OkHttp 5.x nâng cao** bao gồm Kotlin coroutines
- **Bypass Certificate Transparency** cho Android hiện đại
- **Hỗ trợ Cronet** cho network stack dựa trên Chrome
- **Cải thiện xử lý WebView** cho triển khai hiện đại
- **Xử lý lỗi và logging tốt hơn**

### Các Phiên Bản Trước
- Hỗ trợ Android legacy (API 17-30)
- Hỗ trợ OkHttp 2.x-4.x cơ bản
- Chức năng bypass SSL pinning tiêu chuẩn

---

**Tuyên Bố Từ Chối Trách Nhiệm**: Công cụ này chỉ dành cho nghiên cứu bảo mật và kiểm tra thâm nhập. Sử dụng có trách nhiệm và tuân thủ luật pháp và quy định hiện hành.
