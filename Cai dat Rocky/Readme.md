## Tài liệu cài đặt Openstack Rocky trên Centos 7 64-bit

## Mục Lục

## 1. Mô Hình:
- Mô hình cài đặt Lab OPS Rocky trên Centos 7 64-bit
![](./images/sodoketnoi.png)

## 2. IP Planning:
- Phân hoạch địa chỉ Ip và yêu cầu phần cứng
![](./images/IP.png)

## 3. Cài đặt môi trường :

### 3.1 Cài đặt môi trường trên node controller:

```sh
yum update -y
```

- Cấu hình Network

```sh
hostnamectl set-hostname CTL
echo "Setup IP  ens32"
nmcli c modify ens32 ipv4.addresses 10.10.10.206/24
nmcli c modify ens32 ipv4.gateway 10.10.10.1
nmcli c modify ens32 ipv4.dns 8.8.8.8
nmcli c modify ens32 ipv4.method manual
nmcli con mod ens32 connection.autoconnect yes
 
echo "Setup IP  ens33"
nmcli c modify ens33 ipv4.addresses 10.10.12.206/24
nmcli c modify ens33 ipv4.method manual
nmcli con mod ens33 connection.autoconnect yes
 
echo "Setup IP  ens34"
nmcli c modify ens34 ipv4.addresses 10.10.11.206/24
nmcli c modify ens34 ipv4.method manual
nmcli con mod ens34 connection.autoconnect yes
 
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
 
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
init 6
```

### 3.2 Cài đặt môi trường trên Node Compute 1:
 
```sh
yum update -y
```

- Cấu hình Network

```sh
hostnamectl set-hostname COM1
echo "Setup IP  ens32"
nmcli c modify ens32 ipv4.addresses 10.10.10.207/24
nmcli c modify ens32 ipv4.gateway 10.10.10.1
nmcli c modify ens32 ipv4.dns 8.8.8.8
nmcli c modify ens32 ipv4.method manual
nmcli con mod ens32 connection.autoconnect yes
 
echo "Setup IP  ens33"
nmcli c modify ens33 ipv4.addresses 10.10.12.207/24
nmcli c modify ens33 ipv4.method manual
nmcli con mod ens33 connection.autoconnect yes
 
echo "Setup IP  ens34"
nmcli c modify ens34 ipv4.addresses 10.10.11.207/24
nmcli c modify ens34 ipv4.method manual
nmcli con mod ens34 connection.autoconnect yes
 
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
 
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
init 6
```

### 3.3 Cài đặt NTP

- Cài đặt trên node Controller

```sh
yum install chrony -y
sed -i 's/0.centos.pool.ntp.org/0.asia.pool.ntp.org/g' /etc/chrony.conf
sed -i 's/1.centos.pool.ntp.org/1.asia.pool.ntp.org/g' /etc/chrony.conf
sed -i 's/2.centos.pool.ntp.org/2.asia.pool.ntp.org/g' /etc/chrony.conf
sed -i 's/3.centos.pool.ntp.org/3.asia.pool.ntp.org/g' /etc/chrony.conf
echo "allow 10.10.10.0/24" >> /etc/chrony.conf
systemctl enable chronyd.service
systemctl start chronyd.service
```

- Cài đặt trên node Compute

```sh
yum install chrony -y
sed -i 's/server 0.centos.pool.ntp.org iburst/#server 0.centos.pool.ntp.org iburst/g' /etc/chrony.conf
sed -i 's/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/g' /etc/chrony.conf
sed -i 's/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/g' /etc/chrony.conf
sed -i 's/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/g' /etc/chrony.conf
echo "server 10.10.10.206 iburst" >> /etc/chrony.conf
systemctl enable chronyd.service
systemctl start chronyd.service
```

- Để kiểm tra hoạt động của chrony trên controller và compute gõ lệnh

  ```sh
  chronyc sources
  ```
### 3.4 Cài đặt các OpenStack packages

- Trên node Controller chạy lệnh sau

```sh
yum install centos-release-openstack-rocky -y
yum upgrade -y
yum install python-openstackclient -y
yum install openstack-selinux -y
init 6
```
### 3.5 Cài đật SQL database

