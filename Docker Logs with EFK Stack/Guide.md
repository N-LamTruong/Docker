# Tập trung logs Docker sử dụng EFK Stack

![Docker Logs with EFK](https://user-images.githubusercontent.com/97789851/165694689-bca684a7-4c1e-4def-8168-ac23a22c3baf.jpeg)

## Giới thiệu

Khi bạn đưa các containers Docker vào sử dụng, bạn sẽ thấy nhu cầu thu thập logs ngày càng cần thiết. Để duy trì nhận logs ở một nơi nào đó dễ sử dụng, phân tích hơn containers. **Fluentd** là một công cụ thu thập dữ liệu mã nguồn mở được thiết kế để thống nhất cơ sở hạ tầng ghi logs của bạn. Nó tập hợp các kỹ sư vận hành, kỹ sư ứng dụng và kỹ sư dữ liệu lại với nhau bằng cách làm cho việc thu thập và lưu trữ logs trở nên đơn giản và có thể mở rộng.

**Fluentd** giúp bạn dễ dàng thu thập logs và định tuyến chúng đến một nơi khác, chẳng hạn như trong hướng dẫn này là Elasticsearch, để bạn có thể phân tích dữ liệu. Và sau đó chúng được chuyển tới **Kibana**, một giao diện thân thiện với bạn hơn.

### Thiết lập cần thiết khi bắt đầu

Để hoàn thành hướng dẫn này, bạn sẽ cần những thứ sau:

* Sử dụng Ubuntu server và có quyền root
* Docker được cài đặt trên máy chủ của bạn bằng cách làm theo Cách cài đặt và sử dụng Docker trên Ubuntu
  
## Bước 1: Xây dựng image Fluentd
Tạo một thư mục mới cho các tài nguyên Fluentd Docker của bạn và chuyển vào đó:
    
    sudo mkdir Fluent-Aggregator-Docker && cd Fluent-Aggregator-Docker