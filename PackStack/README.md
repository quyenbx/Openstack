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