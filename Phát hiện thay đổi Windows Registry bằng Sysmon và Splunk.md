## Mục tiêu

- Giám sát và phát hiện thay đổi của windows rgistry bằng sysmon và splunk.

## Sơ đồ mạng

<img width="676" height="450" alt="image" src="https://github.com/user-attachments/assets/cac5ce25-aed7-4853-82e4-cd9dd245aeea" />

## Lý thuyết

- **Sysmon**: Sysmon (System Monitor) là một tiện ích mạnh mẽ từ bộ công cụ Sysinternals của Microsoft, dùng để giám sát hành vi hệ thống ở mức thấp và ghi nhật ký các sự kiện bảo mật chi tiết vào Windows Event Log.

- **Windows Registry**: Windows Registry là cơ sở dữ liệu phân cấp được hệ điều hành Windows sử dụng để lưu trữ thông tin cấu hình cho: Hệ điều hành, Ứng dụng đã cài, các thiết bị phần cứng, người dùng và phiên đăng nhập. Registry được tổ chức giống như cây thư mục gồm 5 khóa gốc (root keys):

  + HKCR(HKEY_Class_Root): Quản lý liên kết định dạng file và COM Object
    
  + HKCU(HKEY_Curent_User): Quản lý thông tin về người đăng nhập hiện thời
    
  + HKLM(HKEY_Local_Machine): Cấu hình hệ thống, phần cứng, phần mềm toàn cục
 
  + HKU(HKEY_Users): Quản lý tài khoản toàn bộ người dùng
 
  + HKCC(HKEY_Current_Config): Cấu hình phần cứng hiện tại (RAM, màn hình...)

- **Splunk Universal Forwarder**: Là một lightweight agent được cài trên máy Windows (hoặc các hệ điều hành khác) để thu thập log theo thời gian thực và gửi về máy chủ Splunk.

- **Splunk Enterprise**: Là một nền tảng SIEM mạnh mẽ hỗ trợ thu thập, tìm kiếm, phân tích, cảnh báo và trực quan hóa dữ liệu log từ nhiều nguồn.

## Thực hành

- **Bước 1**: Tạo các index trên splunk enterprise:

<img width="1888" height="183" alt="image" src="https://github.com/user-attachments/assets/d3248669-b8dc-487a-a1c4-e58b21a08482" />

