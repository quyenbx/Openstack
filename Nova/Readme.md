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
```sh
openstack flavor create --id auto --ram <dung lượng ram> --disk <dung lượng disk> --vcpu <số lượng cpu> --public <tên flavor>
```
Lưu ý: Đơn vị dung lượng Ram tính bằng MB và Disk tính bằng GB
- Liệt kê flavors
```sh
openstack flavor list
```

- Hiển thị chi tiết một flavor
```sh
openstack flavor show <tên hoặc ID của flavor>
```

- Xóa bỏ 1 flavor
```sh
openstack flavor delete <name or id của flavor>
```

- Tạo keypair
```sh
openstack keypair create [--public-key <file> | --private-key <file>] <name>
```

- List tất cả các key pair có trong openstack
```sh
openstack keypair list
```

- Xóa bỏ 1 keypair
```sh
openstack keypair delete <tên keypair>
```

- Tạo máy ảo từ image
```sh
openstack server create --flavor <tên flavor> --image <tên image> --nic net-id=<id của network> --security-group <tên security group> --key-name <tên keypair> <tên vm>
```

- Tạo máy ảo từ volume
```sh
openstack server create --flavor <tên flavor> --volume <tên volume> --nic net-id=<id của network> --security-group <tên security group> --key-name <tên keypair> <tên vm>
```

- Xóa máy ảo
```sh
openstack server delete <tên VM>
```

- Tắt máy ảo
```sh
openstack server stop <tên VM>
```

- Bật máy ảo
```sh
openstack server start <tên VM>
```

- Reboot máy ảo
```sh
openstack server reboot <tên VM>
```

- List tất cả máy ảo
```sh
openstack server list
```

- Tạo mới snapshot
```sh
openstack snapshot create <tên snapshot> <tên máy ảo>
```

- Hiển thị danh sách các snapshot
```sh
openstack snapshot list
```

- Xóa snapshot
```sh
openstack snapshot delete <tên hoặc ID của snapshot>
```

- Xem danh sách các hypervisor
```sh
openstack hypervisor list
```
