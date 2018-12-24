#                                                    Mục Lục
## 1. Chuẩn bị:
### 1.1. Giới thiệu:
### 1.2. Môi trường thực hiện:
### 1.3. Mô Hình:
### 1.4. IP Planning:
## 2. Các bước cài đặt:
### 2.1. Setup trên Controller:
### 2.2. Setup trên Compute 1:
### 2.3. Setup trên Compute 2:
### 2.4. Thực hiện cài đặt trên Controller:
### 2.5. Kiểm tra hệ thống sau khi cài đặt:
### 2.6. Sử dụng Dashboad:
===========================================

## 1. Các Bước Chuẩn Bị:
### 1.1. Giới thiệu:
Lưu ý: Trong tài liệu này chỉ thực hiện cài đặt OpenStack, bước cài đặt CEPH ở tài liệu khác.
- Packstack là một công cụ cài đặt OpenStack nhanh chóng.
- Packstack được phát triển bởi redhat
- Chỉ hỗ trợ các distro: RHEL, Centos
- Tự động hóa các bước cài đặt và lựa chọn thành phần cài đặt.

Nhanh chóng dựng được môi trường OpenStack để sử dụng làm PoC nội bộ, demo khách hàng, test tính năng.

**Nhược điểm 1** : Đóng kín các bước cài đối với người mới.

**Nhược điểm 2** : Khó bug các lỗi khi cài vì đã được đóng gói cùng với các tool cài đặt tự động (puppet)

### 1.2. Môi Trường Thực Hiện:
Sử dụng VMWare ESXI để tạo các máy cài đặt OpenStack
- Distro: CentOS 7.6.1810
- OpenStack Queens

Sử dụng 03 NICs đối với các máy: Controller1, Compute1, Compute2. Các dải mạng đều là VLAN được tạo từ VMWARE ESXi
- **NIC1** - ens32: là dải mạng sử dụng cho các traffic MGNT + API + Horizon
- **NIC2** - ens33: Là dải mạng DATA VM mà các máy ảo sẽ truyền thông với nhau (giữa máy ảo với máy ảo).
- **NIC3** - ens34: Là dải mạng Provider mà các máy ảo sẽ giao tiếp với bên ngoài.

### 1.3. Mô Hình:
<img src="https://imgur.com/a/BGXcAnn">

### 1.4. IP Planning:
## 2. Các Bước Cài Đặt:
### 2.1. Setup Trên Controller:
- Setup môi trường
```sh
hostnamectl set-hostname controller
echo "Setup IP  ens32"
nmcli c modify ens32 ipv4.addresses 10.10.10.206/24
nmcli c modify ens32 ipv4.gateway 10.10.10.1
nmcli c modify ens32 ipv4.dns 8.8.8.8
nmcli c modify ens32 ipv4.method manual
nmcli con mod ens32 connection.autoconnect yes
 
echo "Setup IP  ens33"
nmcli c modify ens33 ipv4.addresses 10.10.11.206/24
nmcli c modify ens33 ipv4.method manual
nmcli con mod ens33 connection.autoconnect yes
 
echo "Setup IP  ens34"
nmcli c modify ens34 ipv4.addresses 10.10.12.206/24
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

- Setup tải các gói OpenStack

```sh
yum install -y epel-release
sudo yum install -y byobu
byobu
 
sed -i 's|enabled=1|enabled=0|'g /etc/yum.repos.d/epel.repo
echo "proxy=http://10.10.10.41:3142" >> /etc/yum.conf
 
yum install -y python-setuptools
sudo yum install -y centos-release-openstack-queens
yum update -y
 
sudo yum install -y wget crudini fping
yum install -y openstack-packstack
```

### 2.2. Setup Trên Compute 1:
- Setup môi trường
```sh
hostnamectl set-hostname compute1
echo "Setup IP  ens32"
nmcli c modify ens32 ipv4.addresses 10.10.10.207/24
nmcli c modify ens32 ipv4.gateway 10.10.10.1
nmcli c modify ens32 ipv4.dns 8.8.8.8
nmcli c modify ens32 ipv4.method manual
nmcli con mod ens32 connection.autoconnect yes
 
echo "Setup IP  ens33"
nmcli c modify ens33 ipv4.addresses 10.10.11.207/24
nmcli c modify ens33 ipv4.method manual
nmcli con mod ens33 connection.autoconnect yes
 
echo "Setup IP  ens34"
nmcli c modify ens34 ipv4.addresses 10.10.12.207/24
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

