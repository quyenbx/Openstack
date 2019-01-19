## Cơ chế filter và weight của cinder-scheduler

## Mục lục

## 1. Giới thiệu tổng quan về cinder-scheduler

- Là một daemon chịu trách nhiệm cho việc quyết định xem sẽ tạo cinder volume ở đâu khi mô hình có hơn một backend storage. Mặc định nếu người dùng không chỉ rõ host để tạo máy ảo thì cinder-scheduler sẽ thực hiện filter và weight theo những opions sau

![](./images/2.png)

- Khi sử dụng sử dụng multiple-storage back ends ta bắt buộc phải filter_scheduler

## 2. Cinder Scheduler Filters

- AvailabilityZoneFilter : Filter bằng availability zone
- CapabilitiesFilter : Filter theo tài nguyên (máy ảo và volume)
- CapacityFilter : Filter dựa vào công suất sử dụng của volume backend
- DifferentBackendFilter : Lên kế hoạch đặt các volume ở các backend khác nhau khi có 1 danh sách các volume
- DriverFilter : Dựa vào ‘filter function’ và metrics.
- InstanceLocalityFilter : lên kế hoạch cho các volume trên cùng 1 host. Để có thể dùng filter này thì Extended Server Attributes cần được bật bởi nova và user sử dụng phải được khai báo xác thực trên cả nova và cinder.
- JsonFilter : Dựa vào JSON-based grammar để chọn lựa backends
- RetryFilter : Filter những node chưa từng được schedule
- SameBackendFilter : Lên kế hoạch đặt các volume có cùng backend như những volume khác

## 3. Cinder Scheduler Weights

- AllocatedCapacityWeigher : Allocated Capacity Weigher sẽ tính trọng số của host bằng công suất được phân bổ. Nó sẽ đặt volume vào host được khai báo chiếm ít tài nguyên nhất.
- CapacityWeigher : Trạng thái công suất thực tế chưa được sử dụng.
- ChanceWeigher : Tính trọng số random, dùng để tạo các volume khi các host gần giống nhau
- GoodnessWeigher : Gán trọng số dựa vào goodness function.
  - 0 -- host is a poor choice
  - 50 -- host is a good choice
  - 100 -- host is a perfect choice

