+++
title = "Azure Automation hay là PowerShell as a service"
date = "2014-10-31T10:20:00"
tags = ["Azure", "PowerShell"]
categories = ["DevOps", "Cloud", "Azure"]
menu = ""
banner = "banners/azure-sma.jpg"
+++

## Giới thiệu về Azure Automation
Microsoft thông báo là đã cho ra mắt liền tù tì cả lố dịch vụ mới, đọc mà chóng cả mặt. Thế là lại lặn lội tìm xem có gì hay, đọc sơ qua cái sớ thì thấy có em Azure Automation là gây ấn tượng mạnh nhất. Mò ngay :))

Azure Automation giúp cho các developer (hoặc DevOps) tự động hóa các công việc lập đi lập lại hàng ngày, các tác vụ mất nhiều thời gian (long-running) hoặc các tác vụ mà chúng ta làm thường mắc lỗi (ví dụ như là chỉnh sửa các thông tin cấu hình mỗi lần deployment). Chúng ta có thể dễ dàng create / deploy / monitoring / maintenance các resources trên môi trường Microsoft Azure dùng ngôn ngữ PowerShell (chính xác là PowerShell Workflow). Theo tôi thấy thì Auzure Automation cũng kết hợp cả Azure Scheduler để kích hoạt chạy các tác vụ trên Azure Automation.

Rõ ràng PowerShell ngày càng trở nên quan trọng trong hệ sinh thái của Microsoft. Việc Microsoft đưa PowerShell phục vụ cho các tác vụ tự động hóa trên môi trường Azure sẽ nâng tầm lên một bước nữa, mà tôi xin đặt tên một cách vui vẻ là PowerShell-As-A-Service (nhại theo Everything Is A Service 🙂 )

## Lợi ích khi sử dụng Azure Automation
- Tiết kiệm thời gian và tiền bạc với Automation
- Loại bỏ các công việc tay chân lặp đi lặp lại, các công việc tay chân dễ mắc lỗi hoặc các tác vụ long-runing
- Nâng cao hiệu quả và độ tin cậy
- Tương tác với các dịch vụ của Azure và các third-party dùng PowerShell
- Tăng khả năng sẵn sàng

## Chính sách giá của Azure Automation
Có 2 gói, Free và Standard, hiện tại gói Standard đang được dùng thử miễn phí cho đến cuối tháng 11. Thử ngay nhé, nếu không thì chúng ta vẫn còn cơ hội xài chùa vì đã có gói Free, chi tiết như sau:
![Container in visualizer](/images/azurepowershell01.png)

## Bạn cần phải chuẩn bị gì để bắt đầu với Azure Automation?
Để bắt đầu cần phải chuẩn bị các bước sau:

- Đăng ký account dùng thử Microsoft Azure (dĩ nhiên roài hehe)
- Kích hoạt dùng thử Azure Automation
- Bạn phải biết cơ bản kiến thức về PowerShell
- Nếu đã dùng thử Azure Scheduler rồi là một lợi thế