<h1 style="text-align: center;">Wazuh Docker 4.4</h1>
<img src="Picture/wazuh-image.png">

## Mục tiêu:
- Triển khai Multi-node (đảm bảo tính HA)
  - 2 node Wazuh manager (1 master và 1 worker)
  - 3 node Wazuh indexer
  - 1 node Wazuh dashboard
- Scale tăng, giảm node worker Wazuh manager để chia tải cho hệ thống

**Tài liệu của nhà sản xuất: [here](https://documentation.wazuh.com/current/deployment-options/docker/index.html)**

**Lưu ý:** Phần scale trên trang chủ Wazuh hiện tại chỉ có tài liệu năm 2019 cho version 3.9 và không áp dụng được đối với hướng dẫn này <img src="Picture/chuma%20hme.jpg"  width="20" height="20">

## Điều kiện kiên quyết
- Tối thiểu 6GB RAM
- Tăng **max_map_count**
    ```console
    sysctl -w vm.max_map_count=262144
    ```
    Đặt làm giá trị vĩnh viễn, chèn xuống cuối file **/etc/sysctl.conf**:

    ```console
    vm.max_map_count=262144
    ```
    Update lại cấu hình:
    ```console
    sysctl -p
    ```
- Cài đặt **docker engine** và **docker-compose** (đối với Linux: kernel version >= 3.10 và Docker Compose >= 1.29)

## I. Triển khai Multi-node
### 1. Sao chép repo về hệ thống:
```console
git clone https://github.com/wazuh/wazuh-docker.git -b v4.4.1
```
Sau đó chuyển đến thư mục **multi-node** để thao tác

### 2. Generate self-signed certificates xác thực cho mỗi node:
```console
docker-compose -f generate-indexer-certs.yml run --rm generator
```
Lệnh trên sẽ lưu chứng chỉ vào thư mục **config/wazuh_indexer_ssl_certs**

### 3. Bắt đầu triển khai Wazuh multi-node sử dụng **docker-compose**
```console
docker-compose up -d
```
Tài khoản và mật khẩu mặc định Wazuh-dashboard: **admin** - **SecretPassword**
### 4. Thay đổi mật khẩu của tài khoản mặc định ([Click here](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html#change-the-password-of-an-existing-user))

## II. Scale worker Wazuh manager
Tăng số lượng worker lên:
1. Chỉnh sửa docker-compose file:
  - Thêm service worker wazuh 2 (edit 3 dòng cuối trong đoạn ánh xạ volume)
  - Thêm volume cho worker wazuh 2 xuống cuối
  - Sửa service nginx

2. Thêm vào folder /config/wazuh_cluster file config wazuh2_worker.conf (cp từ wazuh_worker.conf nhưng sẽ sửa dòng có chữ worker01 -> worker02)

3. Edit file /config/certs.yml (thêm wazuh2.worker)

4. Edit file /config/nginx/nginx.conf (đoạn upstream thêm server wazuh2.worker:1514)

Giảm số lượng worker xuống:
1. Chỉnh sửa docker-compose file:
  - Comment lại đoạn service worker wazuh 2
  - Comment lại đoạn thêm ở service nginx

2. Edit file /config/nginx/nginx.conf (comment lại phần server wazuh2.worker:1514)