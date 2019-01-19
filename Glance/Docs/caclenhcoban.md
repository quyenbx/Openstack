## Các lệnh thường dùng trong Glance

- Hiển thị danh sách các image: 
`openstack image list`

- Hiển thị chi tiết một image: `openstack image show <tên hoặc ID của Image>'

- Tạo image
```sh
openstack image create <tên image> --disk-format qcow2 --container-format bare --file <path file image cần upload> --public 
```

- Xóa image
```sh
openstack image delete <tên hoặc ID của image>
```

- Lưu lại một image về 1 thư mục
```sh
openstack image save --file <filename> <tên image>
```