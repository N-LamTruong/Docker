# Docker on Linux

![image](https://user-images.githubusercontent.com/97789851/166855507-abb04309-989a-4b96-b1de-97ff8b0d50c8.png)

## Giới thiệu Docker
**Docker** là một công cụ giúp tạo ra và triển khai các **container** để phát triển, chạy ứng dụng được dễ dàng. Các container là môi trường, mà ở đó lập trình viên đưa vào các thành phần cần thiết để ứng dụng của họ chạy được. Bằng cách đóng gói ứng dụng cùng với container như vậy, nó đảm bảo ứng dụng chạy được trên nhiều hệ điều hành khác nhau (Linux, Windows, Desktop, Server,...)

**Docker** gần giống như máy ảo (nếu các bạn đã từng sử dụng qua các nền tảng ảo hóa như **Virtual Box**, **VMWare**). Nhưng điểm khác là thay vì tạo ra toàn bộ hệ thống (ảo hóa), **Docker** lại cho phép ứng dụng sử dụng nhân của hệ điều hành đang chạy Docker để chạy ứng dụng bằng cách bổ sung thêm các thành phần còn thiếu cung cấp bởi container. Cách này làm tăng hiệu xuất và giảm kích thước ứng dụng.

**=> Ai dùng Docker?** Docker mang lại lợi ích cho cả **lập trình viên** lẫn **quản trị hệ thống**!!

## Install Docker on Ubuntu
1. Chạy từng lệnh sau để cài đặt:
```console
sudo apt update
```
```console
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```
```console
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
```console
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```
```console
sudo apt update
```
```console
apt-cache policy docker-ce
```
```console
sudo apt install docker-ce -y
```
2. Sau đó kiểm tra **dịch vụ Docker** đã hoạt động hay chưa bằng lệnh:
```console
sudo systemctl status docker
```
3. Sau khi cài đặt, bạn có thể cho user hiện tại thuộc **group docker**, để khi gõ lệnh không cần xin quyền **sudo**
```console
sudo usermod -aG docker $USER
```

### => Để có hiệu lực bạn cần **Logout** user đó khỏi server. Sau đó **Login** lại.
