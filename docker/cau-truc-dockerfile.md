# Cấu trúc Dockerfile
[Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)

## 1. Tên file
Dockerfile có thể đặt tên khác, tuy nhiên prefer là *Dockerfile* thì khi build image không cần quan tâm về tên file.

## 2. Cấu trúc

Dưới đây là danh sách hay dùng

Key|Mô tả
-|-
ARG|Tạo biến sử dụng trong Dockerfile
FROM|Tạo image mới từ môt image gốc
MAINTAINER|Mô tả chủ sở hữu của image
LABEL|Thêm mô tả metadata cho image
WORKDIR|Thay đổi folder làm việc, giống như lệnh `cd`
COPY|Copy file, folder từ local vào image
ADD|Tương tự như copy nhưng có tác dụng cả với url
RUN|Command được chạy trong khi build image
EXPOSE|Mô tả ports và protocals mà ứng dụng sẽ lắng nghe
ENV|Thiết lập biến môi trường và giá trị
CMD|Command được chạy khi khởi chạy container

---
#### Ví dụ Dockerfile được build dựa trên image `php:8.3-fpm-alpine`

```Dockerfile
FROM php:8.3-fpm-alpine

WORKDIR /var/www/html

RUN apk add --no-cache linux-headers

RUN apk update && apk add --no-cache \
    bash \
    curl \
    git \
    icu-dev \
    libpng-dev \
    libjpeg-turbo-dev \
    freetype-dev \
    libzip-dev \
    libxml2-dev \
    oniguruma-dev \
    libxslt-dev \
    supervisor \
    mysql-client \
    openssh \
    g++ \
    autoconf \
    make \
    shadow \
    zlib-dev \
    linux-headers

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

COPY . .

EXPOSE 9000

CMD ["php-fpm"]

```