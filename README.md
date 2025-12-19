# Ansible Playbooks

Collection các Ansible playbooks được sử dụng bởi AWX (Kafka-driven) để cấu hình các Kubernetes nodes.

## 📋 Mục đích

Thư mục này chứa các playbook Ansible để tự động hóa việc cài đặt và cấu hình các thành phần Kubernetes trên các node Ubuntu 22.04, bao gồm:
- **kubelet**: Kubernetes node agent
- **kubeadm**: Tool để bootstrap Kubernetes cluster
- **kubectl**: Kubernetes command-line tool
- **containerd**: Container runtime

## 📁 Cấu trúc thư mục

```
ansible-playbooks/
├── k8s-node.yml          # Playbook chính để cài đặt K8s node
├── group_vars/
│   └── all.yml          # Biến mặc định cho tất cả hosts
├── .gitignore           # Các file/folder bị ignore
└── README.md            # Tài liệu này
```

## 🎯 Playbooks

### k8s-node.yml

Playbook này thực hiện các tác vụ sau:

1. **Cài đặt dependencies**: Các package cần thiết như `apt-transport-https`, `ca-certificates`, `curl`, `gnupg`, `lsb-release`, `net-tools`

2. **Thêm repository keys**: 
   - Docker GPG key
   - Kubernetes APT key

3. **Thêm repositories**:
   - Docker repository
   - Kubernetes repository

4. **Cài đặt packages**:
   - `containerd.io`
   - `kubelet`, `kubeadm`, `kubectl` (version được chỉ định)

5. **Hold packages**: Giữ các package K8s ở version hiện tại để tránh tự động update

6. **Khởi động kubelet**: Enable và start service kubelet

## ⚙️ Yêu cầu hệ thống

- **OS**: Ubuntu 22.04 (hoặc tương thích)
- **Cloud-init**: Đã được cấu hình
- **SSH Access**: Public key của AWX phải có trong `authorized_keys` của target hosts
- **Privileges**: Cần quyền sudo/root để thực thi playbook
- **Python**: Python 3 (được chỉ định trong `group_vars/all.yml`)

## 🔧 Cấu hình

### Biến mặc định (group_vars/all.yml)

```yaml
ansible_python_interpreter: /usr/bin/python3
pod_network_cidr: "10.244.0.0/16"
k8s_version: "1.27.*"
```

### Các biến có thể override

- **`k8s_version`**: Version của Kubernetes (mặc định: `1.27.*`)
- **`pod_network_cidr`**: CIDR cho pod network (mặc định: `10.244.0.0/16`)

Bạn có thể override các biến này khi chạy playbook:

```bash
ansible-playbook k8s-node.yml -e "k8s_version=1.28.*"
```

## 🚀 Cách sử dụng

### Chạy trực tiếp với Ansible

```bash
ansible-playbook k8s-node.yml -i inventory.yml
```

### Sử dụng với AWX

Playbook này được thiết kế để chạy thông qua AWX với Kafka-driven workflow:

1. AWX nhận job request qua Kafka
2. Playbook được thực thi trên target hosts
3. Join command cho node sẽ được inject vào facts thông qua Kafka consumer
4. Node sẵn sàng để join vào cluster (master đã được init trước đó)

## 📝 Lưu ý quan trọng

- **Master node**: Master node phải được init trước khi chạy playbook này trên worker nodes
- **Join command**: Lệnh join thực tế được lấy từ Redis/key-value store do master node đẩy xuống, không phải từ playbook này
- **Network**: Đảm bảo các node có thể truy cập internet để download packages
- **Firewall**: Cần mở các port cần thiết cho Kubernetes communication

## 🔗 Tích hợp

Playbook này được tích hợp với:
- **AWX**: Automation platform để quản lý và thực thi playbooks
- **Kafka**: Message broker để điều phối jobs
- **Redis**: Key-value store để lưu join commands từ master node

## 📄 License

Xem file LICENSE (nếu có) hoặc liên hệ maintainer để biết thêm chi tiết.
