# Docker on Linux

![image](https://user-images.githubusercontent.com/97789851/166855507-abb04309-989a-4b96-b1de-97ff8b0d50c8.png)

## Giới thiệu Docker
**Docker** là một công cụ giúp tạo ra và triển khai các **container** để phát triển, chạy ứng dụng được dễ dàng. Các container là môi trường, mà ở đó lập trình viên đưa vào các thành phần cần thiết để ứng dụng của họ chạy được. Bằng cách đóng gói ứng dụng cùng với container như vậy, nó đảm bảo ứng dụng chạy được trên nhiều hệ điều hành khác nhau (Linux, Windows, Desktop, Server,...)

**Docker** gần giống như máy ảo (nếu các bạn đã từng sử dụng qua các nền tảng ảo hóa như **Virtual Box**, **VMWare**). Nhưng điểm khác là thay vì tạo ra toàn bộ hệ thống (ảo hóa), **Docker** lại cho phép ứng dụng sử dụng nhân của hệ điều hành đang chạy Docker để chạy ứng dụng bằng cách bổ sung thêm các thành phần còn thiếu cung cấp bởi container. Cách này làm tăng hiệu xuất và giảm kích thước ứng dụng

**Docker compose** là công cụ dùng để định nghĩa và run multi-container cho Docker application

**=> Ai dùng Docker?** Docker mang lại lợi ích cho cả **lập trình viên** lẫn **quản trị hệ thống**!!

[**Script cài đặt docker và docker-compose**](/install_docker.sh) bản release mới nhất nếu các bạn không muốn thực hiện từng bước

## Install Docker on Ubuntu (step by step)

**Lưu ý: Áp dụng từ bản 20.04 trở lên**

1. Update hệ thống:
    ```console
    sudo apt update
    ```
2. Cài đặt các gói phụ thuộc:
    ```console
    sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
    ```
3. Add key docker:
    ```console
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```
4. Add repo docker:
    ```console
    echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
5. Update lại hệ thống nhận repo docker mới:
    ```console
    sudo apt update
    ```
6. Check các version docker có thể cài đặt trong hệ thống:
    ```console
    apt-cache policy docker-ce
    ```
7. Cài đặt mặc định version docker
    ```console
    sudo apt install docker-ce -y
    sudo systemctl start docker
    sudo systemctl enable docker
    ```
8. Sau đó kiểm tra **dịch vụ Docker** đã hoạt động hay chưa bằng lệnh:
    ```console
    sudo systemctl status docker
    ```
9. Sau khi cài đặt, bạn có thể cho user hiện tại thuộc **group docker**, để khi gõ lệnh không cần xin quyền **sudo** (nếu user có quyền sudo thì bỏ qua bước này)
    ```console
    sudo usermod -aG docker $USER
    ```
    Để bước 9 có hiệu lực bạn cần **Logout** user đó khỏi server. Sau đó **Login** lại.

## Install Docker Compose
Truy cập **Github docker compose** để cài đặt phiên bản như mong muốn: [Click here](https://github.com/docker/compose/releases)

Hướng dẫn này mình sẽ sử dụng phiên bản **v2.24.0-birthday.10 🥳**

1. Download file cài sẵn docker-compose:
    ```console
    wget https://github.com/docker/compose/releases/download/v2.24.0-birthday.10/docker-compose-linux-x86_64
    ```
2. Đổi tên file để dễ thao tác khi sử dụng:
    ```console
    mv docker-compose-linux-x86_64 docker-compose
    ```
3. Thêm quyền thực thi cho docker-compose và di chuyển vào /usr/local/bin sẽ giúp thao tác gọi được từ mọi nơi trong hệ thống:
    ```console
    chmod +x docker-compose && mv docker-compose /usr/local/bin/
    ```
4. Check lại phiên bản docker-compose:
    ```console
    docker-compose -v
    ```