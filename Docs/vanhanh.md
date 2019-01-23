## Một số thao tác vận hành

- Reset password sử dụng nova set-password: Thao tác lệnh như bên dưới
```sh
source admin-openrc
nova set-password id_vm
```
Note: Với id_vm là id của VM cần reset password

- Extend Volume Root của VM
  - Stop VPS
  - Chuyển trạng thái của volume cần reszise về available
  ```sh
  openstack volume set --state available id_volume
  ```
  Note: Với id_volume là ID của volume cần extend
  - Resize
  ```sh
  openstack volume set --size <new_size> <id_vm>
  ```
  Note: new_size là dung lượng mới của Volume tính theo Gb,id_vm là ID của VM cần resize
  - Start VPS

- Resize Ram, CPU
```sh
openstack server resize --flavor <name_flavor> <id_vm>
openstack server resize --confirm <id_vm>
```
Note: Trong đó name_flavor là tên flavor mới, id_vm là ID của VM muốn chuyển Ram, CPU

- Đổi IP cho VM
  - Tạo port
  ```sh
  openstack port create --network <id_network> --fixed-ip subnet=<id_subnet>,ip-address=<ipaddress> <name-port>
  ```
  Note: id_network là ID của network, id_subnet là ID của subnet của network, ipaddress là địa chỉ IP gán cho port, name-port là tên của port
  - Detach port trên VM
  ```sh
  nova interface-detach <server> <port_id>
  ```
  Note: server là name hoặc ID của VM, port_id là port muốn detach
  - Attach port mới vào VM
  ```sh
  nova interface-attach --port-id <id-port> <id-vm>
  ```
  Note: id-port là id hoặc name của port, id-vm là ID của VM
  
- Thêm IP mới cho VM
  - Tạo port
  ```sh
  openstack port create --network <network-id> <name-port>
  ```
  Note: network-id là id của network, name-port là tên của port
  - Update
  ```sh
  neutron port-update <id-port-vm> --allowed-address-pair ip_address=<ipaddress>
  ```
  Note: id-port-vm là id port của VM cần add thêm IP, ipaddress là IP của port mới tạo
  
- Migrate VM
  - Cold-Migrate
    - Stop VM
	- Migrate với lệnh bên dưới
	```sh
	openstack server migrate <id_vm>
	```
  Note: id_vm là IP của VM cần migrate
    - Confirm để chuyển trạng thái VM
	```sh
	openstack server resize --confirm <id_vm>
	```
	
  - Live-Migrate
    - Chỉnh sửa libvirt trong note Compute
	```sh
	cp /etc/libvirt/libvirtd.conf /etc/libvirt/libvirtd.conf.bak
    sed -i 's|#listen_tls = 0|listen_tls = 0|'g /etc/libvirt/libvirtd.conf
    sed -i 's|#listen_tcp = 1|listen_tcp = 1|'g /etc/libvirt/libvirtd.conf
    sed -i 's|#tcp_port = "16509"|tcp_port = "16509"|'g /etc/libvirt/libvirtd.conf
    sed -i 's|#auth_tcp = "sasl"|auth_tcp = "none"|'g /etc/libvirt/libvirtd.conf
    cp /etc/sysconfig/libvirtd /etc/sysconfig/libvirtd.orig 
    sed -i 's|#LIBVIRTD_ARGS="--listen"|LIBVIRTD_ARGS="--listen"|'g /etc/sysconfig/libvirtd
	```
	- Migrate
	```sh
	openstack server migrate --live <Hostname node compute moi> <ID VM se Migrate>
	```
	


  
