---
tags:
  - docker
---
# Tổng quan về Docker
Docker một nền tảng mở để phát triển và chạy các ứng dụng, Khi triển khai trên môi trường Docker ứng dụng của bạn được tách biệt với cơ sở hạ tầng giúp bạn phân phối và phát triển ứng dụng nhanh hơn. Còn đối với những người không phải là nhà phát triển thì chúng ta sẽ dùng Docker để chạy rất nhiều ứng dụng được phân phối trên nền tảng này mà không cần quan tâm đến hạ tầng đang có.

<figure markdown="span">
  ![Image title](https://www.hostinger.com/tutorials/wp-content/uploads/sites/2/2022/10/how-does-docker-work.jpg){ width="500" }
  <figcaption>Docker Platform</figcaption>
</figure>

# Các thành phần để chạy ứng dụng.
## Docker file
Dockerfile về cơ bản chỉ là một tệp văn bản (chúng ta có thể sửa đổi nó bằng bất kỳ trình soạn thảo văn bản nào), chứa tất cả các hướng dẫn mà người dùng có thể chạy trên dòng lệnh để tạo {++image++}. Nó chứa tất cả các hướng dẫn mà Docker cần để xây dựng {++image++}.

## Docker image
Các container sẽ chạy một hệ thống tập tin độc lập tách biệt được cung cấp bằng các {++image++}, trong {++image++} sẽ bao gồm mọi thứ cần thiết để chạy ứng dụng, tất cả các phụ thuộc, cấu hình, tập lệnh, nhị phân, v.v.

- Một số lệnh cơ bản về image
``` bash
docker image list # Liệt kệ danh sách image
docker image rm image-id # Xóa một image
```

## Docker container
Docker cung cấp một môi trường tách biệt để đóng gói và chạy ứng dụng được gọi là các container.

- Một số lệnh cơ bản về container
``` bash
docker run # Chạy container
docker ps # Liệt kê các container đang hoạt động
docker stop container-name # Dừng một container
docker rm container-name # Gở bỏ một container
docker exec container-name -it bash # Chạy bash từ trong container
```
<figure markdown="span">
  ![Image title](https://linuxiac.com/wp-content/uploads/2021/06/what-is-docker-container.png){ width="500" }
  <figcaption>Docker Container</figcaption>
</figure>

## Docker volume
Có thể hiểu đơn giản {++volume++} là cơ sở dữ liệu của một docker container và được docker quản lý, thông thường có hai cách để sử dụng dữ liệu của một container:
- Gắn trực tiếp một thư mục local vào container.
- Tạo và sử dụng một docker volume được quản lý bởi docker.

## Docker compose
Docker Compose là một công cụ để xác định và chạy các ứng dụng. Đây là chìa khóa để mở ra trải nghiệm phát triển, triển khai hợp lý và hiệu quả.
Compose đơn giản hóa việc kiểm soát toàn bộ ngăn xếp ứng dụng của bạn, giúp dễ dàng quản lý các dịch vụ, mạng và ổ đĩa trong một tệp cấu hình YAML duy nhất, dễ hiểu. Sau đó, với một lệnh duy nhất, bạn tạo và khởi động tất cả các dịch vụ từ tệp cấu hình của mình.

``` bash
docker compose up -d
```

``` yaml title="docker-compose.yml"
---
version: "3.9"
services:
  authelia:
    image: authelia/authelia:latest
    container_name: authelia
    volumes:
      - ./config:/config
    networks:
      - proxy
    expose:
      - 9091
    restart: unless-stopped
networks:
  proxy:
    external: true #network của Traefik Proxy
```

# Cài đặt Docker trên HĐH Ubuntu 22.04
Chúng ta sẽ không cài đặt docker có sẵn trong Ubuntu vì nó thường cũ rồi, để cài đặt bản mới nhất và dễ dàng cập nhật chúng ta sẽ thêm vào nguồn gói cài đặt Docker chính thức từ chính chủ.

Cập nhật các gói ứng dụng của Ubuntu:
``` bash
sudo apt update
```

Cài thêm một số gói yêu cầu nếu chưa có:
``` bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Thêm GPG từ nguồn của Docker:
``` bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Thêm Docker vào nguồn APT của Ubuntu:
``` bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Cập nhật lại các gói ứng dụng sau khi đã thêm Docker:
``` bash
sudo apt update
```

Cài đặt Docker:
``` bash
sudo apt install docker-ce
```

Kiểm tra trạng thái của Docker, nếu bạn thấy {++active++} (running) thì đã ok rồi:
``` bash
sudo systemctl status docker
```

Ngoài ra để chạy lệnh docker mà không cần phải chạy với quyền root, bạn phải thêm user hiện tại vào nhóm docker:
``` bash
sudo usermod -aG docker ${USER}
su - ${USER}
```

Chạy lệnh sau nếu hiện thông tin phiên bản của Docker thì bạn đã thành công:
``` bash
docker version
```

# Cài đặt Docker Compose
Cũng như phần trên để có được phiên bản tốt nhất bạn sẽ tải về và cài đặt docker compose trực tiếp từ nguồn chính chủ của Docker.

Vào trang [release](https://github.com/docker/compose/releases) để tìm và copy link phiên bản mới nhất. Hiện tại là v2.29.1.

Lệnh tải về hệ thống:
``` bash
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.29.1/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
```

Cài đặt quyền tập tin docker-compose để nó có thể thực thi:
``` bash
chmod +x ~/.docker/cli-plugins/docker-compose
```

Kiểm tra phiên bản đã cài đặt:
``` bash
docker compose version
```

