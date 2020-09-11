## Preparation
### SSH to VPS
SSH lên server lần đầu tiên

```text
1. ssh [user_name]@[IP]
2. Nhập password
```

### Add SSH key to VPS
Mở file `authorized_keys` nằm trong thư mục **/root/.ssh/authorized_keys** để add ssh key của máy mình lên đó.

```text
nano .ssh/authorized_keys
```

### Create IAM user

 Sử dụng lệnh `adduser`. Mặc định nó sẽ nằm trong thư mục `/home/[username]` và được copy từ thư mục `/etc/skel`.

```text
adduser [username]
```

Xóa user bằng lệnh `deluser`

```text
deluser [username]
```

Tạo thư mục `.ssh/` và file `authorized_keys` cho user vừa tạo, sau đó add ssh key máy cho user đó để thực hiện connect tới server bằng account đó.

## Installation

https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql

Cài đặt trên tài khoản root của Server

### Install environment
**Nginx**

Link tham khảo, [tại đây](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04).

Câu lệnh cài Nginx
```text
apt-get install nginx
```

**PHP và PHP-FPM** (7.3)

Link tham khảo, [tại đây](https://www.rosehosting.com/blog/how-to-install-php-7-3-on-ubuntu-16-04/).

Câu lệnh cài php
```text
LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php
apt-get update
apt-get install php7.3 php7.3-cli php7.3-common
apt-get install php-pear php7.3-curl php7.3-dev php7.3-gd php7.3-mbstring php7.3-zip php7.3-mysql php7.3-xml php7.3-fpm php7.3-imagick php7.3-recode php7.3-tidy php7.3-xmlrpc php7.3-intl
```

Nếu nhỡ cài `libapache2-mod-php7.3` thì xóa file `index.html` trong thư mục `/var/www/html`.

**Config Nginx với PHP-fpm**

```text
cd /etc/nginx/sites-available
touch conf
```
Thêm đoạn config sau vào file vừa tạo

```text
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /var/www/html/[name_project]/public;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name localhost;

    location / {
      try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
      try_files $uri /index.php =404;
      fastcgi_pass unix:/var/run/php/php-fpm.sock;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
Sau đó thực hiện symlink từ file vừa tạo sang thư mục `//etc/nginx/sites-enabled/` bằng câu lệnh:

```text
sudo ln -s /etc/nginx/sites-available/[file_name] /etc/nginx/sites-enabled/
```
kiểm tra lại bằng `nginx -t` nếu thấy báo successfully thì thành công.

**MySQL**

Link tham khảo, [tại đây](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-16-04).

Câu lệnh cài MySQL:
```text
sudo apt-get update
sudo apt-get install mysql-server
```
Kiểm tra trạng thái của MySQL
```text
systemctl status mysql.service
```
**Redis**

Link tham khảo, [tại đây](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-redis-on-ubuntu-16-04).

### Create user deploy

**Add user deploy and generate SSH key**

```text
adduser deploy
ssh-keygen -t rsa (Generate SSH key)
cat /home/deploy/.ssh/id_rsa (Private key)
cat /home/deploy/.ssh/id_rsa.pub (Public key)
```

Note: Setup password cho user deploy

**Config run Nginx as user deploy**

Vì thư mục source code sẽ nằm trong thằng deploy nên cần để deploy là owner của thư mục đó. Khi đó Nginx sẽ phải chạy bằng deploy để có quyền thực thi các file. Hoăc có cách khác là để www-data (mặc định của Nginx) trở thành owner của project, nhưng không nên làm theo cách này.

```text
sudo nano /etc/nginx/nginx.conf
Sửa user www-data => user deploy
sudo /etc/init.d/nginx restart
```