- Cài đặt trên node controller

```sh
yum install mariadb mariadb-server python2-PyMySQL -y
```

```sh
cd /etc/my.cnf.d/
cat >> openstack.cnf << "EOF"
[mysqld]
bind-address = 10.10.10.206

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
```

```sh
systemctl enable mariadb.service
systemctl start mariadb.service
```
- Tạo pass root mysql 
```sh
mysql_secure_installation
```
### 3.6 Message queue
- Cài đặt rabbitMQ trong node controller

```sh
yum install rabbitmq-server -y
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
```

- Tao user openstack

```sh
rabbitmqctl add_user openstack RABBIT_PASS
```

Note: RABBIT_PASS la pass sử dụng cho user openstack

- Cho phép user openstack có quyền cấu hình, đọc viết

```sh
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

### 3.7 Cài đătj Memcached
- Tiến hành cài đặt trên node Controller

```sh
yum install memcached python-memcached -y
sed -i 's/OPTIONS="-l 127.0.0.1,::1"/OPTIONS="-l 127.0.0.1,10.10.10.206"/g' /etc/sysconfig/memcached
systemctl enable memcached.service
systemctl start memcached.service
```

## 4. Cài đặt Identity Servvice - Keystone
### 4.1 Chuẩn bị
- Keystone sẽ được cài đặt trên node Controller
- Trước khi cài đặt và cấu hình Identity service, chúng ta cần tạo 1 database cho service
  - Truy cập kết nối tới database server với tài khoản root
  ```sh
  mysql -u root -p
  ```
  - Tạo keystone database
  ```sh
  CREATE DATABASE keystone;
  ```
  - Gán quyền truy câp keystone database cho user
  ```sh
  GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';
  GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';
  FLUSH PRIVILEGES;
  quit
  ```
   - Với KEYSTONE_DBPASS là pass của user keystone trong database server
   
### 4.2 Cài đặt và cấu hình :
- Cài đặt các packages cần thiết

```sh
yum install openstack-keystone httpd mod_wsgi -y
```

- Sửa file /etc/keystone/keystone.conf

```sh
cd /etc/keystone/
mv keystone.conf keystone.conf.bak
cat >> keystone.conf << "EOF"
[DEFAULT]
[application_credential]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[credential]
[database]
connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@10.10.10.206/keystone
[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[federation]
[fernet_tokens]
[healthcheck]
[identity]
[identity_mapping]
[ldap]
[matchmaker_redis]
[memcache]
[oauth1]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[policy]
[profiler]
[resource]
[revoke]
[role]
[saml]
[security_compliance]
[shadow_users]
[signing]
[token]
provider = fernet
[tokenless_auth]
[trust]
[unified_limit]
[wsgi]
EOF
chmod 640 keystone.conf
chown root:keystone keystone.conf
```

Note: KEYSTONE_DBPASS là pass của user keystone trong database server, 10.10.10.206 là IP tương ứng của manager trên note controller

```sh
su -s /bin/sh -c "keystone-manage db_sync" keystone
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

```sh
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

Note: controller ở đây thay bằng IP của note controller, ADMIN_PASS thay bằng pass admin muốn thiết lập cho hệ thống

- Cấu hình config apache
```sh
echo "ServerName controller" >> /etc/httpd/conf/httpd.conf
```
Note: controller thay bằng hostname của node Controller

```sh
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
systemctl enable httpd.service
systemctl start httpd.service
```

- Tạo môi trường cho client trong openstack
  - Tạo 1 file **admin-openrc** với nội dung như bên dưới
  ```sh
  cd
  cat >> admin-openrc << "EOF"
  export OS_PROJECT_DOMAIN_NAME=Default
  export OS_USER_DOMAIN_NAME=Default
  export OS_PROJECT_NAME=admin
  export OS_USERNAME=admin
  export OS_PASSWORD=ADMIN_PASS
  export OS_AUTH_URL=http://controller:5000/v3
  export OS_IDENTITY_API_VERSION=3
  export OS_IMAGE_API_VERSION=2
  EOF
  ```
  Note: Thay đổi nội dung ADMIN_PASS với pass tương ứng và controller thay bằng IP manager node controller
  

  
  














  
  














