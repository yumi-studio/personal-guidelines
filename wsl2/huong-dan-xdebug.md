# Hướng dẫn cài sử dụng php xdebug trên wsl2

## 1. File wslboot.bat: chạy một số cài đặt cần thiết cho việc tạo kết nối giữa ubuntu và window. NHỚ CHẠY VỚI QUYỀN QUẢN TRỊ CỦA WINDOWS

> Note: Nếu tất cả các bước dưới đây đều không được, có khả năng cao tường lửa vẫn chặn kết nối giữa wsl và windows, trong trường hợp đó, có thể tắt tường lửa

Kiểm tra Ethernet adapter
```
# Chạy command line kiểm tra
ipconfig

Output:
.
.
.
Ethernet adapter vEthernet (WSL (Hyper-V firewall)):

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::a903:b7a8:e208:2392%49
   IPv4 Address. . . . . . . . . . . : 172.21.240.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . :
.
.
.
```
### a. Ethernet adapter = "vEthernet (WSL (Hyper-V firewall))"
```
netsh interface ip add address "vEthernet (WSL (Hyper-V firewall))" 192.168.50.1 255.255.255.0
netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=192.168.50.2
netsh interface portproxy add v4tov4 listenport=443 listenaddress=0.0.0.0 connectport=443 connectaddress=192.168.50.2
wsl -u root ip addr add 192.168.50.2/24 broadcast 192.168.50.255 dev eth0 label eth0:1
powershell -command "New-NetFirewallRule -DisplayName \"WSL\" -Direction Inbound  -InterfaceAlias \"vEthernet (WSL (Hyper-V firewall))\"  -Action Allow"
```
### b. Ethernet adapter = "vEthernet (WSL)"
```
netsh interface ip add address "vEthernet (WSL)" 192.168.50.1 255.255.255.0
netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=192.168.50.2
netsh interface portproxy add v4tov4 listenport=443 listenaddress=0.0.0.0 connectport=443 connectaddress=192.168.50.2
wsl -u root ip addr add 192.168.50.2/24 broadcast 192.168.50.255 dev eth0 label eth0:1
powershell -command "New-NetFirewallRule -DisplayName \"WSL\" -Direction Inbound  -InterfaceAlias \"vEthernet (WSL)\"  -Action Allow"
```
### c. Nếu có nhiều instance được cài đặt thì sử dụng distribution name
```
# Chay command line, kiem tra danh sach instance
wsl -l -v

Output:
  NAME                   STATE           VERSION
* docker-desktop-data    Stopped         2
  Ubuntu20.04LTS1        Running         2
  docker-desktop         Stopped         2
```
Name là "Ubuntu20.04LTS1" thì thay
```
wsl -u root ip addr add 192.168.50.2/24 broadcast 192.168.50.255 dev eth0 label eth0:1
```
thành
```
wsl --distribution Ubuntu20.04LTS1 -u root ip addr add 192.168.50.2/24 broadcast 192.168.50.255 dev eth0 label eth0:1
```

# 2. Config PHPStorm
Vào Settings > PHP > Debug > Xdebug

![image](https://github.com/user-attachments/assets/c4d999fd-cb68-4c4e-a554-744a695c6673)

Debug port để tùy chọn nhưng sẽ liên quan tới config xdebug ở ubuntu. nên đặt theo phiên bản php của từng dự án. VD dự án sử dụng php8.1 thì để port là 9081

![image](https://github.com/user-attachments/assets/f6186438-836c-439d-b566-e153d7b658cd)

Vào Settings > PHP > Server, nếu có cái nào trong này thì xóa hết đi, bấm cái dấu `-`

# 3. Config xdebug trên ubuntu
### 3.1. Điều kiện cần và đủ:
- Đã cài đặt LAMP stack hoặc LEMP stack
- Đã cài đặt multiple php version
- Biết sử dụng và cài đặt vhost với custom domain name hoặc localhost với một custom port
VD Custom domain: toi-la-domain.com trỏ về địa chỉ IP 192.168.59.2
VD Custom port, mở listen port trên apache hoặc nginx với port là 8088 hoặc bất kì port nào chưa được sử dụng:
localhost:8088
127.0.0.1:8088
- Đã cài đặt xdebug extension cho php đúng phiên bản, VD: php8.1-xdebug
```
sudo apt install php8.1-xdebug
```

### 3.2. Config xdebug
```
sudo nano /etc/php/8.1/mods-available/xdebug.ini
```
Cập nhật nội dung như sau:
```
zend_extension=xdebug.so
xdebug.remote_enable=1
xdebug.mode = debug
xdebug.start_with_request = default
xdebug.client_host = 192.168.50.1
xdebug.client_port = 9081
xdebug.idekey = PHPSTORM
xdebug.discover_client_host=1
```
Giải thích ngắn gọn cài đặt
- remote_enable=1: cho phép phpstorm nói riêng và các editor hỗ trợ debug php nói chung, có thể lắng nghe và tiến hành debug trên editor tương ứng
- mode=debug: bật chế độ debug
- start_with_request=default: Có nhiều giá trị khác nhưng ở đây chỉ đề cập tới giá trị default. Có 2 cách để phpstorm kết nối với xdebug trong trường hợp này:
C1: Sử dụng xdebug extension của trình duyệt và đặt breakpoint trong phpstorm.
C2: Hoặc sử dụng hàm đặc biệt khi xdebug được bật là hàm xdebug_break() được gọi trực tiếp trong code. Lưu ý: xdebug chỉ chạy, khi code chạy tới hàm xdebug_break(), bất kì breakpoint được đặt trước đó sẽ không có tác dụng
- client_host=192.168.50.1: Đây là địa chỉ IP của máy nơi mà PHPStorm được mở, hay cụ thể trong trường hợp sử dụng wsl2, thì là địa chỉ IP của windows trong mạng Ethernet của wsl
- client_port=9081: Đây là port mà xdebug sẽ gửi tín hiệu tới, phải trùng khớp với port mà PHPStorm đang lắng nghe
- discover_client_host=1: Không cần quan tâm, cứ để vào
- idekey=PHPSTORM: Không cần quan tâm, cứ để vào

Sau đó lưu lại và restart lại php8.1-fpm
```
sudo systemctl restart php8.1-fpm
hoặc
sudo service php8.1-fpm restart
```


# Chúc bạn may mắn =))
