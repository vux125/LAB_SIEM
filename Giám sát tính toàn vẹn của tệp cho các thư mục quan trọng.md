**Mục tiêu**

- Mục tiêu của nhiệm vụ này là giám sát và điều tra các thay đổi trái phép đối với các tệp và thư mục quan trọng trên máy tính Windows bằng cách sử dụng Sysmon để ghi nhật ký chi tiết và Splunk để phân tích.
Nhiệm vụ này bao gồm phát hiện việc tạo, sửa đổi và xóa tệp trong các thư mục nhạy cảm để xác định các sự cố bảo mật tiềm ẩn.

**Sơ đồ mạng**


**Thực hành**

1. Thiết lập

- File inputs.conf:

<img width="677" height="650" alt="image" src="https://github.com/user-attachments/assets/196fafa0-a07d-47ce-abbc-16a5b9251a11" />

- File outpus.conf:

<img width="677" height="650" alt="image" src="https://github.com/user-attachments/assets/6714ba66-ee33-4015-b553-94b69b478ec3" />

-  Khởi động lại Universal Forwarder: `splunk restart`

2. Mô phỏng các thay đổi tệp trái phép

- Tạo một tập tin mới:

`echo MaliciousContent > C:\Windows\System32\malicious.exe`

<img width="919" height="103" alt="image" src="https://github.com/user-attachments/assets/3c45b57c-cdee-4336-a57e-9cf08dfd3761" />

- Sửa đổi một tập tin hiện có:

`echo AlteredContent >> C:\Windows\System32\config.sys`

<img width="921" height="78" alt="image" src="https://github.com/user-attachments/assets/f9b1fe71-8b8c-414f-afcc-47b5ddc3c93d" />

- Xóa một tập tin quan trọng:

`del C:\Windows\System32\important.dll`

<img width="722" height="79" alt="image" src="https://github.com/user-attachments/assets/79e87c4f-90ce-4c17-a3b0-58f532c6e34a" />

3. Phát hiện 

- Log tạo file:
 
<img width="1129" height="439" alt="image" src="https://github.com/user-attachments/assets/3138d309-64c2-47f8-b1b6-dc74f6de603c" />

- Log ghi đè file:

<img width="1129" height="439" alt="image" src="https://github.com/user-attachments/assets/a4093093-cf9a-44d7-8c3e-1c4ac1c19287" />

- Log xóa file:

<img width="1129" height="439" alt="image" src="https://github.com/user-attachments/assets/05f98938-b81a-44a5-919a-ed0ffef2afda" />

4. Báo cáo

a. Thông tin chung

- Tên sự cố: PowerShell Abuse - Tạo & Thao Tác Tập Tin Hệ Thống

- Ngày phát hiện: 02/08/2025

- Hệ thống bị ảnh hưởng: VUTV_B22DCAT318.B22DCAT318.ptit

- Nguồn phát hiện: Splunk SIEM - PowerShell Logs (EventCode 4104)

b. Tóm tắt sự cố

- Ba lệnh PowerShell đáng ngờ được phát hiện thực thi liên tục trong khoảng thời gian ngắn, có thể là dấu hiệu của hành vi tấn công hoặc thử nghiệm phá hoại hệ thống thông qua PowerShell:

  - Tạo file thực thi độc hại (malicious.exe) trong thư mục C:\Windows\System32.

  - Ghi đè file cấu hình hệ thống (config.sys) bằng nội dung lạ.

  - Xóa file thư viện quan trọng (important.dll) trong thư mục hệ thống.  

c. Dữ liệu kỹ thuật

- Thời gian	ScriptBlock ID	Command thực thi

  - 09:20:41 PM	460bd9df-ab25-4580-95a7-4d55bddea93d	echo MaliciousContent > C:\Windows\System32\malicious.exe

  - 09:21:13 PM	b00fca56-e977-4a3c-9a53-4dfef150143d	echo AlteredContent >> C:\Windows\System32\config.sys

  - 09:21:26 PM	aa9156d3-494a-497b-84fe-344740414d42	del C:\Windows\System32\important.dll

- Thông tin bổ sung:

  - LogName: Microsoft-Windows-PowerShell/Operational

  - EventCode: 4104

  - EventType: 5 (Verbose ScriptBlock Logging)

  - TaskCategory: Execute a Remote Command

  - User: NOT_TRANSLATED

  - SID: S-1-5-21-2168365451-2537764932-582089911-1000

d. Phân tích sự cố

- Các hành vi như tạo .exe khả nghi, ghi đè file cấu hình hệ thống và xóa file DLL trong thư mục System32 là dấu hiệu rõ ràng của hành vi lạm dụng PowerShell với mục đích phá hoại hoặc cấy mã độc.

- Việc các script này được log lại qua EventCode 4104 cho thấy ScriptBlock Logging đã được bật, giúp ghi nhận chi tiết hành vi.

- Không thấy thông tin xác thực người dùng (User=NOT_TRANSLATED), cần truy vết qua SID hoặc correlation với các sự kiện đăng nhập.

e. Đánh giá tác động

- Nguy cơ bị chiếm quyền kiểm soát hệ thống cao nếu malicious.exe là payload độc hại.

- Việc xóa important.dll có thể dẫn đến lỗi ứng dụng hoặc crash hệ thống.

- Ghi đè config.sys có thể làm hệ điều hành không khởi động đúng cách hoặc gây lỗi bảo mật.

f. Biện pháp đề xuất

- Đã cô lập máy VUTV_B22DCAT318.

- Export toàn bộ log liên quan đến PowerShell, ScriptBlock, và các sự kiện đăng nhập từ máy này.

- Quét toàn bộ hệ thống bằng phần mềm diệt virus (Windows Defender & Malwarebytes).

- Tạm thời vô hiệu hóa quyền sử dụng PowerShell với non-admin users.
