+++
title = "Tương tác với Azure Scheduler qua .NET Client"
date = "2014-10-27T18:00:00"
tags = ["Azure"]
categories = ["Cloud", "Azure"]
menu = ""
banner = ""
+++

## Giới thiệu
Azure Scheduler giúp chúng ta tạo các Job và thực thi chúng theo cách không thể dễ dàng hơn. Nếu ai đã từng làm Windows Services hoặc các background services thì với Azure Scheduler, có lẽ chúng ta sẽ không còn cần đến chúng nữa 🙂 Hôm nay tôi xin giới thiệu với các bạn cách tương tác với Azure Scheduler qua .NET Client.

## Tạo Certificate và Import vào MSDN Subscription
Sau khi đã chuẩn bị Certificate xong rồi, chúng ta bắt đầu tạo một Project, chúng ta sẽ theo các bước sau để lấy thông tin một Certificate trong C#:

- Mở Visual Studio (2013), nhấn vào File, New, Project
- Chọn Templates, Visual C#, Console Application. Đặt tên project là TestAzureScheduler, sau đó nhấn OK.
![Visual Studio](/images/asn01.png)
- Mở file Program.cs, chúng ta thêm vào hàm sau để đọc thông tin Certificate. Để biết thêm chi tiết về hàm này, các bạn đọc thêm thông tin tại http://msdn.microsoft.com/library/azure/ee460782.aspx (mục Authenticate using a management certificate)
```
private static X509Certificate2 GetStoreCertificate(string thumbprint)
{
    List<StoreLocation> locations = new List<StoreLocation>
    {
        StoreLocation.CurrentUser, 
        StoreLocation.LocalMachine
    };

    foreach (var location in locations)
    {
        X509Store store = new X509Store("My", location);
        try
        {
            store.Open(OpenFlags.ReadOnly | OpenFlags.OpenExistingOnly);
            X509Certificate2Collection certificates = store.Certificates.Find(
                X509FindType.FindByThumbprint, thumbprint, false);
            if (certificates.Count == 1)
            {
                return certificates[0];
            }
        }
        finally
        {
            store.Close();
        }
    }
    throw new ArgumentException(string.Format(
        "A Certificate with Thumbprint '{0}' could not be located.",
        thumbprint));
}
```
- Hàm trên có một thông số là thumbprint, đây chính là thông tin thumbprint của Certificate mà chúng ta đã tạo ra ở bài trước. Để lấy thông tin này, chúng ta nhấn Windows+R, sau đó gõ vào certmgr.msc và nhấn Enter. Cửa sổ Certificates Management sẽ hiển thị lên, chúng ta tiếp tục chọn Personal, Certificates ở cửa sổ phía trái. Khi đó ở cửa sổ phải, sẽ xuất hiện DevTest Certificate như hình dưới.
![Certificates](/images/asn02.png)
- Nhấn đôi chuột vào DevTest để hiển thị thông tin chi tiết của Certificate, chọn tab Details, kéo scroll bar xuống dưới cùng thì chúng ta sẽ thấy thông tin của Thumbprint, nhấn vào mục Thumbprint, chúng ta sẽ thấy nội dung thông tin của Thumbprint như hình sau:
![Certificate](/images/asn03.png)
- Copy nội dung của Thumbprint, và paste vào hàm Main trong Program.cs như sau:
```
static void Main(string[] args)
{
    var certificate = GetStoreCertificate("0c 0c 8b 3d 83 f1 16 dd fe 84 a5 c4 96 f7 1a 62 59 f1 d8 fe");
}
```
## Tạo request đến Azure Scheduler REST API
Đến đây thì ta đã chuẩn bị xong vụ Certificate, tiếp theo là sẽ tạo một HttpRequest đến Azure Scheduler REST API. Tiếp tục các bước sau:

- Từ Visual Studio, nhấn chuột phải vào project TestAzureScheduler chọn Manage Nuget Packages…
- Từ cửa sổ Manage Nuget Packages, mục bên trái ta chọn Online, ở cửa sổ tìm kiếm gõ vào HttpClient, sau đó sẽ hiển thị Microsoft HTTP Client Libraries, chọn mục này và nhấn Install, nhấn I Accept để cài library nào vào project, sau đó nhấn Close
- Sau đó, chúng ta sẽ thêm mã để tạo ra một request đến Azure Scheduler REST API như sau:
```
static void Main(string[] args)
{
    var certificate = GetStoreCertificate("0c 0c 8b 3d 83 f1 16 dd fe 84 a5 c4 96 f7 1a 62 59 f1 d8 fe");
 
    var apiVersion = "api-version=2014-04-01";
    var uri = string.Format("https://management.core.windows.net/3a030e09-3eb5-4224-b282-746af502fb55/cloudservices/CS-SoutheastAsia-scheduler/resources/scheduler/~/JobCollections/jc1?{0}", apiVersion);
 
    var handler = new WebRequestHandler();
    handler.ClientCertificates.Add(certificate);
    using (var client = new HttpClient(handler))
    {
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

## AND RUN ...
Chúng ta chạy thử chương trình, nó sẽ gửi request đến Azure Scheduler REST API và trả về danh sách các job đang có 🙂
![Program](/images/asn04.png)

Happy coding :)))