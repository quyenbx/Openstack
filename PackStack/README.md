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
'''sh
hostnamectl set-hostname controller
echo "Setup IP  ens160"
nmcli c modify ens160 ipv4.addresses 10.10.10.45/24
nmcli c modify ens160 ipv4.gateway 10.10.10..1
nmcli c modify ens160 ipv4.dns 8.8.8.8
nmcli c modify ens160 ipv4.method manual
nmcli con mod ens160 connection.autoconnect yes
 
echo "Setup IP  ens192"
nmcli c modify ens192 ipv4.addresses 10.10.11.45/24
nmcli c modify ens192 ipv4.method manual
nmcli con mod ens192 connection.autoconnect yes
 
echo "Setup IP  ens224"
nmcli c modify ens224 ipv4.addresses 10.10.12.45/24
nmcli c modify ens224 ipv4.method manual
nmcli con mod ens224 connection.autoconnect yes
 
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
 
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
init 6
'''


