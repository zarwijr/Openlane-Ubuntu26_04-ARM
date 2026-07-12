# Openlane-Ubuntu_26.04

---

# 📘 CẨM NANG TỐI THƯỢNG: CÀI ĐẶT & SỬ DỤNG OPENLANE TRÊN UBUNTU 26.04 (ARM64)

> **💡 Bí quyết cốt lõi:** Tài liệu này cung cấp toàn bộ vòng đời sử dụng môi trường thiết kế vi mạch OpenLane tối ưu nhất cho máy ảo Ubuntu 26.04 trên chip Apple Silicon/ARM64. Mọi quy trình đều sử dụng 100% sức mạnh của Docker để môi trường luôn sạch, mượt mà và tránh triệt để lỗi biên dịch C++ (`libparse`/`klayout`) khi cài qua `pip`.

## 📑 Mục lục
- [Phương pháp 1: Cài đặt cực nhanh (Tải từ Google Drive - Khuyên dùng)](#phương-pháp-1-cài-đặt-cực-nhanh-tải-từ-google-drive---khuyên-dùng)
- [Phương pháp 2: Cài đặt từ đầu (Dành cho mạng mạnh)](#phương-pháp-2-cài-đặt-từ-đầu-dành-cho-mạng-mạnh)
- [Hướng dẫn "Xóa sổ" OpenLane & Docker](#-hướng-dẫn-xóa-sổ-openlane--docker)
- [Dành cho nhà phát triển: Tự tạo bản sao lưu](#-dành-cho-nhà-phát-triển-tự-tạo-bản-sao-lưu)

---

## 🚀 PHƯƠNG PHÁP 1: CÀI ĐẶT CỰC NHANH (TẢI TỪ GOOGLE DRIVE - KHUYÊN DÙNG)

Nếu bạn không muốn mất hàng giờ để tải dữ liệu, hoặc thường xuyên gặp lỗi mạng/link die trong quá trình cài đặt, mình đã đóng gói sẵn toàn bộ môi trường và thư viện (cập nhật mới nhất) cho chip ARM64. 

### Bước 1: Tải bộ công cụ Offline
Hãy tải 2 file dưới đây và lưu trực tiếp vào thư mục Home (`~/`) của máy ảo Ubuntu:

1. 🐳 **Docker Image OpenLane (Bản Offline):** [Tải openlane_docker_backup.tar tại đây]([[DÁN LINK DRIVE CỦA BẠN VÀO ĐÂY]](https://drive.google.com/file/d/1IMMXw-lQ8O42SJ5j_st3sQYXGQzzJJoJ/view?usp=sharing) *(Nặng khoảng ~500MB+)*
2. 🗂️ **Mã nguồn & Thư viện SkyWater 130nm PDK:** [Tải openlane_source_pdk_backup.tar.gz tại đây]([[DÁN LINK DRIVE CỦA BẠN VÀO ĐÂY]](https://drive.google.com/file/d/1kdi7o3El-TNeL4f99qCRx4HmDMmwGvgP/view?usp=sharing) *(Nặng khoảng ~1.4GB+)*

### Bước 2: Cài đặt Docker chuẩn xác cho Ubuntu 26.04
Mở Terminal và chạy tuần tự các lệnh sau để cài đặt và cấu hình Docker không bị lỗi giao tiếp (`docker.sock`):

```bash
# 1. Cập nhật và cài đặt Docker
sudo apt update
sudo apt install -y docker.io

# 2. Cấp quyền User cho Docker và dọn dẹp các tiến trình lỗi
sudo usermod -aG docker $USER
newgrp docker
sudo pkill -9 dockerd
sudo systemctl stop docker docker.socket
sudo rm -f /run/docker.sock /var/run/docker.sock /var/run/docker.pid

# 3. Khởi động lại siêu sạch và cấp quyền file kết nối
sudo systemctl start docker.socket
sudo systemctl start docker
sudo systemctl enable docker
sudo chmod 666 /var/run/docker.sock

```

### Bước 3: Bung nén và Chạy thử nghiệm

Khi Docker đã sẵn sàng và 2 file tải từ Drive đã nằm trong thư mục Home (`~/`), chạy các lệnh sau:

```bash
# 1. Nạp thẳng Docker Image từ file nén (Không cần mạng)
docker load -i ~/openlane_docker_backup.tar

# 2. Di chuyển vào thư mục Home và bung nén Mã nguồn + PDK
cd ~
tar -xzvf openlane_source_pdk_backup.tar.gz

# 3. Chạy bài test thiết kế mạch mặc định (spm - Multiplier)
cd ~/OpenLane
make test

```

> **🎉 Thành quả:** KLayout và OpenROAD sẽ chạy mượt mà qua các bước Synthesis, Routing... Nếu dòng cuối cùng hiện chữ **`Successful!`**, cỗ máy của bạn đã sẵn sàng 100%. (Bạn có thể dùng lệnh `rm -f ~/openlane_*.tar*` để xóa 2 file cài đặt lấy lại dung lượng).

---

## 🌐 PHƯƠNG PHÁP 2: CÀI ĐẶT TỪ ĐẦU (DÀNH CHO MẠNG MẠNH)

Dành cho những ai muốn tự clone trực tiếp từ GitHub gốc và kéo image mới nhất từ server.

### 1. Chuẩn bị hệ thống và cài đặt Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git make build-essential curl wget nano util-linux-extra docker.io

sudo usermod -aG docker $USER
newgrp docker
sudo pkill -9 dockerd
sudo systemctl stop docker docker.socket
sudo rm -f /run/docker.sock /var/run/docker.sock /var/run/docker.pid
sudo systemctl start docker.socket
sudo systemctl start docker
sudo systemctl enable docker
sudo chmod 666 /var/run/docker.sock

```

### 2. Tải OpenLane và công cụ

```bash
cd ~
git clone [https://github.com/The-OpenROAD-Project/OpenLane.git](https://github.com/The-OpenROAD-Project/OpenLane.git)
cd OpenLane
make pull-openlane

```

### 3. Cấp quyền và tải thư viện PDK (SkyWater 130nm)

```bash
mkdir -p ~/.ciel
sudo chown -R $USER:$USER ~/.ciel
make pdk
make test

```

---

## 🧹 HƯỚNG DẪN "XÓA SỔ" OPENLANE & DOCKER

Quá trình gỡ bỏ được chia thành 2 cấp độ:

### Cấp độ 1: Chỉ xóa OpenLane & PDK (Giữ lại Docker)

```bash
# Xóa mã nguồn, thư viện Ciel và dọn dẹp Docker
rm -rf ~/OpenLane ~/.ciel
sudo docker system prune -a --volumes -f

```

### Cấp độ 2: Xóa tận gốc Docker (Về trạng thái nguyên thủy)

```bash
sudo systemctl stop docker docker.socket
sudo apt purge -y docker.io containerd runc
sudo apt autoremove -y && sudo apt clean
sudo rm -rf /var/lib/docker /etc/docker
sudo rm -rf /run/docker.sock /var/run/docker.sock /var/run/docker.pid
sudo groupdel docker

```

---

## 🛡️ DÀNH CHO NHÀ PHÁT TRIỂN: TỰ TẠO BẢN SAO LƯU

Nếu bạn muốn tự mình tạo 2 file nén `.tar` và `.tar.gz` để backup cho máy tính khác, hãy chạy chuỗi lệnh sau khi máy đang hoạt động tốt:

```bash
# 1. Xem tên chính xác của Image OpenLane đang chạy
docker images

# 2. Đóng hộp Docker Image (Đổi tên image nếu cần)
docker save ghcr.io/the-openroad-project/openlane:ff5509f65b17bfa4068d5336495ab1718987ff69-arm64v8 -o ~/openlane_docker_backup.tar

# 3. Cấp lại quyền để tránh lỗi "Permission Denied" khi nén báo cáo ngầm
sudo chown -R $USER:$USER ~/OpenLane ~/.ciel

# 4. Di chuyển vào Home và nén thư mục (Tránh lỗi lồng đường dẫn)
cd ~
tar -czvf openlane_source_pdk_backup.tar.gz OpenLane .ciel

```

```


```