- Setup package OpenStack

```sh
yum install -y epel-release
sudo yum install -y byobu
byobu
 
sed -i 's|enabled=1|enabled=0|'g /etc/yum.repos.d/epel.repo
echo "proxy=http://10.10.10.41:3142" >> /etc/yum.conf
yum install -y python-setuptools
sudo yum install -y centos-release-openstack-queens
yum update -y
 
sudo yum install -y wget crudini fping
yum install -y openstack-packstack
```

### 2.3. Setup trên Compute 2:
- Setup môi trường
```sh
hostnamectl set-hostname compute1
echo "Setup IP  ens32"
nmcli c modify ens32 ipv4.addresses 10.10.10.208/24
nmcli c modify ens32 ipv4.gateway 10.10.10.1
nmcli c modify ens32 ipv4.dns 8.8.8.8
nmcli c modify ens32 ipv4.method manual
nmcli con mod ens32 connection.autoconnect yes
 
echo "Setup IP  ens33"
nmcli c modify ens33 ipv4.addresses 10.10.11.208/24
nmcli c modify ens33 ipv4.method manual
nmcli con mod ens33 connection.autoconnect yes
 
echo "Setup IP  ens34"
nmcli c modify ens34 ipv4.addresses 10.10.12.208/24
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

- Setup package OpenStack

```sh
yum install -y epel-release
sudo yum install -y byobu
byobu
 
sed -i 's|enabled=1|enabled=0|'g /etc/yum.repos.d/epel.repo
echo "proxy=http://10.10.10.41:3142" >> /etc/yum.conf
yum install -y python-setuptools
sudo yum install -y centos-release-openstack-queens
yum update -y
 
sudo yum install -y wget crudini fping
yum install -y openstack-packstack
```

### 2.4. Thực hiện cài đặt trên Controller:
- Cài đặt Packstack
```sh
byobu
```

```sh
packstack packstack --gen-answer-file=/root/rdotraloi.txt \
    --allinone \
    --default-password=Welcome123 \
    --os-cinder-install=y \
    --os-ceilometer-install=n \
    --os-trove-install=n \
    --os-ironic-install=n \
    --os-swift-install=n \
    --os-panko-install=n \
    --os-heat-install=n \
    --os-magnum-install=n \
    --os-aodh-install=n \
    --os-neutron-ovs-bridge-mappings=extnet:br-ex \
	--os-neutron-ovs-bridge-interfaces=br-ex:ens34 \
    --os-neutron-ovs-bridges-compute=br-ex \
    --os-neutron-ml2-type-drivers=vxlan,flat \
    --os-controller-host=10.10.10.206 \
    --os-compute-hosts=10.10.10.207,10.10.10.208 \
    --os-neutron-ovs-tunnel-if=ens33 \
    --provision-demo=n
```

```sh
packstack --answer-file rdotraloi.txt
```

- Setup sau khi cài đặt xong trên Controller:

```sh
sed -i -e 's/enable_isolated_metadata=False/enable_isolated_metadata=True/g' /etc/neutron/dhcp_agent.ini
ssh -o StrictHostKeyChecking=no root@10.10.10.207 "sed -i -e 's/COM1/10.10.10.207/g' /etc/nova/nova.conf"
ssh -o StrictHostKeyChecking=no root@10.10.10.208 "sed -i -e 's/COM2/10.10.10.208/g' /etc/nova/nova.conf"
 
 
systemctl stop iptables
systemctl disable iptables
 
ssh -o StrictHostKeyChecking=no root@10.10.10.207 "systemctl stop iptables"
ssh -o StrictHostKeyChecking=no root@10.10.10.207 "systemctl disable iptables"
 
ssh -o StrictHostKeyChecking=no root@10.10.10.208 "systemctl stop iptables"
ssh -o StrictHostKeyChecking=no root@10.10.10.208 "systemctl disable iptables"
 
ssh -o StrictHostKeyChecking=no root@10.10.10.207 "init 6"
ssh -o StrictHostKeyChecking=no root@10.10.10.208 "init 6"
init 6
```

- Sau khi cài đặt hoàn tất, trên màn hình sẽ xuất hiên thông báo về các kết quả cài đặt cùng với link truy cập và các tài khoản.
- Truy cập vào web theo địa chỉ http://10.10.10.206/dashboard
 - Tài khoản : admin
 - Mật khẩu : Welcome123




