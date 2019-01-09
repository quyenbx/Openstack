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
  
## 5. Cài đặt Image Service - Glance

### 5.1 Chuẩn bị
- Glance được tiến hành cài đặt trên node controller
- Tạo database
```sh
mysql -u root -p
```

```sh
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'GLANCE_DBPASS';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';
FLUSH PRIVILEGES;
quit;
```

Note: GLANCE_DBPASS là pass của user glance trong database

- Tạo thông tin cho service GLANCE_DBPASS

```sh
source /root/admin-openrc
```
  - Tạo project service
  ```sh
  openstack project create --description "service project" service
  ```
  
  - Tạo user glance
  ```sh
  openstack user create --domain default --password-prompt glance
  ```

  - Gán quyền admin cho user glance trong project service
  ```sh
  openstack role add --project service --user glance admin
  ```
  
  - Tạo service glance
  ```sh
  openstack service create --name glance --description "OpenStack Image" image
  ```
  
- Tạo API endpoint cho image service
```sh
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292
```

Note: controller là IP manager của node controller

### 5.1 Cài đặt và cấu hình
- Cài đặt packages
```sh
yum install openstack-glance -y
```

- Sửa file cấu hình glance-api.conf

```sh
cd /etc/glance/
mv glance-api.conf glance-api.conf.bak
cat >> glance-api.conf << "EOF"
[DEFAULT]
[cors]
[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
www_authenticate_uri  = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = GLANCE_PASS
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]
EOF
chmod 640 glance-api.conf
chown root:glance glance-api.conf
```

Note: Với GLANCE_DBPASS là pass của user glance trong database server, controller là IP manager của node controller, GLANCE_PASS là pass của user glance

- Sửa file cấu hình glance-registry.conf

```sh
cd /etc/glance/
mv glance-registry.conf glance-registry.conf.bak
cat >> glance-registry.conf << "EOF"
[DEFAULT]
[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = GLANCE_PASS
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
EOF
chmod 640 glance-registry.conf
chown root:glance glance-registry.conf
```

Note: GLANCE_DBPASS là pass của user glance trong database, controller là IP manage của node controller, GLANCE_PASS là pass của user glance

```sh
su -s /bin/sh -c "glance-manage db_sync" glance
systemctl enable openstack-glance-api.service openstack-glance-registry.service
systemctl start openstack-glance-api.service openstack-glance-registry.service
```

### 5.2 Kiểm tra hoạt động của glance

```sh
source /root/admin-openrc
```

- Download source cirros
```sh
wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
```

- Upload Image
```sh
openstack image create "cirros" --file cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```

- Xác nhận upload image
```sh
openstack image list
```

## 6. Cài đặt Compute service - Nova
### 6.1. Cài đặt trên node controller
- Chuẩn bị

  - Tạo database
  ```sh
  mysql -u root -p
  ```
  
  ```sh
  CREATE DATABASE nova_api;
  CREATE DATABASE nova;
  CREATE DATABASE nova_cell0;
  CREATE DATABASE placement;
  GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'PLACEMENT_DBPASS';
  GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'PLACEMENT_DBPASS';
  FLUSH PRIVILEGES;
  quit;
  ```
  Note: Thay thế NOVA_DBPASS và PLACEMENT_DBPASS với pass tương ứng
  
- Thao tác với quyền admin

```sh
source /root/admin-openrc
```
  - Tạo user Nova
  ```sh
  openstack user create --domain default --password-prompt nova
  ```
  
  - Cấp quyền admin cho user nova trong project service
  ```sh
  openstack role add --project service --user nova admin
  ```
  
  - Tạo service Nova
  ```sh
  openstack service create --name nova --description "OpenStack Compute" compute
  ```

- Tạo Compute API service endpoints

```sh
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
```

Note: thay đổi controller thành IP manager của node controller

- Tạo user placement
```sh
openstack user create --domain default --password-prompt placement
```

- Gán quyền admin cho user placement trong project service
```sh
openstack role add --project service --user placement admin
```

- Tạo Placement API trong danh mục service
```sh
openstack service create --name placement --description "Placement API" placement
openstack endpoint create --region RegionOne placement public http://controller:8778
openstack endpoint create --region RegionOne placement internal http://controller:8778
openstack endpoint create --region RegionOne placement admin http://controller:8778
```

Note: Thay controller bằng IP manager của node controller

