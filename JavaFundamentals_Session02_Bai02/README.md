# Báo cáo Bài 2: Khởi tạo User thường và thiết lập đặc quyền quản trị (Sudoers Configuration)

## 1. Mục tiêu
- Tạo một tài khoản người dùng thường (`devops`) trên máy chủ Ubuntu Server nhằm tuân thủ nguyên tắc đặc quyền tối thiểu (**Least Privilege**), hạn chế tối đa rủi ro khi thao tác trực tiếp bằng tài khoản `root`.
- Cấp đặc quyền quản trị an toàn cho user `devops` thông qua nhóm `sudo`.
- Sao chép cấu hình SSH Key từ `root` sang thư mục home của `devops` để cho phép đăng nhập qua SSH an toàn bằng SSH Key.
- Thiết lập phân quyền bảo mật theo chuẩn OpenSSH cho thư mục `.ssh` và file `authorized_keys`.

---

## 2. Các bước thực hiện trên VPS (với quyền root)

### Bước 1: Khởi tạo tài khoản `devops`
Chạy lệnh tạo user mới với thư mục home và shell mặc định:
```bash
adduser devops
```
*Hệ thống yêu cầu nhập mật khẩu cho user `devops` và xác nhận thông tin cơ bản.*

### Bước 2: Cấp quyền quản trị (gán vào nhóm `sudo`)
```bash
usermod -aG sudo devops
```
*Lệnh này thêm user `devops` vào nhóm `sudo` để có thể thực thi các lệnh quản trị thông qua `sudo`.*

### Bước 3: Sao chép SSH Key và phân quyền bảo mật
Sử dụng công cụ `rsync` với tùy chọn lưu thuộc tính và cập nhật ownership:
```bash
rsync --archive --chown=devops:devops ~/.ssh /home/devops/
```

Đảm bảo phân quyền bảo mật theo tiêu chuẩn OpenSSH:
```bash
chmod 700 /home/devops/.ssh
chmod 600 /home/devops/.ssh/authorized_keys
```

Kiểm tra lại quyền sở hữu và phân quyền thư mục `.ssh`:
```bash
ls -la /home/devops/.ssh
```

---

## 3. Nhật ký kiểm tra (Terminal Log)

### 3.1. Đăng nhập từ máy cá nhân bằng tài khoản `devops`
Từ terminal máy local:
```bash
$ ssh devops@221.121.3.55
# Hoặc với private key chỉ định:
# ssh -i ~/.ssh/id_ed25519 devops@221.121.3.55

Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-xx-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Thu Oct  1 18:40:00 2026 from 14.161.x.x
devops@ubuntu-server:~$
```
=> **Kết quả:** Đăng nhập thành công bằng SSH Key vào user `devops` mà không cần nhập mật khẩu hệ thống.

### 3.2. Kiểm tra đặc quyền sudo
Thực thi lệnh kiểm tra danh tính quyền quản trị:
```bash
devops@ubuntu-server:~$ sudo whoami
[sudo] password for devops: ********
root
```
=> **Kết quả:** Lệnh trả về `root`, xác nhận user `devops` đã có quyền thực thi tác vụ quản trị an toàn qua `sudo`.

---

## 4. Kết luận
- Đã cấu hình thành công tài khoản `devops` có quyền `sudo`.
- Đã thiết lập SSH Key và phân quyền thư mục chính xác (`700` cho `.ssh` và `600` cho `authorized_keys`).
- Đáp ứng đầy đủ các tiêu chuẩn bảo mật hệ thống Linux trong môi trường thực tế.
