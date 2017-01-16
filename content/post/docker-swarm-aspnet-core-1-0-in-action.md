+++
title = "Docker Swarm, aspnet core 1.0 in action"
date = "2016-06-29T11:44:08+02:00"
tags = ["asp.net core", "Docker Swarm", "Docker Compose"]
categories = ["Azure", "Cloud", "Programming", "DevOps", "Docker"]
menu = ""
banner = "banners/microsoft-loves-linux.jpg"
images = ["static/images/aspnetcore01.png"]
+++

Cuối cùng thì Microsoft cũng ra mắt [aspnet core 1.0](https://blogs.msdn.microsoft.com/webdev/2016/06/27/announcing-asp-net-core-1-0/), sau một thời gian dài trải qua các version RC1, RC2. Như các bạn cũng biết asp.net core có nhiều tính năng được built-in mà chúng ta không phải tìm kiếm & sử dụng nhiều libraries ngoài như asp.net 5. Có thể kể ra một vài tính năng như sau:

- Chạy cross-platform trên Windows, Mac và Linux
- Xây dựng trên .NET core, hỗ trợ side-by-side versioning
- Hỗ trợ built-in dependency injection
- Tag Helper kiểu tự nhiên như HTML data attributes (bắt chước Angular 1 :v )
- Có thể host trên IIS hoặc self-host
- Không còn phân biệt giữa MVC/API nữa

Có nhiều cách / tools để bắt đầu một dự án dùng aspnet core 1.0. Chúng ta có thể dùng [Visual Studio 2015 RC3](https://www.visualstudio.com/downloads/visual-studio-prerelease-downloads#sec1), [Visual Studio Code](https://code.visualstudio.com/) hoặc [Eclipse Che](https://eclipse-che.readme.io/) để viết code, deploy lên Windows / Linux / Mac hay Docker tùy thích. Nói chung là có máy nào xài máy đó, thích tool nào xài tool đó :), không còn phụ thuộc vào tools hay environment nữa.

Trong bài này, tôi sẽ giới thiệu cách develop một dự án aspnet core 1.0 dùng Visual Studio Code trên Mac, deploy lên Docker Swarm Cluster và dùng Docker Compose để scale các service trên Swarm Cluster.

## Cài đặt .NET Core 1.0, Visual Studio Code trên Mac

Để bắt đầu một dự án aspnet core 1.0 trên Mac, chúng ta phải chuẩn bị environment như sau:

- Cài đặt .NET Core 1.0 theo hướng dẫn [ở đây](https://www.microsoft.com/net/core#macos). Sau khi cài đặt xong, dùng terminal chạy lệnh ```dotnet --version``` để kiểm tra version của dotnet tool và nếu thấy giá trị là **1.0.0-preview2-003121** nghĩa là chúng ta đã on track.
- Cài đặt Visual Studio Code
- Cài đặt [NodeJS](https://nodejs.org/en/) và [Yeoman](http://yeoman.io/) (optional): bước này không bắt buộc, tuy nhiên tôi khuyến khích sử dụng Yeoman vì rất thuận tiện để tạo project structure. Sau khi cài Yeoman xong thì cài thêm [aspnet generator](https://www.npmjs.com/package/generator-aspnet) là chúng ta đã sẵn sàng. Nếu đã cài yeoman & aspnet generator trước đó thì cần update generator để có bộ aspnet generator mới nhất.

## Xin chào aspnet core 1.0

Khá dễ dàng để bắt đầu với aspnet core 1.0. Trên Mac, mở terminal và gõ lệnh ```yo aspnet```, chọn **Empty Web Application**, đặt tên dự án là **hello-aspnet** và nhấn enter. Yeoman sẽ tạo ra một project structure cho chúng ta.

```
$ mkdir ~/projects & cd ~/projects
$ yo aspnet

     _-----_     ╭──────────────────────────╮
    |       |    │      Welcome to the      │
    |--(o)--|    │  marvellous ASP.NET Core │
   `---------´   │        generator!        │
    ( _´U`_ )    ╰──────────────────────────╯
    /___A___\   /
     |  ~  |     
   __'.___.'__   
 ´   `  |° ´ Y ` 

? What type of application do you want to create? Empty Web Application
? What's the name of your ASP.NET application? hello-aspnet
   create hello-aspnet/.gitignore
   create hello-aspnet/Program.cs
   create hello-aspnet/Startup.cs
   create hello-aspnet/project.json
   create hello-aspnet/web.config
   create hello-aspnet/Dockerfile
   create hello-aspnet/Properties/launchSettings.json
   create hello-aspnet/README.md


Your project is now created, you can use the following commands to get going
    cd "hello-aspnet"
    dotnet restore
    dotnet build (optional, build will also happen when it's run)
    dotnet run
```

Sau đó chúng tao vào thư mục **hello-aspnet** và chạy các lệnh ```dotnet restore```, ```dotnet build``` và ```dotnet run``` để chạy dự án.

```
➜  hello-aspnet dotnet run
Project hello-aspnet (.NETCoreApp,Version=v1.0) was previously compiled. Skipping compilation.
Hosting environment: Production
Content root path: /Users/phihuynh/projects/hello-aspnet
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:5000/  
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 88.3022ms 20
```

Sau đó, mở trình duyệt và nhập vào địa chỉ ```http://localhost:5000``` là chúng ta đã có web app đầu tiên chạy trên aspnet core 1.0.

![say hello to aspnet core](/images/aspnetcore01.png)

## Xin chào Docker

Chúng ta đã có thể say hello với aspnet core 1.0 trên Mac, giờ chúng ta tiếp tục bắt đầu với Docker. Tôi giả sử là chúng ta đã là quen sơ sơ với Docker rồi nên sẽ không nói nhiều về benefits của Docker nữa.

Để chạy hello-aspnet app vừa rồi trên Docker, cơ bản chúng ta sẽ đi qua các bước như sau:

1. Cài đặt Docker Toolbox, bộ Docker Toolbox sẽ bao gồm Docker Machine, Kitematic & VirtualBox
2. Dùng **docker-machine** để tạo ra một máy ảo boot2docker
3. Build image cho app hello-aspnet
4. Deploy hello-aspnet app lên Docker bằng docker commands    

### Cài đặt Docker Toolbox

[Docker Toolbox](https://www.docker.com/products/docker-toolbox) có 2 phiên bản cho Windows & Mac. Chúng ta lựa chọn phiên bản phù hợp với OS của mình. Sau khi cài xong, chúng ta sẽ có Docker Machine, Kitematic & Virtualbox để sẳn sàng tạo máy ảo.

### Tạo máy ảo bằng docker-machine

Sau khi đã cài Docker Toolbox, mở terminal lên, dùng lệnh sau để tạo máy ảo

```
$ docker-machine create -d virtualbox docker00 
```