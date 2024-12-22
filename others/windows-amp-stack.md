
# Hướng dẫn cài đặt AMP stack (Apache, MySQL, PHP) trên windows
Guide này sẽ hướng dẫn cơ bản về cài đặt AMP stack, giải pháp thay thế cho XAMPP, WAMP, etc... và sử dụng được nhiều PHP version cùng lúc.

## Apache 
1. Truy cập [Apache VS17 binaries and modules download (apachelounge.com)](https://www.apachelounge.com/download/), tải ***Apache 2.4.56 Win64*** (có thể khác đối với những phiên bản sau này), cài đặt ***Visual C++ Redistributable Visual Studio 2015-2022*** theo yêu cầu  
2. Giải nén file zip đã tải xuống, làm theo hướng dẫn trong ***ReadMe.txt***  
3. Ở trang download, tải ***mod_fcgid***, giải nén zip và sao chép ***mod_fcgid.so*** vào thư mục `path_to_apache/modules`  
4. Mở file `path_to_apache/conf/httpd.conf`, sửa lại như sau:  
	```
	...
	#Define some global constant
	Define SRVROOT "path_to_apache"
	Define PHP74ROOT "path_to_php74"
	Define PHP81ROOT "path_to_php81"
	...
	...
	#Define port that apache will listen, 80 for http, 443 for https
	Listen 80
	Listen 443
	...
	...
	#Uncomment this
	LoadModule rewrite_module modules/mod_rewrite.so
	...
	#Uncomment this
	LoadModule ssl_module modules/mod_ssl.so
	...
	#Uncomment this
	LoadModule version_module modules/mod_version.so
	...
	#Uncomment this
	LoadModule vhost_alias_module modules/mod_vhost_alias.so
	...
	#Add this
	LoadModule fcgid_module modules/mod_fcgid.so
	```
5. Khởi động lại apache

## MySQL
Để dễ dàng thì nên sử dụng **MySql Community Installer**, rồi tích chọn **Server Only** trong quá trình cài đặt, hoặc các package khác tùy mục đích sử dụng
[MySQL :: Download MySQL Installer](https://dev.mysql.com/downloads/installer/)

## PHP (multiple version)
Mục tiêu là sử dụng được nhiều phiên bản PHP cùng lúc và chạy cùng lúc nhiều website với phiên bản PHP khác nhau.
1. Tải file zip trên trang [PHP For Windows: Binaries and sources Releases](https://windows.php.net/download), lựa chọn version loại ***Thread Safe***, các phiên bản thường dùng là ***7.4*** và ***8.1***.
2. Tạo folder để lưu trữ file sau khi giải nén, VD download bản 7.4 thì tạo folder `php74`, rồi giải nén toàn bộ file zip mới download vào folder mới tạo.
3. Trong file được giải nén, lưu ý:
	- Copy và đổi tên file `php.ini-production` thành `php.ini`, file này để config các thông số cho php sau này
	- Copy và đổi tên file `php.exe` cho phù hợp với phiên bản tương ứng, VD: php version 7.4 thì đổi tên thành `php74.exe`

## Virtual Host
Sử dụng Vhost để tạo domain ảo, cũng như chạy các version php khác nhau cho từng domain khác nhau. VD sau sẽ hướng dẫn config để chạy một domain ảo sử dụng php version 7.4

1. Tạo thư mục chứa code `D:/somepath/localhost74`, tạo file ***index.php*** với nội dung như sau:
	```php
	<?php
	// Path: D:/somepath/localhost74/index.php
	phpinfo();
	?>
	```
2. Thêm cấu hình vào file `path_to_apache/conf/extra/httpd-vhosts.conf`
	```
	...
	<VirtualHost *:80>
		ServerAdmin webmaster@local.web
		DocumentRoot "D:/somepath/localhost74"
		ServerName localhost74.com
		ServerAlias www.localhost74.com
		ErrorLog "logs/localhost74.com-error.log"
		CustomLog "logs/localhost74.com-access.log" common
		DirectoryIndex index.php
	  
		# Use below statement to load the specific version of php configuration
		FcgidInitialEnv PHPRC "${PHP74ROOT}/php.ini"
		FcgidIOTimeout 84600
		<Directory "D:/somepath/localhost74">
			Options Indexes FollowSymLinks
			AllowOverride All
			Require all granted
			<Files ~ "\.php$">
				AddHandler fcgid-script .php
				FcgidWrapper "${PHP74ROOT}/php-cgi.exe" .php
				Options +ExecCGI
			</Files>
		</Directory>
	</VirtualHost>
	...
	```
3. Restart apache
4. Mở file `C:\Windows\System32\drivers\etc\hosts` thêm địa chỉ IP cho domain ảo, ở đây sử dụng môi trường localhost nên IP sẽ là **127.0.0.1** và tên domain **localhost74.com**
	```
	...
	127.0.0.1	localhost74.com
	...
	```
5. Truy cập localhost74.com, trình duyệt hiển thị thông tin cấu hình php nghĩa là thành công

## Xdebug
Xdebug là một extension của php, sử dụng để hỗ trợ debug php

1. Mở file `php.ini`, tìm và bỏ comment `extension_dir = "ext"`, restart apache, truy cập lại domain ảo localhost74.com
2. Copy toàn bộ nội dung thông tin cấu hình php, truy cập https://xdebug.org/wizard, điền nội dung copy vào ô text, click ***Analyse my phpinfo() output***, rồi làm theo hướng dẫn.
3. Restart apache nếu cần thiết
4. Truy cập lại localhost74.com, tìm kiếm từ khóa xdebug, nếu có thông tin về xdebug là thành công
5. Cấu hình Xdebug
	- Mở **command line**
	- Gõ lệnh `cd "path_to_php74"` để đi tới thư mục cài đặt php
	- Gõ lệnh `php74 -v`
	- Output như sau tức là xdebug đã được cài đặt và đang sử dụng bản 3.1.6
	  ```
	  PHP 7.4.33 (cli) (built: Nov  2 2022 16:00:55) ( ZTS Visual C++ 2017 x64 )
	  Copyright (c) The PHP Group
	  Zend Engine v3.4.0, Copyright (c) Zend Technologies
		  with Xdebug v3.1.6, Copyright (c) 2002-2022, by Derick Rethans
	  ```
	- Mở file `php.ini` và thêm vào cuối file cấu hình của xdebug, lưu lại, restart apache
		```
		...
		[xdebug]
		xdebug.remote_enable = 1
		xdebug.mode = debug
		xdebug.start_with_request = yes
		xdebug.client_host = 127.0.0.1
		xdebug.client_port = 9074
		xdebug.idekey = PHPSTORM
		xdebug.discover_client_host=1
		```
	- Đọc thêm về xdebug [Xdebug: Documentation](https://xdebug.org/docs/)

## Fake SSL
Fake chứng chỉ SSL để truy cập domain ảo với phương thức https

1. Truy cập [Win32/Win64 OpenSSL Installer for Windows - Shining Light Productions (slproweb.com)](https://slproweb.com/products/Win32OpenSSL.html) rồi download **Win64 OpenSSL v3.1.0 Light** bản **EXE**, chạy và cài đặt. Mặc định sẽ có file `C:\Program Files\OpenSSL-Win64\bin\openssl.exe`
2. Add `C:\Program Files\OpenSSL-Win64\bin` vào **Path** của **Environment Variables**
3. Tạo 2 thư mục `path_to_apache\ssl\private` và `path_to_apache\ssl\certs`
4. Mở command line trong thư mục `path_to_apache\ssl`
5. Chạy lệnh sau
	```
	openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout private/localhost74.com.key -out certs/localhost74.com.crt
	```
6. Các bước nhập quan trọng ở bước ***Common Name*** cần nhập đúng tên domain, VD: localhost74.com
	```
	You are about to be asked to enter information that will be incorporated
	into your certificate request.
	What you are about to enter is what is called a Distinguished Name or a DN.
	There are quite a few fields but you can leave some blank
	For some fields there will be a default value,
	If you enter '.', the field will be left blank.
	-----
	Country Name (2 letter code) [AU]:VN
	State or Province Name (full name) [Some-State]:asdas
	Locality Name (eg, city) []:asdasd
	Organization Name (eg, company) [Internet Widgits Pty Ltd]:asdasd
	Organizational Unit Name (eg, section) []:asdasd
	Common Name (e.g. server FQDN or YOUR name) []:localhost74.com
	Email Address []:admin@example.com
	```
7. Sau khi hoàn thành, thu được 2 file `private/localhost74.com.key` và `certs/localhost74.com.crt`
8. Mở file `path_to_apache/conf/extra/httpd-vhosts.conf`, copy cấu hình vhost cũ tạo cấu hình vhost mới như sau
	```
	...
	<VirtualHost *:443>
		ServerAdmin webmaster@local.web
		DocumentRoot "D:/somepath/localhost74"
		ServerName localhost74.com
		ServerAlias www.localhost74.com
		ErrorLog "logs/localhost74.com-error.log"
		CustomLog "logs/localhost74.com-access.log" common
		DirectoryIndex index.php
	  
		# Use below statement to load the specific version of php configuration
		FcgidInitialEnv PHPRC "${PHP74ROOT}/php.ini"
		FcgidIOTimeout 84600
		<Directory "D:/somepath/localhost74">
			Options Indexes FollowSymLinks
			AllowOverride All
			Require all granted
			<Files ~ "\.php$">
				AddHandler fcgid-script .php
				FcgidWrapper "${PHP74ROOT}/php-cgi.exe" .php
				Options +ExecCGI
			</Files>
		</Directory>

		SSLEngine on
		SSLCertificateFile "${SRVROOT}/ssl/certs/localhost74.com.crt"
		SSLCertificateKeyFile "${SRVROOT}/ssl/private/localhost74.com.key"
	</VirtualHost>
	...
	```
## Tips and Tricks
Nothing here yet.