- **Bước 2**: Cấu hình Splunk Universal Forwarder và gửi logs của Sysmon

  + Tạo file input.conf:

  `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

  + Nội dung file:
 
  <img width="644" height="617" alt="image" src="https://github.com/user-attachments/assets/87ffdc8e-c71f-4e75-b8d4-256ceaff87bb" />

  + Khởi động lại:
 
  `C:\WINDOWS\system32>"C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" restart`

- **Bước 3**: Thực hiện các hành vi độc hại trên windows

  + Thêm mục persistence giả:

<img width="1674" height="338" alt="image" src="https://github.com/user-attachments/assets/d146677d-ec7d-44ae-88b8-b9fe661236df" />

  Tạo một giá trị (registry value) có tên là MalwareTest tại key Run trong nhánh HKCU và liên kết tới "C:\malwaretest.exe". Khi người dùng login thì "C:\malwaretest.exe" sẽ tự động start.

  + Sửa đổi một cài đặt bảo mật quan trọng:

<img width="1668" height="63" alt="image" src="https://github.com/user-attachments/assets/f667218c-3728-426d-913a-040cd70ecb1d" />

  Lệnh này đặt giá trị của registry key có tên DisableIPSourceRouting thành 1 => Vô hiệu hóa IP source routing trên tất cả adapter => Có thể bị lợi dụng cho kỹ thuật tấn công mạng, như spoofing hoặc bypass firewall.

 + Xóa một khóa registry

<img width="1570" height="129" alt="image" src="https://github.com/user-attachments/assets/be6bb5a1-6204-4395-902d-90748ee2d500" />

Xóa giá trị MalwareSimulation trong Run Key của người dùng hiện tại. => Nếu có một malware đã tạo key này để chạy mỗi khi user đăng nhập, thì lệnh này xóa bỏ cơ chế persistence đó. Defender hoặc analyst để gỡ mã độc.

- **Bước 4**: Phân tích logs

  **Tìm kiếm các mục tiêu persistence giả trong Splunk:**
 
  <img width="1494" height="985" alt="image" src="https://github.com/user-attachments/assets/ca64afe1-50ed-4d23-aba8-a8e28d88a8db" />

  ***Sự kiện thiết lập Registry Run Key – Dấu hiệu tạo persistence***

  ****Thời gian (UTC):**** 2025-07-27 02:30:57.730

  ****Thời gian (giờ địa phương):**** 07/27/2025 09:30:57 AM

  ****Hostname:**** VUTV_B22DCAT318

  ****User thực hiện:**** VUTV_B22DCAT318\bblack

  ****Process thực thi:****

  *****Image:***** `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

  *****Process ID:***** 10048
  
  *****Process GUID:***** {90fe1516-831c-6885-3802-000000008e00}

  ****Hành vi****: Tạo hoặc sửa đổi một giá trị trong Registry tại vị trí Run Key, cụ thể: `HKU\S-33\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\MalwareTest`, thiết lập `C:\malwaretest.exe`

  Điều này đồng nghĩa với việc chương trình malwaretest.exe sẽ tự động được khởi chạy mỗi khi người dùng đăng nhập.

  ****Mô tả kỹ thuật****: Run Key là một kỹ thuật phổ biến để thiết lập persistence – giúp phần mềm hoặc mã độc tự động khởi chạy sau khi khởi động hoặc đăng nhập lại vào hệ thống.

  ****Mapping MITRE ATT&CK:****

    + Technique ID: T1547.001
 
    + Technique Name: Registry Run Keys / Startup Folder

  ****Mức độ nghi vấn:**** Cao nếu không rõ mục đích của malwaretest.exe

  ****Đề xuất điều tra:****

  *****Xác minh file C:\malwaretest.exe:*****
    
  + Tra cứu hash trên VirusTotal: https://www.virustotal.com

  + Phân tích(Phân tích tĩnh và phân tích động)

  *****Phân tích hành vi tiến trình powershell.exe đã tạo giá trị này*****

  + Xác định câu lệnh cụ thể đã được chạy

  + Kiểm tra tiến trình cha mẹ

  *****Xác định nguồn gốc tập tin C:\malwaretest.exe*****

  + Xác định thời gian tạo

  + Xác định đường dẫn

  *****Điều tra người dùng thực hiện hành động*****

  *****Gỡ bỏ persistence*****

  *****Tăng cường phòng thủ*****
  
**Sự kiện xóa Registry Run Key – Khả năng liên quan đến hành vi xóa persistence**

<img width="1903" height="579" alt="image" src="https://github.com/user-attachments/assets/3d29d100-4be2-49be-b219-617d93719661" />

<img width="1393" height="869" alt="image" src="https://github.com/user-attachments/assets/37b9707b-830f-463c-8bab-b77ed9690c36" />

  ****Thời gian (UTC):**** 2025-07-27 02:48:51.320

  ****Thời gian (giờ địa phương):**** 07/27/2025 09:48:51 AM

  ****Hostname:**** VUTV_B22DCAT318

  ****User thực hiện:**** VUTV_B22DCAT318\bblack

  ****Process thực thi:****

  *****Image:***** `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

  *****Process ID:***** 10048
  
  *****Process GUID:***** {90fe1516-831c-6885-3802-000000008e00}

  ****Hành vi****: Tiến trình PowerShell đã xóa một giá trị registry tại vị trí Run Key, cụ thể: `HKU\S-1-5-21-2168365451-2537764932-582089911-1000\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\MalwareSimulation`

  ****Mô tả kỹ thuật****:  Hành vi xóa giá trị này nhiều khả năng là một nỗ lực nhằm xóa dấu vết hoặc vô hiệu hóa persistence đã được thiết lập trước đó.

  ****Mapping MITRE ATT&CK:****

    + Technique ID: T1547.001
 
    + Technique Name: Registry Run Keys / Startup Folder

  ****Mức độ nghi vấn:**** Trung bình

  ****Đề xuất điều tra:****
  
  ***+*Nếu hành vi là chủ động hợp pháp (từ người dùng, bảo mật...), ghi nhận là hợp lệ.*****

  *****Nếu hành vi là từ mã độc hoặc nghi ngờ mã độc tự xóa dấu vết, cần:*****

  + Kiểm tra toàn bộ host để tìm file độc hại khác.

  + Kiểm tra các persistence khác như Scheduled Task, Service...
