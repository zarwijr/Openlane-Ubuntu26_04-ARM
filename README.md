# Openlane-Ubuntu_26.04
---
Dưới đây là bản **Cẩm Nang Tối Thượng (README.md)** đã được mình tinh chỉnh lại cấu trúc, cập nhật bản vá lỗi phân quyền (chown) vào Phần 3, dọn dẹp sạch sẽ tàn dư `~/.volare` cũ và đưa phần Phụ lục vào đúng chuẩn Markdown để hiển thị tuyệt đẹp trên GitHub.

Bạn chỉ cần nhấn nút Copy ở góc phải khối code bên dưới và dán thẳng vào kho lưu trữ (Repository) của mình là xong!

---

Bạn nói rất chuẩn, phần đầu và phần `git clone` ở Giai đoạn 3 bị dính một lỗi định dạng nhỏ (đường link GitHub bị chèn thêm ngoặc vuông `[...]` làm hỏng lệnh copy của Bash), ngoài ra các dấu backtick (```) bao quanh bị thừa khiến file hiển thị không được đẹp.

Mình đã gọt giũa lại toàn bộ, sửa lỗi link, và căn chỉnh lại các khoảng trắng để file **README.md** này đạt độ hoàn hảo 100% khi đưa lên GitHub.

Bạn chỉ cần bấm nút **Copy** ở góc trên cùng bên phải của khung đen dưới đây và dán thẳng vào file trên GitHub nhé:

```markdown
# 📘 CẨM NANG TỐI THƯỢNG: CÀI ĐẶT, GỠ BỎ & SAO LƯU OPENLANE TRÊN UBUNTU 26.04 (ARM64)

> **💡 Bí quyết cốt lõi:** Tuyệt đối **không dùng `pip`** để cài đặt trực tiếp OpenLane nhằm tránh lỗi biên dịch C++ (`libparse`/`klayout`) trên chip ARM. Sử dụng 100% sức mạnh của Docker để môi trường luôn sạch, mượt mà và dễ kiểm soát.

Tài liệu này cung cấp toàn bộ vòng đời sử dụng OpenLane trên môi trường Ubuntu 26.04 (đặc biệt tối ưu cho máy ảo trên Apple Silicon/ARM64). Từ việc xây mới, đập đi, cho đến sao lưu phòng hờ đứt cáp mạng.

## 📑 Mục lục
- [Phần 1: Hướng Dẫn Cài Đặt OpenLane (Docker) Siêu Sạch](#phần-1--hướng-dẫn-cài-đặt-openlane-docker-siêu-sạch)
- [Phần 2: Hướng Dẫn "Xóa Sổ" OpenLane & Docker](#phần-2--hướng-dẫn-xóa-sổ-openlane--docker)
- [Phần 3: Chiến Lược Sao Lưu & Cài Đặt Offline](#phần-3-️-chiến-lược-sao-lưu-và-cài-đặt-offline-chống-đứt-mạng)
- [Phụ lục: Dọn Dẹp File Sao Lưu](#-phụ-lục-xóa-các-file-sao-lưu-offline-backup)

---

## PHẦN 1: 🚀 HƯỚNG DẪN CÀI ĐẶT OPENLANE (DOCKER) SIÊU SẠCH

### Giai đoạn 1: Chuẩn bị Hệ thống Cơ bản
Dọn dẹp và cài đặt các công cụ nền tảng cần thiết nhất, bỏ qua các thư viện đồ họa rườm rà.

```bash
# 1. Cập nhật toàn bộ danh sách phần mềm hệ thống
sudo apt update && sudo apt upgrade -y

# 2. Cài đặt các công cụ kéo file và biên dịch cơ bản
sudo apt install -y git make build-essential curl wget nano util-linux-extra

```

### Giai đoạn 2: Cài đặt và "Đặc trị" lỗi Docker

Ubuntu 26.04 thường gặp xung đột file giao tiếp (`docker.sock`). Các lệnh dưới đây sẽ dọn dẹp và ép Docker chạy chuẩn xác.

```bash
# 1. Cài đặt Docker Engine
sudo apt install -y docker.io

# 2. Thêm tài khoản của bạn vào nhóm đặc quyền Docker
sudo usermod -aG docker $USER
newgrp docker

# 3. Dọn dẹp các tiến trình treo và file rác gây kẹt Docker
sudo pkill -9 dockerd
sudo systemctl stop docker docker.socket
sudo rm -f /run/docker.sock /var/run/docker.sock /var/run/docker.pid

# 4. Khởi động lại Docker một cách sạch sẽ
sudo systemctl start docker.socket
sudo systemctl start docker
sudo systemctl enable docker

# 5. Cấp quyền đọc/ghi tuyệt đối cho file kết nối (Giúp gõ lệnh không cần sudo)
sudo chmod 666 /var/run/docker.sock

# 6. Kiểm tra lại kết nối (Màn hình hiện các cột CONTAINER ID là thành công)
docker ps

```

### Giai đoạn 3: Tải và Tích hợp OpenLane

Sau khi Docker đã thông suốt, tiến hành kéo mã nguồn và công cụ.

```bash
# 1. Quay về thư mục gốc và tải mã nguồn OpenLane từ GitHub
cd ~
git clone https://github.com/The-OpenROAD-Project/OpenLane.git

# 2. Di chuyển vào thư mục dự án
cd OpenLane

# 3. Ép Docker tải bản Image chứa sẵn mọi công cụ EDA (KLayout, Yosys, OpenROAD...)
make pull-openlane

```

### Giai đoạn 4: Cấp quyền, Cài PDK và Chạy thử

Ngăn chặn lỗi `Permission denied` khi tải thư viện lõi, sau đó nạp đạn và chạy thử.

```bash
# 1. Tạo trước thư mục PDK và cấp quyền để user thoải mái tải file mà không bị chặn
mkdir -p ~/.ciel
sudo chown -R $USER:$USER ~/.ciel

# 2. Tự động tải thư viện công nghệ vật lý SkyWater 130nm
make pdk

# 3. Chạy bài test thiết kế mạch mặc định (spm - Multiplier)
make test

```

> **🎉 Thành quả:** Chỉ cần đợi hệ thống chạy qua các bước Tổng hợp, Đi dây... Nếu dòng cuối cùng hiện chữ **`Successful!`**, cỗ máy thiết kế vi mạch của bạn đã sẵn sàng.

### 💡 Mẹo Hệ thống (Khuyên dùng)

Tắt chế độ ngủ để tránh máy ảo tự khóa màn hình và ngắt kết nối khi đang chạy render chip.

```bash
# Tắt khóa màn hình khi Sleep/Suspend
gsettings set org.gnome.desktop.screensaver ubuntu-lock-on-suspend false

# Tắt tự động khóa khi để máy không (Idle)
gsettings set org.gnome.desktop.screensaver lock-enabled false

```

---

## PHẦN 2: 🧹 HƯỚNG DẪN "XÓA SỔ" OPENLANE & DOCKER

Quá trình gỡ bỏ được chia thành 3 cấp độ tùy thuộc vào mục đích dọn dẹp của bạn.

### Cấp độ 1: Xóa dự án OpenLane và Thư viện lõi (PDK)

Dùng khi bạn chỉ muốn giải phóng không gian (khoảng vài GB) nhưng **vẫn muốn giữ lại Docker** cho các việc khác.

```bash
# 1. Xóa toàn bộ mã nguồn và thư mục chạy của OpenLane
rm -rf ~/OpenLane

# 2. Xóa sạch thư mục chứa PDK SkyWater 130nm (Công cụ Ciel)
rm -rf ~/.ciel

# 3. Dọn dẹp bộ nhớ đệm, các container và image không dùng đến của Docker
sudo docker system prune -a --volumes -f

```

### Cấp độ 2: Gỡ bỏ tận gốc Docker

Dùng khi bạn muốn gỡ hoàn toàn Docker và mọi cấu hình liên quan ra khỏi hệ thống.

```bash
# 1. Dừng hoàn toàn các dịch vụ Docker đang chạy ngầm
sudo systemctl stop docker docker.socket

# 2. Gỡ cài đặt phần mềm lõi của Docker
sudo apt purge -y docker.io containerd runc

# 3. Quét và dọn sạch các tệp tin phụ thuộc tự sinh ra
sudo apt autoremove -y
sudo apt clean

# 4. Xóa vĩnh viễn các thư mục cấu hình và file hệ thống cấp thấp của Docker
sudo rm -rf /var/lib/docker /etc/docker
sudo rm -rf /run/docker.sock /var/run/docker.sock /var/run/docker.pid

# 5. Xóa nhóm phân quyền docker 
sudo groupdel docker

```

### Cấp độ 3: Trả hệ điều hành về "nguyên thủy" (Tùy chọn)

Dùng khi bạn muốn gỡ luôn các công cụ nền tảng để máy ảo hoàn toàn trống trơn.

```bash
# 1. Gỡ các công cụ hệ thống cơ bản
sudo apt purge -y git make build-essential curl wget util-linux-extra

# 2. Dọn rác lần cuối để trả lại không gian đĩa
sudo apt autoremove -y
sudo apt clean

```

---

## PHẦN 3: 🛡️ CHIẾN LƯỢC SAO LƯU VÀ CÀI ĐẶT OFFLINE (Chống đứt mạng)

Nếu một ngày server chết hoặc mất mạng, bạn vẫn có thể bung cỗ máy OpenLane ra chạy bình thường bằng các bản backup dưới đây.

### Phương án 1: Đóng gói toàn bộ mã nguồn và Docker Image

Thực hiện các lệnh này khi máy vẫn đang có mạng để tạo bản backup.

```bash
# 1. Xem tên chính xác của Image OpenLane đang chạy (Ghi nhớ mục REPOSITORY và TAG)
docker images

# 2. Đóng hộp Docker Image thành file nén .tar (thay tên image tương ứng nếu khác)
docker save ghcr.io/the-openroad-project/openlane:ff5509f65b17bfa4068d5336495ab1718987ff69-arm64v8 -o ~/openlane_docker_backup.tar

# 3. Cấp lại quyền sở hữu để tránh lỗi Permission Denied khi nén các file báo cáo ngầm
sudo chown -R $USER:$USER ~/OpenLane ~/.ciel

# 4. Nén thư mục mã nguồn và toàn bộ các thư viện PDK thành một cục duy nhất
tar -czvf ~/openlane_source_pdk_backup.tar.gz ~/OpenLane ~/.ciel

```

> **Lưu ý:** Bạn sẽ thu được 2 file backup cực kỳ quan trọng (`.tar` và `.tar.gz`) trong thư mục Home. Hãy copy chúng ra USB hoặc lưu trữ trên Cloud.

### Phương án 2: Khôi phục Offline (Khi mất mạng)

Copy 2 file backup ở trên vào máy ảo mới và thực hiện lệnh bung nén:

```bash
# 1. Nạp Docker Image trực tiếp từ file tar không cần mạng
docker load -i ~/openlane_docker_backup.tar

# 2. Giải nén mã nguồn OpenLane và thư viện PDK thẳng vào thư mục Home
tar -xzvf ~/openlane_source_pdk_backup.tar.gz -C ~/

```

---

## 🧹 Phụ lục: Xóa các file sao lưu Offline (Backup)

Dùng khi bạn không còn nhu cầu lưu trữ dự phòng và muốn lấy lại hoàn toàn dung lượng ổ cứng.

```bash
# Xóa file sao lưu của Docker Image (.tar)
rm -f ~/openlane_docker_backup.tar

# Xóa file sao lưu của mã nguồn và thư viện PDK (.tar.gz)
rm -f ~/openlane_source_pdk_backup.tar.gz

# (Tùy chọn) Lệnh gộp xóa cả 2 file cùng lúc cho nhanh:
rm -f ~/openlane_docker_backup.tar ~/openlane_source_pdk_backup.tar.gz

```

*Chỉ cần gõ lệnh `rm -f` như trên, hai "cục tạ" backup này sẽ biến mất không dấu vết ngay lập tức! Bạn hoàn toàn làm chủ từng KB dữ liệu trên máy ảo của mình.*

```

```
