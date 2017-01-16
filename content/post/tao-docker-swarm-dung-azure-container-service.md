+++
title = "Tạo Docker Swarm Cluster dùng Azure Container Service"
date = "2016-08-23T16:34:00"
tags = ["Docker", "Azure", "AzureContainerService", "DockerSwarm"]
categories = ["DevOps", "Cloud", "Azure"]
menu = ""
banner = "banners/azurecontainerservice.jpg"
+++

## Giới thiệu
Docker đã quá quen thuộc với anh em làm DevOps và ngày càng trở nên phổ biến với cả developers. Build ra các images, tạo ra các containers gần như là các công việc thường ngày của dân IT. Hôm nay tôi xin giới thiệu cách dùng **Azure Container Service** để tạo ra một **Swarm Cluster** chỉ với vài bước đơn giản. Có điều lưu ý ở đây là Docker Swarm chứ không phải Docker Swarm-Mode (chỉ có từ Docker v1.12 nhé).

## Chuẩn bị

- Bạn cần có account của Azure
- Bạn cần cài đặt Docker trên máy tính của mình, có thể là **Docker for Mac** hoặc **Docker for Windows** tùy hệ điều hành đang sử dụng.

## Tạo ssh key
Azure Container Service yêu cầu có ssh public key để tạo service. Do đó, chúng ta cần phải tạo ra key này để nhập vào lúc đăng ký Azure Container Service. Key này sẽ được dùng để đăng nhập mà không cần phải gõ password.

Để tạo key, chúng ta mở terminal lên, sau đó gõ lệnh ```ssh-keygen```: 
```
$ ssh-keygen          
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/phihuynh/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/phihuynh/.ssh/id_rsa.
Your public key has been saved in /Users/phihuynh/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ZR+bg7xg4SKjU8yiQTMGvDS+soVi19fFB1GeWbU+Tzc phihuynh@phis-mbp.harveynash.vn.local
The key's randomart image is:
+---[RSA 2048]----+
|o          .o. .o|
|.+         .. + .|
|o=o     . + o+ . |
|oooo   . = = =.  |
|..o B . S + *  Eo|
|+=.= + + o . . .=|
|=o+   .   .     .|
|.  .             |
|                 |
+----[SHA256]-----+
```

Sau đó, chúng ta dùng lệnh ```cat ~/.ssh/id_rsa.pub``` để xem giá trị của public key. Chúng ta lưu lại giá trị này để nhập vào trong bước đăng ký **Azure Containre Service**.
```
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC/KIrbbnrP2X60Qr65dGIU3c0qDZP4bQpuFA71xbKBp45Rr4HPdQ4GBEnBf1oryNKBDOlhQCor7mIDn4hlwCvCGkiIP4IZEMDeSnraHlDUuyWd+CR1K+pi8OwP4DlSajEsikZ+zPQ8wr/aD94bs4yjEANclXGxQSGZMXdCdoizI9Qw3PZmd/xYLvtUatW1FOD9Jxt8+QHkAIroImviWUy6Wr7YAk9RhaMuD+yTvI9QETtquQhUOZNP8KD8jCPE3IgJIW2uWW5AFMkCIm54NDyMx1sVGuMb3/Vnxq2wjo+9if0DwGHNOsBfiq/zrTeSJ/9DjyCy8qFI7YcRkia4IzKt phihuynh@phis-mbp.harveynash.vn.local

```
## Tạo Azure Container Service
Sau khi đã tạo xong ssh public key, bước tiếp theo là đăng ký một Azure Container Service để tạo ra một Swarm Cluster. Azure Container Service có thể tạo ra một Swarm Cluster hay Mesos Master.

