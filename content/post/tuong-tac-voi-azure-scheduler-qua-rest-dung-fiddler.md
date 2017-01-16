+++
title = "Tương tác với Azure Scheduler qua REST API dùng Fiddler"
date = "2014-10-22T15:00:00"
tags = ["Azure", "Azure Scheduler"]
categories = ["Cloud", "Azure"]
menu = ""
banner = ""
+++

## Giới thiệu về Windows Azure Scheduler

Azure Scheduler cho phép chúng ta tạo ra các jobs trên cloud để gọi các services bên trong hoặc ngoài Microsoft Azure. Ví dụ như Azure Scheduler có thể gọi các request HTTP/HTTPS enpoints hoặc gửi các message đến Azure Storage Queue. Bạn có thể chạy job ngay lập tức, hoặc theo một lịch nào đó, hoặc vào một thời điểm nào đó ở tương lai.

## Quản lý Azure Scheduler bằng cách nào?
Bạn có thể quản lý Azure Scheduler bằng các cách sau:

- REST API
- .NET Client
- PowerShell
- xpat-cli
- Azure Portal

Dùng Azure Portal dĩ nhiên là cách đơn giản và trực quan nhất, các bạn có thể login vào Azure Portal rồi tạo Job Collection mới, sau đó tạo các job mới. Hôm nay mình muốn giới thiệu dùng REST API để tạo các job mới.

## Tạo một Management Certificate và import vào Subscription

Để có thể gọi REST API, trước tiên cần phải authenticate các request. Cơ bản có 2 cách để authenticate request REST API, một là dùng Windows Azure AD, hai là dùng management certificate. Bài này tôi xin giới thiệu dùng cách thứ 2. Bạn có thể tham khảo 2 cách này ở http://msdn.microsoft.com/library/azure/ee460782.aspx

Để import certificate vào Azure Subscription, chúng ta sẽ tạo ra một certificate trước, và phải import vào mục Personal, chi tiết xem tại http://msdn.microsoft.com/en-US/library/azure/gg551722.aspx

Để tạo một certificate, làm theo các bước sau:

- Mở CMD
- Gõ lệnh makecert -sky exchange -r -n “CN=DevTest” -pe -a sha1 -len 2048 -ss My “DevTest.cer” . Lệnh này sẽ tạo ra một Certificate là DevTest và lưu trong thư mục Personal của Certificate Management.
- Nhấn Windows+R rồi gõ vào certmgr.msc để mở Certificate Management
- Vào thư mục Certificates -> Personal -> Certificates, chúng ta sẽ thấy một certificate tên là DevTest mới vừa tạo.
- Nhấn chuột phải vào DevTest, chọn All Tasks, sau đó chọn Export, nhấn Next, Next, Next và nhập vào tên file để export. Ví dụ: C:\Users\phihuynh\Desktop\DevTest.cer, sau đó nhấn Next, Finish
![Certificates](/images/asr01.png)

## Import certificate vào Azure Subscription

Sau khi  đã tạo ra một certificate và export nó ra màn hình desktop, bước kế tiếp là import nó vào Azure Subscription, theo các bước như sau:

- Đăng nhập vào manage.windowsazure.com
- Chọn mục Settings, nhấn vào Management Certificates, sau đó chọn Upload và chọn file DevTest.cer vừa export ở Desktop và nhấn dấu Checked để upload.
![Upload Certificate](/images/asr02.png)
- Sau khi upload thành công sẽ có màn hình như bên dưới:
![Upload Certificate](/images/asr03.png)

## Gọi Azure Scheduler REST API dùng Fiddler

Sau khi đã chuẩn bị xong certificate, bạn phải copy certificate vào thư mục C:\Users\phihuynh\Documents\Fiddler2 để Fiddler sẽ dùng certificate này để mã hóa các request và gửi lên Azure Scheduler server.

Để gửi một request lên Azure Scheduler cũng giống như các request bình thường khác, chỉ có 2 điểm cần lưu ý:

```
URL Parameter: ?api-version=2014-04-01
Header: x-ms-version: 2013-03-01
```
![Upload Certificate](/images/asr04.png)
Kết quả trả về như sau
![Upload Certificate](/images/asr05.png)

## Kết luận

Cũng giống như tất cả các REST API Service khác, chúng ta cần chuẩn bị các bước để authenticate trước. Sau đó thì gọi service một cách bình thường.