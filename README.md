# Openlane-Ubuntu_26.04

---

---

# 📘 CẨM NANG TỐI THƯỢNG: CÀI ĐẶT & SỬ DỤNG OPENLANE TRÊN UBUNTU 26.04 (ARM64)

> **💡 Bí quyết cốt lõi:** Tài liệu này cung cấp toàn bộ vòng đời sử dụng môi trường thiết kế vi mạch OpenLane tối ưu nhất cho máy ảo Ubuntu 26.04 trên chip Apple Silicon/ARM64. Mọi quy trình đều sử dụng 100% sức mạnh của Docker để môi trường luôn sạch, mượt mà và tránh triệt để lỗi biên dịch C++ (`libparse`/`klayout`) khi cài qua `pip`.

## 📑 Mục lục
- [Phương pháp 1: Cài đặt Tiêu chuẩn (Kéo qua mạng - Ưu tiên)](#-phương-pháp-1-cài-đặt-tiêu-chuẩn-kéo-qua-mạng---ưu-tiên)
- [Phương pháp 2: Cài đặt Offline (Tải từ Google Drive)](#-phương-pháp-2-cài-đặt-offline-tải-từ-google-drive)
- [Hướng dẫn "Xóa sổ" OpenLane, Docker & File rác](#-hướng-dẫn-xóa-sổ-openlane-docker--file-sao-lưu)
- [Dành cho nhà phát triển: Tự tạo bản sao lưu](#-dành-cho-nhà-phát-triển-tự-tạo-bản-sao-lưu)

---

## 🌐 PHƯƠNG PHÁP 1: CÀI ĐẶT TIÊU CHUẨN (KÉO QUA MẠNG - ƯU TIÊN)

Đây là phương pháp chuẩn nhất để luôn có được phiên bản mới nhất từ kho lưu trữ của dự án.

### 1. Chuẩn bị hệ thống và cài đặt Docker
Ubuntu 26.04 thường gặp xung đột file kết nối Docker. Chuỗi lệnh dưới đây sẽ cài đặt và "đặc trị" lỗi này:

```bash
# Cập nhật và cài đặt công cụ
sudo apt update && sudo apt upgrade -y
sudo apt install -y git make build-essential curl wget nano util-linux-extra docker.io

# Cấp quyền User cho Docker và dọn dẹp tiến trình lỗi
sudo usermod -aG docker $USER
newgrp docker
sudo pkill -9 dockerd
sudo systemctl stop docker docker.socket
sudo rm -f /run/docker.sock /var/run/docker.sock /var/run/docker.pid

# Khởi động lại siêu sạch và cấp quyền file kết nối
sudo systemctl start docker.socket
sudo systemctl start docker
sudo systemctl enable docker
sudo chmod 666 /var/run/docker.sock

```

### 2. Tải OpenLane và Kéo bộ công cụ (Docker Image)

```bash
cd ~
git clone https://github.com/The-OpenROAD-Project/OpenLane.git
cd OpenLane
make pull-openlane

```

### 3. Cấp quyền, Tải thư viện PDK và Chạy thử

```bash
# Tạo thư mục và cấp quyền để tải thư viện không bị lỗi Permission Denied
mkdir -p ~/.ciel
sudo chown -R $USER:$USER ~/.ciel

# Tải thư viện SkyWater 130nm và chạy bài test
make pdk
make test

```

> **🎉 Thành quả:** Nếu hệ thống chạy mượt mà và kết thúc bằng chữ **`Successful!`**, cỗ máy của bạn đã sẵn sàng 100%.

---

## 🚀 PHƯƠNG PHÁP 2: CÀI ĐẶT OFFLINE (TẢI TỪ GOOGLE DRIVE)

Nếu mạng của bạn thiếu ổn định hoặc tải từ GitHub/Docker quá chậm, hãy dùng bộ cài đặt đóng gói sẵn này.

### Bước 1: Tải bộ công cụ Offline

Tải 2 file dưới đây và lưu trực tiếp vào thư mục Home (`~/`) của máy ảo:

1. 🐳 **Docker Image OpenLane:** [Tải openlane_docker_backup.tar tại đây](https://drive.google.com/file/d/1IMMXw-lQ8O42SJ5j_st3sQYXGQzzJJoJ/view?usp=sharing) *(~500MB)*
2. 🗂️ **Mã nguồn & PDK:** (https://drive.google.com/file/d/1kdi7o3El-TNeL4f99qCRx4HmDMmwGvgP/view?usp=sharing) *(~1.5GB)*

### Bước 2: Cài đặt nền tảng Docker

Hãy chạy toàn bộ chuỗi lệnh cài đặt Docker ở **[Mục 1 của Phương pháp 1](https://www.google.com/search?q=%231-chu%E1%BA%A9n-b%E1%BB%8B-h%E1%BB%87-th%E1%BB%91ng-v%C3%A0-c%C3%A0i-%C4%91%E1%BA%B7t-docker)** để máy có sẵn phần mềm Docker chuẩn xác nhất.

### Bước 3: Bung nén và Chạy thử

Khi Docker đã bật và 2 file đã nằm trong `~/`, hãy chạy:

```bash
# 1. Nạp Docker Image trực tiếp từ file (Không cần internet)
docker load -i ~/openlane_docker_backup.tar

# 2. Di chuyển vào Home và bung nén Mã nguồn + PDK
cd ~
tar -xzvf openlane_source_pdk_backup.tar.gz

# 3. Chạy bài test thiết kế
cd ~/OpenLane
make test

```

---

## 🧹 HƯỚNG DẪN "XÓA SỔ" OPENLANE, DOCKER & FILE SAO LƯU

Quá trình gỡ bỏ được chia thành các cấp độ để bạn dễ kiểm soát:

### Cấp độ 1: Xóa OpenLane, PDK & File tải về (Giữ lại Docker)

```bash
# Xóa mã nguồn, thư viện và dọn rác bộ nhớ Docker
rm -rf ~/OpenLane ~/.ciel
sudo docker system prune -a --volumes -f

# Xóa các file sao lưu .tar / .tar.gz (Nếu bạn dùng Phương pháp 2)
rm -f ~/openlane_docker_backup.tar ~/openlane_source_pdk_backup.tar.gz

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

Nếu bạn muốn tự mình tạo 2 file nén `.tar` và `.tar.gz` để backup cho máy tính khác, hãy chạy chuỗi lệnh sau khi hệ thống đang hoạt động tốt:

```bash
# 1. Xem tên chính xác của Image OpenLane đang chạy
docker images

# 2. Đóng hộp Docker Image
docker save ghcr.io/the-openroad-project/openlane:ff5509f65b17bfa4068d5336495ab1718987ff69-arm64v8 -o ~/openlane_docker_backup.tar

# 3. Cấp lại quyền để tránh lỗi "Permission Denied" khi nén báo cáo ngầm
sudo chown -R $USER:$USER ~/OpenLane ~/.ciel

# 4. Di chuyển vào Home và nén thư mục
cd ~
tar -czvf openlane_source_pdk_backup.tar.gz OpenLane .ciel

```

```

```
