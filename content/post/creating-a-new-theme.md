+++
title = "Mở port cho Docker Machine"
date = "2016-06-28T11:20:46+02:00"
tags = ["Docker", "Docker Machine"]
categories = ["DevOps"]
menu = ""
banner = "banners/portforward.png"
+++

## Giới thiệu

Khi sử dụng Docker Machine trên MacOS, chúng ta thường chạy các containers, sau đó mở port cho các containers đó, thông thường chúng ta sẽ truy cập vào các containers đó thông qua địa chỉ của docker machine (virtual machine) thông qua IP Address.

Ví dụ: 
```
docker run -it -p 8080:8080 --name dotnetapp microsoft/dotnet:latest
```

Để truy cập vào website, chúng ta sẽ dùng địa chỉ ```http://192.168.99.100:8080``` với **192.168.99.100** chính là địa chỉ máy ảo (mặc định) trên Docker Machine.

Để có thể truy cập bằng địa chỉ localhost, chúng ta phải forward port từ máy ảo sang máy host. Từ đó chúng ta có thể truy cập bằng địa chỉ ```http://localhost:8080```

Có nhiều cách để forward port, nhưng tôi thấy bạn Johan Haleby làm là cách dễ nhớ nhất. Đó là dùng tool **pf**

#### Download bash script
```
git clone https://github.com/johanhaleby/docker-machine-port-forwarder && cd docker-machine-port-forwarder
```

#### Mở port 8080 cho máy default
```
./pf 8080
```

#### Hoặc mở port 8080 cho một máy docker machine có tên là **swarm-00**
```
./pf 8080 -e swarm-00
```

#### Hoặc là forward port 8080 sang port 80 trên máy host
```
./pf 80:8080 -e swarm-00
```

#### Stop port forwarding
```
./pf 8080 -s
```

#### Chạy ở chế độ foreground
```
./pf 8080 -f
```