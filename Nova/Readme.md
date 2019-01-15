## Tìm hiểu về Compute service - Nova

## Mục Lục

## 1. Giới thiệu về Computer service - Nova
- Nova là service có chức năng quản lý các hệ thống cloud computing.Đây là thành phần trung tâm quan trọng nhất của cấu trúc infrastructure as a service
- OpenStack Compute giao tiếp với OpenStack Identity (Keystone) để xác thực, OpenStack Image để lấy images và OpenStack Dashboard để lấy giao diện cho người dùng và người quản trị
- OpenStack Compute không chứa các phần mềm ảo hóa, Thay vào đó nó sẽ định nghĩa các drivers để tương tác với các kĩ thuật ảo hóa khác chạy trên hệ điều hành của bạn và cung cấp các chức năng thông qua một web-based API

## 2. Các thành phần của Nova
- nova-api : Tiếp nhận và phản hồi các Compute API call từ user 
- nova-api-metadata : Tiếp nhận các request metadata từ máy ảo
- nova-compute : Là service chịu trách nhiệm tạo và hủy các máy ảo qua hypervisors APIs
- nova-placement-api : Được dùng để theo dõi thống kê và muức độ sử dụng của mỗi một resource provider. Provider ở đây có thể là compute node, shared storage pool hoặc IP allocation pool
- nova-scheduler : Service này sẽ lấy các yêu cầu cài đặt máy ảo từ queue và xác định xem chúng được chạy trên compute server host nào
- nova-conductor : Là module chịu trách nhiệm về các tương tác giữa nova-compute và database. Nó sẽ loại bỏ tất cả các kết nối trực tiếp từ nova-compute tới database
- nova-consoleauth : Xác thực tocken cho user mà console proxies cung cấp
- nova-novncproxy : Cung cấp proxy cho việc truy cập các máy ảo đang chạy thông qua VNC connection. Nó hỗ trợ các trình duyệt based novnc clients
- nova-spicehtml5proxy : Cung cấp 1 proxy truy cập máy ảo đang chạy thông qua kết nối SPICE
- nova-xvpvncproxy: Cung cấp 1 proxy cho truy cập máy ảo đang chạy thông qua kết nối VNC
- The queue : Trung tâm giao tiếp giữa các daemons. Thường dùng RabbitMQ hoặc các AMQP message queue khác như ZeroMQ
- SQL database : Dùng để lưu các trạng thái của hạ tâng cloud

## 3. Các câu lệnh thường gặp:
- Tạo mới 1 flavor
- Liệt kê flavors
- Hiển thị chi tiết một flavor
- Xóa bỏ 1 flavor
- Tạo keypair
- List tất cả các key pair có trong openstack
- Xóa bỏ 1 keypair
- Tạo máy ảo từ image
- Tạo máy ảo từ volume
- Xóa máy ảo
- Tắt máy ảo
- Bật máy ảo
- Reboot máy ảo
- List tất cả máy ảo
- Tạo mới snapshot
- Hiển thị danh sách các snapshot
- Xóa snapshot
- Xem danh sách các hypervisor
