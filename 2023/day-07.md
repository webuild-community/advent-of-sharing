---
title: Network packet journey in Docker bridge
author: kien (ntk148v)
date: 2023-12-07
---

Trong phạm vi của tài liệu này, chúng ta sẽ chỉ nói đến bridge mặc định của Docker: `docker0`, không tính đến các user-defined bridge network. Như đã nói ở trên, Docker daemon sử dụng iptables để kiểm soát, điều hướng kết nối ra/vào các containers.

Để dễ hình dung, lấy một ví dụ như sau, chúng ta có một webserver chạy container:

- Network bridge và expose port 8000 trên host server.
- Host server có ip 10.0.10.26, docker0 sub-network 172.17.0.0/16.

![](https://raw.githubusercontent.com/ntk148v/til/master/docker/networking/images/image1.png)

## 1. Luồng packet với kết nối từ bên ngoài vào trong container

Các chain `DOCKER`, `DOCKER-USER` và `DOCKER-ISOLATION-STAGE-*` đều do Docker daemon khởi tạo, có thể kiểm tra bằng các lệnh sau:

```shell
iptables -t nat -S
iptables -S
```

Sau đây là flow di chuyển:

- Gửi request từ host bên ngoài đến endpoint 10.0.10.26:8000
- (1) `-t nat -A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER`
  - Packet chuyển đến chain DOCKER (nat)
- (2) -`t nat -A DOCKER ! -i docker0 -p tcp -m tcp --dport 8000 -j DNAT --to-destination 172.17.0.2:8000`
  - DNAT, chuyển destination ip thành ip dải bridge 172.17.0.2
- (3) `-t filter -A FORWARD -j DOCKER-USER`
  - Packet đến chain FOWARD (filter), được chuyển đến chain DOCKER-USER (filter). Tại đây, packet đi qua các rules được người dùng định nghĩa, nếu không có rule DROP/REJECT nào thì sẽ trả về chain FOWARD (filter) (4) để thực hiện rule tiếp theo của chain FORWARD.
- (5) `-t filter -A FORWARD -j DOCKER-ISOLATION-STAGE-1`
  - Tương tự như chain DOCKER-USER (filter), không có rule nào thì trả về chain FOWARD (filter) (6). Nếu có rule match thì chuyển sang DOCKER-ISOLATION-STAGE-2.
- (7) `-t filter -A FORWARD -o docker0 -j DOCKER`
  - Packet chuyển đến chain DOCKER (filter), output docker0. Docker daemon tạo các rule dựa theo expose ports config của container, và tự động thêm vào chain DOCKER (filter).
- (8) `-t filter -A DOCKER -d 172.17.0.2/32 ! -i docker0 -o docker0 -p tcp -m tcp --dport 8000 -j ACCEPT`
  - Tại chain DOCKEr (filter), packet match rule, được accept.
- (9) Packet đi đến container để xử lý.
- (10) Ở chiều ra, packet đi ra chain OUTPUT -> chain POSTROUTING (nat) để thực hiện SNAT.

![](https://raw.githubusercontent.com/ntk148v/til/master/docker/networking/images/image2.png)

## 2. Luồng packet với kết nối giữa các container trong cùng network bridge

Khởi tạo thêm một container nữa, cùng bridge default docker0 với container webserver hiện có. Từ container mới, thực hiện gửi request đến endpoint 10.0.10.26:8000 (dĩ nhiên trong cùng một network, container mới có thể gọi trực tiếp đến 172.17.0.2:8000 của container webserver, tuy nhiên chúng ta không bàn tới trường hợp đó). Luồng packet như sau:

- Packet đi theo default route về gateway docker0.
- Dựa theo rule iptables MASQUERADE, source host và destination port của packet thành 172.17.0.1 và 8000 (do tại ví dụ, chúng ta để exposed port giống port trong container, đều là 8000. Trong trường hợp port trong container là 80, destination port của container đổi thành 80):

```shell
iptables -t nat -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
```

- Packet đi đến container webserver.

![](https://raw.githubusercontent.com/ntk148v/til/master/docker/networking/images/image3.png)

## 3. Luồng packet với kết nối giữa container sử dụng network bridge khác nhau

Đối với trường hợp giao tiếp giữa các container thuộc các dải bridge network khác nhau, request đến endpoint 10.0.10.26:8000 khi đến container webserver sẽ có source 172.17.0.1. Đường đi của packet như sau:

![](https://raw.githubusercontent.com/ntk148v/til/master/docker/networking/images/image4.png)
