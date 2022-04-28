# Tập trung logs Docker sử dụng EFK Stack

![Docker Logs with EFK](https://user-images.githubusercontent.com/97789851/165694689-bca684a7-4c1e-4def-8168-ac23a22c3baf.jpeg)

## Giới thiệu

Khi bạn đưa các containers Docker vào sử dụng, bạn sẽ thấy nhu cầu thu thập logs rất cần thiết để giám sát hoạt động của container. Để duy trì nhận logs ở một nơi nào đó dễ sử dụng, dễ phân tích hơn ở chính containers. Thì **Fluentd** là một công cụ thu thập dữ liệu mã nguồn mở được thiết kế để thống nhất cơ sở hạ tầng ghi logs của bạn. Nó tập hợp các kỹ sư vận hành, kỹ sư ứng dụng và kỹ sư dữ liệu lại với nhau bằng cách làm cho việc thu thập và lưu trữ logs trở nên đơn giản và có thể mở rộng.

**Fluentd** giúp bạn dễ dàng thu thập logs và định tuyến chúng đến một nơi khác, chẳng hạn như trong hướng dẫn này là **Elasticsearch**, để bạn có thể phân tích dữ liệu logs. Và sau đó chúng được chuyển tới **Kibana**, một giao diện thân thiện với bạn hơn.

### Thiết lập cần thiết khi bắt đầu

**Để hoàn thành hướng dẫn này, bạn sẽ cần những thứ sau:**

* Trong hướng dẫn này sử dụng 2 server Ubuntu để thực hiện: 

    **192.168.5.30 cài EFK nhận logs** và **192.168.5.40 gửi logs**
* Sử dụng Ubuntu server và có quyền root
* Docker được cài đặt trên máy chủ của bạn bằng cách làm theo Cách cài đặt và sử dụng Docker trên Ubuntu
* Cập nhật múi giờ trên server Ubuntu:
```console
sudo timedatectl set-timezone Asia/Ho_Chi_Minh
```
  
## Bước 1: Xây dựng image Fluentd (192.168.5.30)
Tạo một thư mục mới cho các tài nguyên Fluentd Docker của bạn và chuyển vào đó:
```console
sudo mkdir Fluent-Aggregator-Docker && cd Fluent-Aggregator-Docker
```
Tạo **Dockerfile**:
```console
sudo nano Dockerfile
```
Thêm chính xác các nội dung sau vào tệp của bạn. Tệp này yêu cầu Docker cài đặt **Ruby**, **Fluentd** và **plugin kết nối Elasticsearch** kèm 1 số lệnh thực thi khi khởi tạo container:
```console
FROM ruby:2.6.6
RUN apt-get update
RUN gem install fluentd -v "~>1.14.0"
RUN mkdir /etc/fluent
RUN apt-get install libcurl4-gnutls-dev -y
RUN /usr/local/bin/gem install fluent-plugin-elasticsearch
RUN /usr/local/bin/gem install fluent-plugin-secure-forward
ADD ./fluent.conf /etc/fluent/
ENTRYPOINT ["/usr/local/bundle/bin/fluentd", "-c", "/etc/fluent/fluent.conf"]
```
Bạn cũng cần tạo một tệp **fluent.conf** trong cùng thư mục đó:
```console
sudo nano fluent.conf
```
Tệp **fluent.conf** sẽ config như sau. Bạn có thể sao chép chính xác tệp này:
```console
<match **.*>
  @type copy
  <store>
    @type elasticsearch
    host 192.168.5.30
    port 9200
    index_name ${tag}
    type_name ${tag}
    enable_ilm true
    include_timestamp true
    flush_interval 5s
  </store>
  <store>
    @type stdout
  </store>
</match>

<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>
```
**-> Cùng phân tích cú pháp cấu hình:**

File config Fluentd được chia làm 2 phần chính là **match** và **source**. Trong đó:

**match** là đầu ra, định tuyến cũng như chỉ đường cho logs biết chúng được vận chuyển tới đâu
* <**.*> giúp fluentd nhận tất cả các "index - chỉ mục theo tag bên phía server fluentd gửi logs"
* @type, host, port: Gửi logs đến Elasticsearch, địa chỉ IP của server chứa Elastic (hướng dẫn này thì bộ thu thập logs EFK cài chung trên 1 server), mặc định Elastic hoạt động trên port 9200
* index_name, type_name, enable_ilm: Nhận chỉ mục, tên,... theo tag từ bên server Fluentd gửi logs đến
* include_timestamp, flush_interval: Khi tạo chỉ mục trên Kibana sẽ có giao diện, hình ảnh trực quan, dễ hiểu. Thời gian làm mới nhận logs và gửi logs
* @type stdout: Plugin output stdout in các sự kiện đầu ra theo tiêu chuẩn (hoặc logs nếu được khởi chạy dưới dạng daemon). Plugin đầu ra này rất hữu ích cho mục đích gỡ lỗi.

**source** là đầu vào, chỉ cho fluentd biết nhận logs từ những nguồn nào
*   @type forward, port, bind: Lắng nghe logs gửi đến mặc định ở port 24224 và bind để nhận từ tất cả các nguồn

Sau đó, xây dựng image Docker của bạn, có thể đặt tên image là **fluentd-aggregator**:
```console
docker build -t fluentd-aggregator .
```
Quá trình này sẽ mất vài phút để hoàn thành. Kiểm tra xem bạn đã tạo thành công các image chưa:
```console
docker image ls
```
Bạn sẽ thấy đầu ra như thế này:

    REPOSITORY           TAG       IMAGE ID       CREATED          SIZE
    fluentd-aggregator   latest    858465f09f14   17 seconds ago   898MB
    ruby                 2.6.6     6d86b0beade7   13 months ago    840MB

## Bước 2: Khởi động container Elasticsearch (192.168.5.30)
Bây giờ, hãy quay lại thư mục gốc hoặc thư mục ưa thích của bạn cho container **Elasticsearch** và trong hướng dẫn này sẽ sử dụng **version 8.1.1**:
```console
cd && docker pull elasticsearch:8.1.1
```
Trước khi khởi động container Elasticsearch, chúng ta cần tăng giá trị **max_map_count** trên máy chủ Docker của bạn vì image này nhanh hơn so với việc tự cấu hình
```console
sudo sysctl -w vm.max_map_count=262144
```
Sau khi tải image Elasticsearch xuống thành công, hãy **khởi chạy** container cùng tham số và sử dụng biến trong container Elasticsearch
```console
docker run -d --name elasticsearch -p 9200:9200 -v /etc/localtime:/etc/localtime:ro -e "ES_JAVA_OPTS=-Xms128m -Xmx128m" -e "discovery.type=single-node" -e "xpack.security.enabled=false" elasticsearch:8.1.1
```
**-> Giải thích 1 chút cho các bạn hiểu :))**
* Mặc định Elasticsearch sử dụng port 9200
* -v /etc/localtime:/etc/localtime:ro ánh xạ múi giờ trên server vào container
* ES_JAVA_OPTS=-Xms128m -Xmx128m cấu hình JVM thường để bằng 1/3 dung lượng RAM server
* discovery.type=single-node hiện tại chỉ sử dụng 1 node Elasticsearch
* xpack.security.enabled=false tắt gói bảo mật của Elasticsearch, hướng dẫn này chưa dùng đến tính năng xác thực account để login

Tiếp theo, hãy đảm bảo rằng **container Elasticsearch** đang chạy bình thường bằng cách kiểm tra các container đang hoạt động:
```console
docker ps
```
Bạn sẽ thấy đầu ra như thế này:

    CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                                 NAMES
    577c7db077d8   elasticsearch:8.1.1   "/bin/tini -- /usr/l…"   33 minutes ago   Up 33 minutes   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 9300/tcp   elasticsearch
Check web

## Bước 3: Khởi động container Kibana kết nối Elasticsearch (192.168.5.30)
**Chú ý: Version của Kibana và Elasticsearch phải cùng nhau**

Tải **Kibana version 8.1.1** về server
```console
docker pull kibana:8.1.1
```
Sau khi tải image Kibana xuống thành công, hãy **khởi chạy** container cùng tham số và sử dụng một số biến trong container Kibana
```console
docker run -d --name kibana -p 5601:5601 -v /etc/localtime:/etc/localtime:ro -e "ELASTICSEARCH_URL=http://192.168.5.30:9200/" -e "ELASTICSEARCH_HOSTS=http://192.168.5.30:9200/" kibana:8.1.1
```

## Bước 3 - Khởi động container Fluentd-Aggregator liên kết Elasticsearch (192.168.5.30)