- Đăng nhập vào **[Azure Portal](https://portal.azure.com)** 
- Từ portal, chọn **New**, gõ vào **Azure Container Service**, sau đó nhấn vào nút **Create**
![Create container service](/images/acs1.png)
- Sau đó, nhập vào **User name**, **SSH public key** chính là giá trị của public key bạn tạo ra ở trên. Tôi dùng **Visual Studio Premium with MSDN** subscription, nếu bạn dùng trial hoặc mua thì có thể là subscription khác. Đặt tên cho service là **test-swarm**, chọn Location là Southeast Asia và sau đó nhấn nút **OK**.
![Basic configuration](/images/acs2.png)
- Ở bước **2 Framework configuration**, chọn **Ochestrator configuration** là **Swarm** sau đó nhấn OK. Do bài này chúng ta tập trung tạo Docker Swarm Cluster, Azure cũng hỗ trợ Mesos Cluster.
- Ở bước **3 Azure Container Service settings**, nhập vào **Agent count** là 3 (chúng ta muốn có 3 nodes, bạn có thể nhập số khác tùy ý). **Master count** là 1 là đủ, **DNS prefix** nhập vào text tùy ý, ví dụ **swarm**.
![3 Azure container service settings](/images/acs3.png)
- Bước 4 validation passed, nhấn **OK**
- Bước 5 Buy, nhấn **Purchase**

Toàn bộ quá trình deployment diễn ra mất khoảng 15-20 phút, chúng ta kiên nhẫn chờ đợi. Sau khi deploy thành công, chúng ta có thể xem các service trong resource group **test-swarm**. Chúng ta thấy có rất nhiều service được tạo ra: 1 virtual machine, 1 virtual machine scaleset (bao gồm 3 instances tương ứng với 3 nodes chúng ta cấu hình ở trên), 2 load balancers, 2 public ips, ...

![Azure container services](/images/acs4.png)

## Kết nối đến Swarm Master
Sau khi đã hoàn tất quá trình deployment, chúng ta đã sẵn sàng để kết nối đến cluster. Để kết nối đến cluster, chúng ta cần địa chỉ IP để SSH. chúng ta chọn resource group **test-swarm**, chọn **swarm-master-lb-[ID]** load balancer, nhấn **Overview** và copy giá trị Public IP Address, ví dụ của tôi là **52.187.69.179**.
![Get load balancer IP Address](/images/acs5.png)

Sau đó mở terminal, gõ lệnh ssh để kết nối đến swarm cluster:
```
$ ssh -L 2375:localhost:2375 -f -N phihuynh@52.187.69.179 -p 2200
```

Lệnh này sẽ tạo ra một SSH Server trên máy local và lắng nghe trên port 2375, sau đó forward traffic lên SSH server của cluster.

Sau đó, chúng ta chạy tiếp lệnh sau ```export DOCKER_HOST=:2375``` để lệnh docker client sẽ chuyển hướng sang localhost:2375

Ok, đến đây là chúng ta đã hoàn tất kết nối đến swarm cluster. Để kiểm tra xem kết nối thành công hay không, chúng ta chạy lệnh ```docker info``` để xem số lượng nodes có phải là 3 hay không.

```
$ docker info
ocker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 3
 swarm-agent-BA631D94000000: 10.0.0.4:2375
  └ Status: Healthy
  └ Containers: 0
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 7.145 GiB
  └ Labels: executiondriver=, kernelversion=3.19.0-65-generic, operatingsystem=Ubuntu 14.04.4 LTS, storagedriver=aufs
  └ Error: (none)
  └ UpdatedAt: 2016-08-23T11:43:31Z
 swarm-agent-BA631D94000002: 10.0.0.6:2375
  └ Status: Healthy
  └ Containers: 0
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 7.145 GiB
  └ Labels: executiondriver=, kernelversion=3.19.0-65-generic, operatingsystem=Ubuntu 14.04.4 LTS, storagedriver=aufs
  └ Error: (none)
  └ UpdatedAt: 2016-08-23T11:43:34Z
 swarm-agent-BA631D94000003: 10.0.0.7:2375
  └ Status: Healthy
  └ Containers: 0
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 7.145 GiB
  └ Labels: executiondriver=, kernelversion=3.19.0-65-generic, operatingsystem=Ubuntu 14.04.4 LTS, storagedriver=aufs
  └ Error: (none)
  └ UpdatedAt: 2016-08-23T11:43:54Z
Plugins:
 Volume: 
 Network: 
Swarm: 
 NodeID: 
 Is Manager: false
 Node Address: 
Security Options:
Kernel Version: 3.19.0-65-generic
Operating System: linux
Architecture: amd64
CPUs: 6
Total Memory: 21.44 GiB
Name: 7434ac726bcc
Docker Root Dir: 
Debug Mode (client): false
Debug Mode (server): false
WARNING: No kernel memory limit support
```

## Chạy thử một container

Sau khi kết nối thành công, chúng ta chạy thử một container ubuntu.

```
$ docker run -it ubuntu bash
```