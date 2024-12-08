# Một số khái niệm

## 1. Docker là gì?

Có thể hiểu docker là một phần mềm ảo hóa, làm đơn giản việc thiết lập môi trường phát triển ứng dụng.

### Chức năng chính
1. Build và quản lý các **Docker Image**
2. Triển khai và quản lý các **Docker Container**

---

### Lợi ích của docker
1. Tạo môi trường với các service cụ thể được cài đặt sẵn (VD setup LAMP/LEMP Stack sẽ dễ dàng hơn so với cài đặt thủ công)
2. Docker tạo ra một môi trường ảo hóa (máy ảo) độc lập so với môi trường cài đặt docker
3. Môi trường docker bất biến, có thể được cài đặt trên máy khác mà không lo sợ bị khác biệt về môi trường cài đặt. Việc này tiện lợi khi thay đổi hệ thống để cài đặt ứng dụng.
4. Rõ hơn về việc một project cần setup những gì để chạy ứng dụng, kể cả khi đã rất lâu không đụng vô
5. Do mỗi project được ảo hóa riêng biệt, nên không lo bị xung đột
6. Tự động khởi động lại trong trường hợp lỗi

## 2. Docker Image

Là một thành phần của Docker, các image như một bản đóng gói của một ứng dụng được xây dựng dựa trên một môi trường cụ thể.

> Image `ubuntu/apache2` là một image được xây dựng trên môi trường `ubuntu`, và được cài đặt sẵn server hosting là `apache`.

Để build một image cần định nghĩa `Dockerfile`. [Chi tiết về Dockerfile](./2__cau-truc-dockerfile.md);

## 3. Docker Container
Nếu **Docker Image** là các bản đóng gói, thì để sử dụng các bản đóng gói này, cần phải có môi trường ảo hóa để chạy. **Docker Container** là các instance được khởi tạo để chạy các **Docker Image**. Các container này sau đó được port ra các cổng và ta có thể truy cập các ứng dụng của image tương ứng.

Có thể tồn tại nhiều **container** của cùng một **image**

## 4. Kubernetes (K8S)
Chưa biết dùng :D