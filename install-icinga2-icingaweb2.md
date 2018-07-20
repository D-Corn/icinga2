## Install Icinga2
Cài đặt Icinga2 trên Ubuntu 16.04  
Add icinga repo  
```
wget -O - https://packages.icinga.com/icinga.key | apt-key add -
echo 'deb https://packages.icinga.com/ubuntu icinga-xenial main' >/etc/apt/sources.list.d/icinga.list
apt-get update
```
Cài đặt icinga2  
```
apt-get install icinga2
```
Cài đặt plugin monitor  
```
apt-get install monitoring-plugins
```
Cấu hình syntax highlight trong vim
```
apt-get install vim-icinga2 vim-addon-manager
vim-addon-manager -w install icinga2
```

## Install Icinga Web 2
Cài đặt apache, mysql, php7
```
apt-get install mysql-server php7.0 libapache2-mod-php7.0
```
Cài đặt module IDO cho MySQL
```
apt-get install icinga2-ido-mysql
```
No --> Yes --> Điền password cho icinga2 ido-mysql --> Ok  
Sử dụng MySQL là DB backend cho icinga  
```
icinga2 feature enable ido-mysql command
```
Restart icinga
```
systemctl restart icinga2
```
Cài đặt icinga web
```
apt-get install icingaweb2
```
Set timezone trong file `/etc/php/7.0/apache2/php.ini` tìm dòng `;date.timezone = ` và sửa thành
```
date.timezone = Asia/Ho_Chi_Minh
```
Restart apache
```
/etc/init.d/apache2 restart
```

## Cấu hình Icinga Web
Trên trình duyệt vào trang `http://<IP>/icingaweb2/setup`  
Icinga Web 2 yêu cầu cung cấp token, sử dụng lệnh sau để tạo token  
```
icingacli setup token create;
```
Điền token và chọn Next.  
![B1](picture/B1.png)  
Bước tiếp theo chọn các module để kích hoạt, ở đây chỉ chọn module monitoring.  
![B2](picture/B2.png)  
Kiểm tra các gói PHP cần thiết  
![B3](picture/B3.png)  
Chọn kiểu xác thực cho Icinga2. Chọn database.  
![B4](picture/B4.png)  
Tạo database cho IcingaWeb2  
![B5](picture/B5.png)  
Điền user & password root của mysql  
![B6](picture/B6.png)  
Cung cấp tên DB backend  
![B7](picture/B7.png)  
Tạo tài khoảng đăng nhập IcingaWeb2  
![B8](picture/B8.png)  
Cấu hình log  
![B9](picture/B9.png)  
Verify cấu hình  
![B10](picture/B10.png)  
Cấu hình monitoring backend  
![B11](picture/B11.png)  
Điền các thông tin trong file `/etc/icinga2/features-enabled/ido-mysql.conf` vào  
![B12](picture/B12.png)  
Chọn kiểu send command  
![B13](picture/B13.png)  
Chọn kiểu security  
![B14](picture/B14.png)  
Verify cấu hình  
![B15](picture/B15.png)  

