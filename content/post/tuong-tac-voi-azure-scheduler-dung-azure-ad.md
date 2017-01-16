+++
title = "Tương tác với Azure Scheduler dùng Azure Active Directory"
date = "2014-10-27T18:00:00"
tags = ["Azure"]
categories = ["Cloud", "Azure"]
menu = ""
banner = ""
+++

## Giới thiệu Azure Active Directory
Azure Active Directory (AAD) là một giải pháp cloud mà Microsoft đưa ra để quản lý identity & access một cách toàn diện. Bạn có thể dùng AAD để quản lý users & groups, quyền truy cập vào các app của bạn một cách dễ dàng. Hôm nay tôi sẽ giới thiệu cách truy cập vào Azure Scheduler REST API dùng OAuth 2.0.

## Tạo App mới trong Azure Directory
Để truy cập vào REST API service của Azure Scheduler, chúng ta phải tạo ra một App trong AAD, sau đó sẽ cấp phép cho App này truy cập vào REST API Services. Thao tác các bước sau để tạo ra một App trong AAD:

- Đăng nhập vào Microsoft Azure (manage.windowsazure.com)
- Từ menu bên trái ở trang chính của Microsoft Azure, chọn mục Active Directory, sau đó chọn Default Directory, sẽ thấy hình như bên dưới:
![Azure AD](/images/as01.png)
- Chọn tab Applications, sau đó nhấn nút Add, chọn Add an application my organization is developing
- Đặt tên cho Application, ví dụ: TestAzureScheduler, chọn NATIVE CLIENT APPLICATION
![Azure AD](/images/as02.png)
- Sau đó nhấn biểu tượng Next
- Nhập vào Redirect URI, ví dụ: http://localhost/testazurescheduler, nhấn OK để hoàn tất tạo app
- Từ cửa sổ của app TestAzureScheduler, chọn tab configure và copy Client ID
![Azure AD](/images/as03.png)
- Cũng ở trang configure application, bạn kéo xuống dưới đến mục permissions to other applications chọn Windows Azure Service Management API và chọn Access Azure Service Management. Nhấn Save để hoàn tất quá trình tạo Application mới.

## Lấy Access Token

- Mở Visual Studio (2013), tạo một Console Project mới, đặt tên là TestAzureScheduler2
- Mở file program.cs, thêm vào hàm bên dưới:
```
private static string GetAccessToken()
{
    var tenantId = "d8ac9062-da58-40c7-87f3-2ddea9fa470f";
    var clientId = "d5690d98-0aea-4596-88de-24e88c53c5a5";
    var redirectUri = "http://localhost/testazurescheduler";
 
    var context = new AuthenticationContext(
        string.Format(
        "https://login.windows.net/{0}/oauth2/token?api-version=1.0",
        tenantId));
 
    var result = context.AcquireToken(
        "https://management.core.windows.net/",
        clientId,
        new Uri(redirectUri));
 
    if (result == null)
        throw new InvalidOperationException("Failed to obtain the JWT token");
 
    return result.AccessToken;
}
```
Trong đó chúng ta sẽ lấy tenantId từ trang Default Directory, chọn View Endpoints
![Azure AD](/images/as04.png)

## Gửi request lên Azure Scheduler dùng Access Token

Sau khi đã có được Token rồi, thì việc gửi request lên Azure Scheduler REST API cũng giống như bài trước , bạn thêm đoạn code sau vào hàm Main

```
static void Main(string[] args)
{
  var token = GetAccessToken();
  
  var apiVersion = "api-version=2014-04-01";
  var uri = string.Format(
  "https://management.core.windows.net/3a030e09-3eb5-4224-b282-746af502fb55/cloudservices/CS-SoutheastAsia-scheduler/resources/scheduler/~/JobCollections/jc1?{0}",
  apiVersion);
  
  using (var client = new HttpClient())
  {
    client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);
    client.DefaultRequestHeaders.Add("x-ms-version", "2013-03-01");
    HttpResponseMessage response = client.GetAsync(uri).Result;
    if (response.IsSuccessStatusCode)
    {
      var result = response.Content.ReadAsStringAsync().Result;
      Console.WriteLine(result);
    }
  }
  
  Console.ReadLine();
}
```