+++
title = "Giới thiệu các tính năng mới trong Docker 1.12"
date = "2016-06-26T16:56:43+02:00"
tags = ["Docker", "DockerCon2016", "DevOps"]
categories = ["DevOps"]
menu = ""
images = ["static/images/dockerswarm01.png", "static/images/visualizer01.png", "static/images/visualizer02.png", "static/images/visualizer03.png"]
banner = "banners/dockercon2016.png"
+++

DockerCon đã giới thiệu một loạt các tính năng mới trong Docker, và với version 1.12 (hiện tại đang rc2) chúng ta đã có thể bắt đầu vọc phá. Hôm nay tôi xin giới thiệu về [Docker Swarm Mode](https://docs.docker.com/engine/swarm/), một tính năng mới có trong Docker Engine (v1.12).

Nếu bạn đang sử dụng các phiên bản trước v1.12.0-rc1, vui lòng xem thêm [ở đây](https://docs.docker.com/swarm/).


## Docker Swarm Mode là gì?
Trong phiên bản v1.12.0, Docker Swarm là một tính năng được tích hợp sẵn trong Docker Engine, và chúng ta có thể xây dựng một swarm cluster, tạo các service trong cluster một cách dễ dàng mà không phải cài thêm bất kỳ phần mềm nào. Đặc biệt, version này cũng bao gồm việc xử lý về vấn đề Security, Networking, State & Cluster Initialization.

Xem thêm ở đây để được giới thiệu về [Docker Swarm Mode](https://www.youtube.com/watch?v=KC4Ad1DS8xU)

## Cài đặt
Trong bài này, chúng ta sẽ tạo ra một swarm cluster gồm 1 cluster manager và 3 worker. Nếu như trước đây chúng ta cần thêm các container khác cho service discovery, load balancer, ... để xây dựng một cluster thì bây giờ mấy thứ đó không cần nữa. Tất cả đã được built-in trong Docker Engine. 

![Environment Setup]
(/images/dockerswarm01.png)

### Mở port để giao tiếp giữa các hosts
- **TCP port 2377**: port này để cluster mananegement
- **TCP** và **UDP port 7946** để giao tiếp giữa các nodes
- **TCP** và **UDP port 4789** dành cho overlay network

Nếu chúng ta sử dụng Boot2Docker phiên bản mới nhất thì nó đã làm sẵn cho chúng ta rồi, nice :)

### Dùng docker-machine để tạo các máy ảo
Lưu ý là chúng ta phải cài Docker Toolbox để có thể xài lệnh docker-machine nhé. Trước đây tôi cũng đã từng nhầm lẫn giữa Docker for Mac & Docker Toolbox, lưu ý đây là 2 sản phẩm hoàn toàn khác nhau nhé.

Chúng ta tạo ra các máy ảo như sau, lưu ý là chúng ta phải sử dụng bản Boot2Docker mới nhất nhé, nếu không chắc thì bạn nên chạy lệnh upgrade để tải bảng boot2docker mới nhất:

* **swarm-00**: máy này sẽ làm cluster manager, IP Address sẽ là: 192.168.99.100
* **swarm-01**: worker số 1
* **swarm-02**: worker số 2
* **swarm-03**: worker số 3

```
$ docker-machine upgrade default
$ docker-machine create -d virtualbox swarm-00
$ docker-machine create -d virtualbox swarm-01
$ docker-machine create -d virtualbox swarm-02
$ docker-machine create -d virtualbox swarm-03
```

Kiểm tra lại địa chỉ IP để chắc chắn là **swarm-00** có địa chỉ là 192.168.99.100

```
$ docker-machine ip swarm-00 swarm-01 swarm-02 swarm-03
192.168.99.103
192.168.99.102
192.168.99.101
192.168.99.100
```

## Tạo một Swarm Cluster
Bây giờ chúng ta đã chuẩn bị sẵn sàng môi trường để ta xây dựng một Swarm Cluster.

1. Kết nối vào **swarm-00** để khởi tạo swarm manager
```
$ docker-machine ssh swarm-00
$ docker swarm init --listen-addr 192.168.99.100:2377
Swarm initialized: current node (56cqz10j5z5inzzt0rsw877ja) is now a manager.
```

2. Chạy lệnh ```docker info``` để kiểm tra trạng thái của **swarm manager**
```
docker@swarm-00:~$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.12.0-rc2
Storage Driver: aufs
 Root Dir: /mnt/sda1/var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 0
 Dirperm1 Supported: true
Logging Driver: json-file
...
```

3. Chạy lệnh docker node ls để kiểm tra thông tin các node trong cluster
```
docker@swarm-00:~$ docker node ls
ID                           NAME      MEMBERSHIP  STATUS  AVAILABILITY  MANAGER STATUS
56cqz10j5z5inzzt0rsw877ja *  swarm-00  Accepted    Ready   Active        Leader
```

## Thêm các node vào Swarm Cluster
1. Thêm node **swarm-01** vào swarm cluster
```
$ docker-machine ssh swarm-01
docker@swarm-01:~$ docker swarm join 192.168.99.100:2377
This node joined a Swarm as a worker.
```

2. Thêm node **swarm-02** vào swarm cluster
```
$ docker-machine ssh swarm-02
docker@swarm-02:~$ docker swarm join 192.168.99.100:2377
This node joined a Swarm as a worker.
```

3. Thêm node **swarm-03** vào swarm cluster
```
$ docker-machine ssh swarm-03
docker@swarm-03:~$ docker swarm join 192.168.99.100:2377
This node joined a Swarm as a worker.
```

4. Kiểm tra lại trạng thái của các nodes trong cluster
```
docker@swarm-00:~$ docker node ls
ID                           NAME      MEMBERSHIP  STATUS  AVAILABILITY  MANAGER STATUS
0h70dwoxxl5jwx53m1g857jf2    swarm-03  Accepted    Ready   Active        
2csi90lknok2o5w3539xm48dl    swarm-02  Accepted    Ready   Active        
56cqz10j5z5inzzt0rsw877ja *  swarm-00  Accepted    Ready   Active        Leader
5dkujue686ddhm0zui9un77rs    swarm-01  Accepted    Ready   Active 
```

## Dùng Visualizer để hiển thị trạng thái các nodes & services
Thật tình là tôi rất thích dùng thằng Visualizer này để hiển thị trạng thái của các nodes & services. Bạn có thể không cần bước này, nhưng mình khuyên là nên dùng.

Bạn có thể tham khảo về Visualzier [ở đây](https://github.com/ManoMarks/docker-swarm-visualizer).
```
docker@swarm-00:~$ docker run -it -d -p 5000:5000 -e HOST=localhost -e PORT=5000 -v /var/run/docker.sock:/var/run/docker.sock manomarks/visualizer
```

Sau khi chạy container xong, chúng ta có thể xem trên browser http://localhost:5000 để xem trạng thái các nodes. 

## Deploy một service lên swarm cluster
Đơn giản nhất, chúng ta tạo một server chạy trên swarm cluster như sau:
```
$ docker-machine ssh swarm-00
$ docker service create --replicas 1 --name helloworld alpine ping docker.com
```

* ```docker service create``` để tạo ra service
* ```--replicas``` chỉ định số instance muốn có, nếu bất kỳ instance nào bị down, cluster sẽ tự động clone số instance bằng số lượng replicas đã cấu hình
* ```--name``` tên service là helloworld
* ```alpine ping docker.com``` image tên là alpine và chạy lệnh ```ping docker.com```

Sau đó mở trình duyệt và truy cập vào http://localhost:5000 chúng ta sẽ thấy một container như trong hình.

![Container in visualizer](/images/visualizer01.png)

Liệt kê danh sách các service trong swarm cluster
```
$ docker service ls
708jyxdzqrim  helloworld  1/1       alpine  ping docker.com
```

Xem chi tiết thông tin service
```
$ docker service tasks helloworld
ID                         NAME          SERVICE     IMAGE   LAST STATE             DESIRED STATE  NODE
439ypow987o6h0iun9wlm7x1f  helloworld.1  helloworld  alpine  Running About an hour  Running        swarm-00
```

## Scale service trong một cluster manager
Ví dụ ta muốn scale out service **helloworld** lên 5 instances, dùng lệnh sau:
```
docker@swarm-00:~$ docker service scale helloworld=5
helloworld scaled to 5
```

Chúng ta xem lại danh sách service để kiểm tra xem trạng thái các service
```
docker@swarm-00:~$ docker service tasks helloworld
ID                         NAME          SERVICE     IMAGE   LAST STATE             DESIRED STATE  NODE
439ypow987o6h0iun9wlm7x1f  helloworld.1  helloworld  alpine  Running About an hour  Running        swarm-00
39zig0tgnb1fh0f6cocmtgq2c  helloworld.2  helloworld  alpine  Running 2 minutes      Running        swarm-00
2p9sslerww0pjd3brn52y3zp3  helloworld.3  helloworld  alpine  Running 2 minutes      Running        swarm-02
9m4p7jsaj0tv3k02ty099cjhd  helloworld.4  helloworld  alpine  Running 2 minutes      Running        swarm-00
26oj2s6xgnr4wo2zu2zig73uq  helloworld.5  helloworld  alpine  Running 2 minutes      Running        swarm-02
```

Hoặc có thể xem qua visualizer http://localhost:5000

![Visualizer Viewer](/images/visualizer02.png)

Như trên hình, chúng ta thấy node **swarm-02** có 2 instances, chúng ta thử stop node này xem chuyện gì sẽ xảy ra.
```
$ docker-machine stop swarm-02
Stopping "swarm-02"...
Machine "swarm-02" was stopped.
```

Chúng ta sẽ thấy swarm cluster sẽ tự động tạo ra thêm 3 nodes trên swarm-00. Tôi cũng không hiểu sao nó không tạo ra trên **swarm-01**.
![Visualizer Viewer](/images/visualizer03.png)

## Kết luận
Đây là một vài tính năng rất cơ bản mà chúng ta lướt sơ qua, còn rất nhiều tính năng thú vị chúng ta sẽ khám phá dần qua các bài viết tiếp theo.