- Cài đặt và cấu hình các thành phần
  - Cài đặt các packages
  ```sh
  yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api -y
  ```
  
  - Sửa file cấu hình
  ```sh
  cd /etc/nova/
  mv nova.conf nova.conf.bak
  cat >> nova.conf << "EOF"
  [DEFAULT]
  enabled_apis = osapi_compute,metadata
  transport_url = rabbit://openstack:RABBIT_PASS@controller
  my_ip = 10.10.10.206
  use_neutron = true
  firewall_driver = nova.virt.firewall.NoopFirewallDriver
  [api]
  auth_strategy = keystone
  [api_database]
  connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
  [barbican]
  [cache]
  [cells]
  [cinder]
  [compute]
  [conductor]
  [console]
  [consoleauth]
  [cors]
  [database]
  connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
  [devices]
  [ephemeral_storage_encryption]
  [filter_scheduler]
  [glance]
  api_servers = http://controller:9292
  [guestfs]
  [healthcheck]
  [hyperv]
  [ironic]
  [key_manager]
  [keystone]
  [keystone_authtoken]
  auth_url = http://controller:5000/v3
  memcached_servers = controller:11211
  auth_type = password
  project_domain_name = default
  user_domain_name = default
  project_name = service
  username = nova
  password = NOVA_PASS
  [libvirt]
  [matchmaker_redis]
  [metrics]
  [mks]
  [neutron]
  [notifications]
  [osapi_v21]
  [oslo_concurrency]
  lock_path = /var/lib/nova/tmp
  [oslo_messaging_amqp]
  [oslo_messaging_kafka]
  [oslo_messaging_notifications]
  [oslo_messaging_rabbit]
  [oslo_messaging_zmq]
  [oslo_middleware]
  [oslo_policy]
  [pci]
  [placement]
  region_name = RegionOne
  project_domain_name = Default
  project_name = service
  auth_type = password
  user_domain_name = Default
  auth_url = http://controller:5000/v3
  username = placement
  password = PLACEMENT_PASS
  [placement_database]
  connection = mysql+pymysql://placement:PLACEMENT_DBPASS@controller/placement
  [powervm]
  [profiler]
  [quota]
  [rdp]
  [remote_debug]
  [scheduler]
  [serial_console]
  [service_user]
  [spice]
  [upgrade_levels]
  [vault]
  [vendordata_dynamic_auth]
  [vmware]
  [vnc]
  enabled = true
  server_listen = $my_ip
  server_proxyclient_address = $my_ip
  [workarounds]
  [wsgi]
  [xenserver]
  [xvp]
  [zvm]
  EOF
  chmod 640 nova.conf
  chown root:nova nova.conf
  ```
  
  Note: Controller thay bằng IP manager của node controller, RABBIT_PASS thay bằng pass của rabbit, NOVA_DBPASS thay bằng pass của user nova trong database server, NOVA_PASS thay bằng pass của user nova, PLACEMENT_DBPASS thay bằng pass của user placement, PLACEMENT_DBPASS thay bằng pass của user placement trong database server,my_ip là IP manager của node controller
  
  - Cho phép truy cập Placement API bằng cách sửa file 00-nova-placement-api.conf
  ```sh
  cd /etc/httpd/conf.d/
  mv 00-nova-placement-api.conf 00-nova-placement-api.conf.bak
  cat >> 00-nova-placement-api.conf << "EOF"
  Listen 8778
  <VirtualHost *:8778>
  WSGIProcessGroup nova-placement-api
  WSGIApplicationGroup %{GLOBAL}
  WSGIPassAuthorization On
  WSGIDaemonProcess nova-placement-api processes=3 threads=1 user=nova group=nova
  WSGIScriptAlias / /usr/bin/nova-placement-api
  <IfVersion >= 2.4>
    ErrorLogFormat "%M"
  </IfVersion>
  ErrorLog /var/log/nova/nova-placement-api.log
  #SSLEngine On
  #SSLCertificateFile ...
  #SSLCertificateKeyFile ...
  </VirtualHost>
  Alias /nova-placement-api /usr/bin/nova-placement-api
  <Location /nova-placement-api>
  SetHandler wsgi-script
  Options +ExecCGI
  WSGIProcessGroup nova-placement-api
  WSGIApplicationGroup %{GLOBAL}
  WSGIPassAuthorization On
  </Location>
  <Directory /usr/bin>
  <IfVersion >= 2.4>
  Require all granted
  </IfVersion>
  <IfVersion < 2.4>
  Order allow,deny
  Allow from all
  </IfVersion>
  </Directory>
  EOF
  chmod 640 00-nova-placement-api.conf
  systemctl restart httpd
  ```
  
  ```sh
  su -s /bin/sh -c "nova-manage api_db sync" nova
  ```
  
  - Register cell0 database
  ```sh
  su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
  ```
  
  - Tạo cell1 cell
  ```sh
  su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova 109e1d4b-536a-40d0-83c6-5f121b82b650
  ```
  
  ```sh
  su -s /bin/sh -c "nova-manage db sync" nova
  ```
  
  - Xác nhận cell0 and cell1 được register hợp lệ
  ```sh
  su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
  ```
  
  ```sh
  systemctl enable openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
  systemctl start openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
  ```
  
  
  
   
   
  
  








  

  
  














  
  














