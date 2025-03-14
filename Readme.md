JustTrustMe
===========

Module trên là một ví dụ về việc sử dụng Xposed Framework để hook và thay đổi hành vi của một số lớp và phương thức liên quan đến SSL/TLS trong ứng dụng Android. Mục đích của việc này là bypass (vượt qua) SSL Pinning, một kỹ thuật bảo mật nhằm xác thực chứng chỉ SSL của máy chủ để đảm bảo rằng ứng dụng chỉ kết nối với các máy chủ đáng tin cậy.
Để build module này ta cần Android Studio đã được cài đặt sẵn trên máy cùng với JDK 17, sau đó build bằng lệnh
Đối với Windows:
```
gradlew assembleRelease
```
Đối với Linux/MacOS:
```
./gradlew assembleRelease
```
Khi build thành công ta tiến hành cài đặt file apk ở đường dẫn app/build/outputs/apk/release/app-release-unsigned.apk

Dưới đây là một số phần quan trọng trong code giải thích cách nó bypass SSL Pinning:

---

## 1. Các Hook Cơ Bản Trong Code

### 1.1. Hook Các Phương Thức Kiểm Tra Chứng Chỉ SSL

- **`findAndHookMethod(X509TrustManagerExtensions.class, "checkServerTrusted", ...)`**  
  Phương thức `checkServerTrusted` của lớp `X509TrustManagerExtensions` được hook để trả về danh sách các chứng chỉ mà không thực hiện bất kỳ kiểm tra nào.

- **`findAndHookMethod("android.security.net.config.NetworkSecurityTrustManager", ...)`**  
  Phương thức `checkPins` của lớp `NetworkSecurityTrustManager` được hook để không thực hiện bất kỳ hành động nào (DO_NOTHING).

### 1.2. Hook Các Constructor và Phương Thức của `DefaultHttpClient` và `SSLSocketFactory`

- **`findAndHookConstructor(DefaultHttpClient.class, ...)`**  
  Các constructor của `DefaultHttpClient` được hook để thay thế `ClientConnectionManager` bằng một phiên bản tùy chỉnh (`getSCCM()` hoặc `getCCM()`) mà tin tưởng tất cả các chứng chỉ.

- **`findAndHookConstructor(SSLSocketFactory.class, ...)`**  
  Constructor của `SSLSocketFactory` được hook để thay thế `TrustManager` bằng `getTrustManager()` mà tin tưởng tất cả các chứng chỉ.

### 1.3. Hook Phương Thức `init` của `SSLContext`

- **`findAndHookMethod("javax.net.ssl.SSLContext", "init", ...)`**  
  Phương thức `init` của `SSLContext` được hook để thay thế `TrustManager` bằng một `TrustManager` tùy chỉnh (`ImSureItsLegitTrustManager`) mà tin tưởng tất cả các chứng chỉ.

### 1.4. Hook Các Lớp và Phương Thức của `HttpsURLConnection`

- **`findAndHookMethod("javax.net.ssl.HttpsURLConnection", "setDefaultHostnameVerifier", ...)`**  
  Phương thức `setDefaultHostnameVerifier` được hook để không thực hiện bất kỳ hành động nào (DO_NOTHING).

- **`findAndHookMethod("javax.net.ssl.HttpsURLConnection", "setSSLSocketFactory", ...)`**  
  Phương thức `setSSLSocketFactory` được hook để không thực hiện bất kỳ hành động nào (DO_NOTHING).

### 1.5. Hook Phương Thức `onReceivedSslError` của `WebViewClient`

- **`findAndHookMethod("android.webkit.WebViewClient", "onReceivedSslError", ...)`**  
  Phương thức `onReceivedSslError` của `WebViewClient` được hook để luôn gọi `proceed()` trên `SslErrorHandler`, bỏ qua bất kỳ lỗi SSL nào.

### 1.6. Hook Các Lớp và Phương Thức của `TrustManagerImpl` (trên thiết bị mới hơn)

- **`findAndHookMethod("com.android.org.conscrypt.TrustManagerImpl", "checkServerTrusted", ...)`**  
  Các phương thức `checkServerTrusted` của `TrustManagerImpl` được hook để trả về một danh sách trống các chứng chỉ, bỏ qua bất kỳ lỗi xác thực nào.

### 1.7. Hook Các Thư Viện Bên Thứ Ba (Như OkHttp, HttpClientAndroidLib)

- Các phương thức liên quan đến kiểm tra chứng chỉ của các thư viện như OkHttp và HttpClientAndroidLib cũng được hook để bỏ qua kiểm tra SSL Pinning.

---

## 2. Tác Động của Việc Bypass SSL Pinning

Bằng cách hook và thay đổi hành vi của các lớp và phương thức liên quan đến SSL/TLS, module này đảm bảo rằng:
- **Mọi chứng chỉ SSL đều được tin tưởng.**  
- **SSL Pinning bị bypass,** cho phép ứng dụng kết nối tới bất kỳ máy chủ nào mà không gặp lỗi xác thực chứng chỉ.

---

## 3. Cách Phòng Chống Bị Ảnh Hưởng Bởi Module Này

Để tránh bị ảnh hưởng bởi các module bypass SSL Pinning sử dụng LSPosed hoặc Xposed, bạn có thể áp dụng các biện pháp sau:

### 3.1. Phát Hiện và Chống Hook

- **Kiểm tra sự hiện diện của Xposed/LSPosed:**  
  Thực hiện kiểm tra trong runtime để phát hiện dấu hiệu của các framework hook (ví dụ: kiểm tra các file hệ thống, process hoặc dấu hiệu bất thường trong các phương thức hệ thống).

- **Kiểm tra tính toàn vẹn của code:**  
  Sử dụng checksum hoặc so sánh chữ ký số (signature) của ứng dụng để phát hiện sự thay đổi so với phiên bản gốc.

### 3.2. Thực Hiện SSL Pinning Ở Cấp Độ Native

- **Chuyển logic xác thực SSL sang mã native (JNI):**  
  Mã native thường khó bị hook hơn so với mã Java, do đó có thể bảo vệ tốt hơn quá trình xác thực chứng chỉ.

### 3.3. Tăng Cường Xác Thực Từ Phía Server

- **Sử dụng các cơ chế attestation:**  
  Áp dụng Google SafetyNet hoặc Play Integrity để xác nhận tính toàn vẹn của ứng dụng từ phía server.

- **Xác thực chứng chỉ từ phía server:**  
  Thực hiện kiểm tra chứng chỉ bổ sung trên server nhằm giảm thiểu rủi ro khi client bị tấn công.

### 3.4. Sử Dụng Kỹ Thuật Obfuscation và Anti-Debugging

- **Obfuscation:**  
  Làm rối code để gây khó khăn cho việc reverse-engineering và phân tích code.
  
- **Anti-debugging:**  
  Sử dụng các kỹ thuật phát hiện và chống debug để ngăn chặn việc hook các phương thức quan trọng.

---

## 4. Kết Luận

Mặc dù không có biện pháp nào đảm bảo tuyệt đối chống lại các kỹ thuật hook tiên tiến như LSPosed, việc kết hợp nhiều lớp bảo vệ (phát hiện hook, thực hiện SSL pinning ở cấp native, tăng cường xác thực phía server, và áp dụng obfuscation/anti-debugging) sẽ làm tăng đáng kể độ khó cho kẻ tấn công. Qua đó, bạn có thể giảm thiểu rủi ro bị bypass SSL Pinning và bảo vệ tốt hơn kết nối SSL/TLS trong ứng dụng Android.

---
