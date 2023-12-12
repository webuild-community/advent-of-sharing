---
title: A clumsy Docker networking notes
author: kien (ntk148v)
date: 2023-12-22
---

> Tiếp tục series về Docker networking

Nguồn:

- <https://github.com/ntk148v/til/tree/master/docker/networking>
- <https://argus-sec.com/docker-networking-behind-the-scenes/>
- <https://docs.docker.com/network/>
- <https://github.com/KamranAzeem/learn-docker/blob/master/docs/docker-networking-deep-dive.md>
- <https://docs.docker.com/engine/tutorials/networkingcontainers/>
- <https://docs.docker.com/network/iptables/>
- <https://dev.to/polarbit/how-docker-container-networking-works-mimic-it-using-linux-network-namespaces-9mj>

Mục lục:

- [1. Khái niệm](#1-khái-niệm)
- [2. Docker và network namespace](#2-docker-và-network-namespace)
- [3. Tản mạn qua Docker networking subsystem](#3-tản-mạn-qua-docker-networking-subsystem)
  - [3.1. docker0 - default bridge network](#31-docker0---default-bridge-network)
  - [3.2. User-defined bridge network](#32-user-defined-bridge-network)
- [4. Một số command hay ho](#4-một-số-command-hay-ho)

## 1. Khái niệm

- Các khái niệm:

  - [Network namespace](https://man7.org/linux/man-pages/man8/ip-netns.8.html):

    - Trước khi đi vào network namespace, cần hiểu về **Namespace**. Namespace là một tính năng của Linux kernel để phân vùng tài nguyên của kernel sao cho một tập hợp các process chỉ nhìn thấy tập hợp các tài nguyên (trong namespace đó) trong khi một tập hợp các process khác nhìn thấy tập các tài nguyên khác (trong namespace khác). Tính năng này hoạt động bằng cách đưa tập các tài nguyên và process vào cùng một namespace, các namespace khác nhau sẽ tham chiếu đến các tài nguyên riêng biệt. Tài nguyên có thể tồn tại trong nhiều không gian. Ví dụ về các tài nguyên như vậy là process ID, hostname, user ID, tên file và một số tên (name) liên quan đến quyền truy cập mạng và giao tiếp giữa các process.
    - Networking namespace là một trong các loại namespace. Mỗi network namespace có [network stack](https://en.wikipedia.org/wiki/Network_stack) riêng: địa chỉ IP, network interface, route table, ...
    - Bạn có thể xem thao tác các network namespace bằng câu lệnh sau:

    ```shell
    $ sudo ip netns <sub-command>
    ```

  - [Network bridge](https://wiki.archlinux.org/index.php/Network_bridge): một thiết bị mạng ảo chuyển tiếp các gói giữa hai hoặc nhiều phân đoạn mạng.
  - **Veth pair**: hiểu đơn giản là virtual network cable nối giữa 2 network interface device (NIC).

- Về cơ bản, Docker sử dụng các tính năng trên như sau:

![](https://res.cloudinary.com/practicaldev/image/fetch/s--tGvJLH6y--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/pszchqfvcjpl13dextrc.png)

## 2. Docker và network namespace

> Ví dụ sau lấy theo các thông số, cấu hình mặc định khi chạy Docker container.

- Khi tạo và chạy 1 container, Docker đồng thời tạo ra một network namespace riêng cho container đó. Sau đó, Docker kết nối network của container đó với Linux bridge **docker0** bằng veth pair.
- Như ở trên đã nói, để xem danh sách network namespace, gõ câu lệnh sau:

```shell
$ sudo ip netns list
# Start a container
$ docker run --name nstest -it busybox

# Open another terminal session
$ sudo ip netns list
# nothing return
```

- Hmm, không có gì trả về. Mặc định Docker không thêm container network namespace vào trong Linux runtime data, bạn có thể tạo softlink cho network namespace đó:

```shell
# Get container process id.
$ pid="$(docker inspect -f '{{.State.Pid}}' nstest)"
# Soft link the network namespace
$ sudo mkdir -p /var/run/netns/
$ sudo ln -sf /proc/$pid/ns/net /var/run/netns/nstest
$ sudo ip netns list
nstest (id: 0)
# ^ this, it works.
```

- Chạy vào network namespace, như đã nói, mỗi network namespace có network stack riêng:

```shell
$ docker exec nstest ip a
docker exec nstest ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

$ sudo ip netns exec nstest ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
# ^ result is the same as docker exec command.
$ ip a | grep docker
5: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
8: veth4866e91@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
```

- Ở đây, eth0@if8 trong container namespace đang kết nối với interface veth4866e91@if7 trên docker0 bridge.

```shell
$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242f0c749bd       no              veth4866e91 # this
lxdbr0          8000.00163e2afe62       no
```

## 3. Tản mạn qua Docker networking subsystem

Kiến trúc Docker networking subsystem dạng pluggable, có thể mở rộng sử dụng drivers. Mặc định, có những drivers sau: `bridge`, `host`, `none`, `overlay`, `macvlan`, `ipvlan`.

Trong bài viết này, chủ yếu nói đến `bridge` driver.

### 3.1. docker0 - default bridge network

![](https://docs.docker.com/engine/tutorials/bridge1.png)

- Mặc định, tất cả container đều kết nối với docker0.
- Docker engine khi start up sẽ tìm một subnet không được sử dụng (bắt đầu trong một danh sách fixed trong code) và gán IP đầu tiên cho docker0. Như trong trường hợp của tui, đó là `172.17.0.1/16`.
- Nếu không có containers nào chạy và kết nối với docker0, trạng thái của docker0 là DOWN.
- Các containers kết nối với docker0, kế thừa cấu hình DNS của host: `/etc/hosts` and `/etc/resolv.conf`.
- Lưu ý, ở docker0 bridge, không có tính năng **service discovery**, do vậy, các containers phải giao tiếp thông qua địa chỉ IP.

```shell
# Start a second container
$ docker run --name nstest2 -it busybox
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
9: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # ping nstest
ping: bad address 'nstest'
/ # ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.263 ms
^C
--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.263/0.263/0.263 ms
```

### 3.2. User-defined bridge network

- Người dùng có thể tự tạo Docker network.

```shell
$ docker network create mynet
ed7da300a506e6e9be68b8f69d1b857dd7cc8db2a9d51c4b85dba37976780293

$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
935865833d99   bridge    bridge    local
f753aeafa1a1   host      host      local
ed7da300a506   mynet     bridge    local
6b0488ceccc8   none      null      local

$ docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "ed7da300a506e6e9be68b8f69d1b857dd7cc8db2a9d51c4b85dba37976780293",
        "Created": "2023-12-12T09:37:34.358888862+07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

# Check with ip addr show, it should show up as a network
# inteface on the host with the name br-<Network ID substring>
$ ip addr show
# ...
11: br-ed7da300a506: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:f8:36:9b:e0 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-ed7da300a506
       valid_lft forever preferred_lft forever

$ brctl show
bridge name     bridge id               STP enabled     interfaces
br-ed7da300a506 8000.0242f8369be0       no
docker0         8000.0242f0c749bd       no
lxdbr0          8000.00163e2afe62       no
```

- Không giống docker0 bridge, network mới có DNS-based service discovery:
  - Ở đây, sử dụng custom image `praqma/network-multitool` để có đủ commands và tools, có thể sử dụng image bất kỳ và cài đặt tool tương tự.

```shell
$ docker run --name=web --network=mynet -d wbitt/network-multitool
e2d841b7a02916aa25a2fa5ceace0427ad7c0ccc967ad01fb5cc423460f0e990

$ docker run --name=db --network=mynet -e MYSQL_ROOT_PASSWORD=secret -d mysql
492ba8a9277d96876f0a605d9434723aed774978a40023e778a97815216754f0

$ docker ps
CONTAINER ID   IMAGE                     COMMAND                  CREATED              STATUS              PORTS                                  NAMES
e2d841b7a029   wbitt/network-multitool   "/bin/sh /docker/ent…"   15 seconds ago       Up 14 seconds       80/tcp, 443/tcp, 1180/tcp, 11443/tcp   web
492ba8a9277d   mysql                     "docker-entrypoint.s…"   About a minute ago   Up About a minute   3306/tcp, 33060/tcp                    db

# Ping each other
$ docker exec -it web bash
e2d841b7a029:/# ping -c 1 db
PING db (172.18.0.2) 56(84) bytes of data.
64 bytes from db.mynet (172.18.0.2): icmp_seq=1 ttl=64 time=0.096 ms

--- db ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.096/0.096/0.096/0.000 ms
e2d841b7a029:/# dig db

; <<>> DiG 9.18.16 <<>> db
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57615
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;db.                            IN      A

;; ANSWER SECTION:
db.                     600     IN      A       172.18.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Tue Dec 12 03:18:39 UTC 2023
;; MSG SIZE  rcvd: 38

e2d841b7a029:/#
```

- Tại sao có thể thao tác phân giải tên container như vậy? Đó là nhờ một **DNS nhúng** ở trong Docker service.

![](https://github.com/KamranAzeem/learn-docker/raw/master/docs/images/docker-service-discovery-2.png)

```shell
e2d841b7a029:/# cat /etc/resolv.conf
nameserver 127.0.0.11
options edns0 trust-ad ndots:0
e2d841b7a029:/# netstat -ntlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.11:45541        0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      1/nginx: master pro
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1/nginx: master pro
udp        0      0 127.0.0.11:42758        0.0.0.0:*                           -
```

- `/etc/resolv.conf` cho thấy container đang trỏ về nameserver 127.0.0.11:53 nhưng rõ ràng không có tiến trình nào lắng nghe port 53 cả. Vậy DNS này hoạt động như thế nào?
- Tiếp tục, tạo thêm 1 container khác, nhưng có nhiều **quyền** hơn.
  - `iptables` chỉ chạy khi có capabilities `-cap-add=NET_ADMIN and --cap-add=NET_RAW`.

```shell
$ docker run \
    --name tool \
    --network mynet \
    --cap-add=NET_ADMIN \
    --cap-add=NET_RAW \
    -it wbitt/network-multitool /bin/bash
ccd72fe225c8:/# dig -c db
;; Warning, ignoring invalid class db

; <<>> DiG 9.18.16 <<>> -c db
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17778
;; flags: qr rd ra; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;.                              IN      NS

;; ANSWER SECTION:
.                       31133   IN      NS      d.root-servers.net.
.                       31133   IN      NS      l.root-servers.net.
.                       31133   IN      NS      k.root-servers.net.
.                       31133   IN      NS      i.root-servers.net.
.                       31133   IN      NS      j.root-servers.net.
.                       31133   IN      NS      e.root-servers.net.
.                       31133   IN      NS      h.root-servers.net.
.                       31133   IN      NS      g.root-servers.net.
.                       31133   IN      NS      a.root-servers.net.
.                       31133   IN      NS      f.root-servers.net.
.                       31133   IN      NS      c.root-servers.net.
.                       31133   IN      NS      b.root-servers.net.
.                       31133   IN      NS      m.root-servers.net.

;; Query time: 8 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Tue Dec 12 03:20:07 UTC 2023
;; MSG SIZE  rcvd: 239

ccd72fe225c8:/# netstat -ntlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.11:39527        0.0.0.0:*               LISTEN      -
udp        0      0 127.0.0.11:35031        0.0.0.0:*                           -
ccd72fe225c8:/# iptables-nft-save
# Generated by iptables-nft-save v1.8.9 (nf_tables) on Tue Dec 12 03:20:27 2023
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:DOCKER_OUTPUT - [0:0]
:DOCKER_POSTROUTING - [0:0]
-A OUTPUT -d 127.0.0.11/32 -j DOCKER_OUTPUT
-A POSTROUTING -d 127.0.0.11/32 -j DOCKER_POSTROUTING
# Queries for DNS:
-A DOCKER_OUTPUT -d 127.0.0.11/32 -p tcp -m tcp --dport 53 -j DNAT --to-destination 127.0.0.11:39527
-A DOCKER_OUTPUT -d 127.0.0.11/32 -p udp -m udp --dport 53 -j DNAT --to-destination 127.0.0.11:35031
# Response from DNS:
-A DOCKER_POSTROUTING -s 127.0.0.11/32 -p tcp -m tcp --sport 39527 -j SNAT --to-source :53
-A DOCKER_POSTROUTING -s 127.0.0.11/32 -p udp -m udp --sport 35031 -j SNAT --to-source :53
COMMIT
# Completed on Tue Dec 12 03:20:27 2023
```

- **iptables** does magic thing! Mọi DNS query đến `127.0.0.11:53` thực tế được điều hướng sang `127.0.0.11:39527` (TCP) và `127.0.0.11:35031` (UDP).
  - Thông thường, DNS sử dụng UDP cho queries nhỏ hơn 512 bytes, trên 512 bytes -> TCP.

![](https://github.com/KamranAzeem/learn-docker/raw/master/docs/images/docker-service-discovery-1.png)

> **Lưu ý**: Port trên hình có thể khác với thực tế.

- Chi tiết về implementation có thể tham khảo ở đây:
  - [container_operations_unix.go](https://github.com/moby/moby/blob/65973c6c40a32380f4f7318de92eb6c8161742f0/daemon/container_operations_unix.go#L398)
  - [resolver_unix.go](https://github.com/moby/moby/blob/65973c6c40a32380f4f7318de92eb6c8161742f0/libnetwork/resolver_unix.go#L13)

```go
	case container.HostConfig.NetworkMode.IsUserDefined():
		// The container uses a user-defined network. We use the embedded DNS
		// server for container name resolution and to act as a DNS forwarder
		// for external DNS resolution.
		// We parse the DNS server(s) that are defined in /etc/resolv.conf on
		// the host, which may be a local DNS server (for example, if DNSMasq or
		// systemd-resolvd are in use). The embedded DNS server forwards DNS
		// resolution to the DNS server configured on the host, which in itself
		// may act as a forwarder for external DNS servers.
		// If systemd-resolvd is used, the "upstream" DNS servers can be found in
		// /run/systemd/resolve/resolv.conf. We do not query those DNS servers
		// directly, as they can be dynamically reconfigured.
		*sboxOptions = append(
			*sboxOptions,
			libnetwork.OptionOriginResolvConfPath("/etc/resolv.conf"),
		)
	default:
```

## 4. Một số command hay ho

- Join container vào process namespace của container khác.
  - Có thể thấy process mysqld của container db.

```shell
$ docker run --name busybox \
                 --pid container:db \
                 --rm -it busybox /bin/sh
/ # ps aux
PID   USER     TIME  COMMAND
    1 999       0:25 mysqld
  237 root      0:00 /bin/sh
  243 root      0:00 ps aux
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
48: eth0@if49: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

- Join thêm network namespace.
  - Địa chỉ IP đang là 172.18.0.2/16 (db network namespace).

```shell
docker run --name busybox \
                 --pid container:db \
                 --network container:db --rm -it busybox /bin/sh
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
38: eth0@if39: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # ps aux
PID   USER     TIME  COMMAND
    1 999       0:26 mysqld
  245 root      0:00 /bin/sh
  252 root      0:00 ps aux
/ #
```
