## Tổng quan về Openstack Image Service - Glance

## Mục Lục

## 1. Giới thiệu về Glance 
## 2. Các thành phần của Glance
## 3. Kiến trúc của Glance
## 4. Các file cấu hình của Glance
## 5. Các dịnh dạng Image của Glance

=============================================================================


## 1. Giới thiệu về Glance
- Image service cho phép user khám phá, đăng ký và truy xuất tới image của máy ảo.Nó chấp nhận các request về Disk image và metadata image máy ảo từ phía người dùng
- Image service hỗ trợ lưu trữ dưới dạng file hệ thống đơn giản tới hệ thống object-storage như OpenStack Object Storage
- Trong Glance, các images được lưu trữ giống như các template. Các Template này sử dụng để vận hành máy ảo mới. Glance là giải pháp để quản lý các ảnh đĩa trên cloud. Nó cũng có thể lấy bản snapshots từ các máy ảo đang chạy để thực hiện dự phòng cho các VM và trạng thái các máy ảo đó

## 2. Các thành phần của Glance
- Glance có một số thành phần như sau
  - glance-api : Tiếp nhận yêu cầu gọi API về tìm kiếm, thu thập và lưu trữ image
  - glance-registry : Tiếp nhận lưu trữ, thu thập và xư lý metadata của image.Metadata bao gồm thông tin về dung lượng và loại của image
  - Database : Lưu trữ metadata của image
  - Storage repository for image files: Có rất nhiều các loại repository được hỗ trợ để lưu image như Object Storage, RADOS block devices, VMware datastore và HTTP
  - Metadata definition service : Một API phổ biến cho các nhà cung cấp, quản trị viên, dịch vụ và người dùng để xác định một metadata của riêng họ
  
## 3. Kiến trúc của Glance

## 4. Các file cấu hình của Glance
- glance-api.conf : File cấu hình cho API của image service
- glance-registry.conf : File cấu hình cho glance image registry - nơi lưu trữ metadata về các images.
- glance-scrubber.conf : Được dùng để dọn dẹp các image đã được xóa
- policy.json : Bổ sung truy cập kiểm soát áp dụng cho các image service. Trong này, chúng tra có thể xác định vai trò, chính sách, làm tăng tính bảo mật trong Glane OpenStack.

## 5. Các dịnh dạng Image của Glance
- Định dạng Disk Formats: Định dạng cơ bản của Disk

| * Disk Format* |                           *Note*                                                              |
| Raw            | Định dạng đĩa phi cấu trúc
| VHD            | Định dạng chung hỗ trợ bởi nhiều công nghệ ảo hóa trong OpenStack, ngoại trừ KVM              | 
| VMDK           | Định dạng hỗ trợ bởi VMWare																	 |
| qcow2          | Định dạng đĩa QEMU, định dạng mặc định hỗ trợ bởi KVM vfa QEMU, hỗ trợ các chức năng nâng cao |
| VDI            | Định dạng ảnh đĩa ảo hỗ trợ bởi VirtualBox													 |
| ISO            | Định dạng lưu trữ cho đĩa quang																 |
| AMI, ARI, AKI  | Định dạng ảnh Amazon machine, ramdisk, kernel												 |

- Định dạng Container Formats: mô tả định dạng files và chứa các thông tin metadata về máy ảo
