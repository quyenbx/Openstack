## Các lệnh cơ bản với cinder

## 1. Tạo, sửa, xóa, liệt kê, hiển thị chi tiết 1 Volume

- Tạo 1 Volume
```sh
openstack volume create --size <dung lượng volume> <tên volume>
```

Note: Ở đây dung lượng volume được tính bằng Gb

- Tạo 2 Volume từ image
```sh
openstack volume create --size <dung lượng volume> --image <tên hoặc ID của image> <tên volume>
```

- Tạo 1 Volume từ Volume khác
```sh
openstack volume create --source <tên volume-source> --size <dung lượng volume> <tên volume>
```

- Tạo 1 Volume từ bản Snapshot
```sh
openstack volume create --snapshot <tên snapshot-source> --size <dung lượng volume> <tên volume>
```

- Xóa Volume
```sh
openstack volume delete <tên hoặc ID volume>
```

- Liệt kê Volume
```sh
openstack volume list
```

- Hiển thị chi tiết 1 Volume
```sh
openstack volume show <tên hoặc ID volume>
```

## 2. Tạo, hiển thị, xóa snapshot Volume

- Tạo snapshot 1 volume
```sh
openstack volume snapshot create --volume <tên hoặc ID của volume để snapshot> <tên snapshot>
```

- List danh sách snapshot
```sh
openstack volume snapshot list
```

- Xóa snapshot
```sh
openstack volume snapshot delete <tên snapshot>
```

## 3. Attach và detach volume cho máy ảo
- Attach Volume
```sh
openstack server add volume <tên VM> <tên volume> --device <tên thiết bị add cho vm>
```

- Detach Volume
```sh
openstack server remove volume <tên VM> <tên volume>
```