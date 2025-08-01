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

3. Phát hiện và điều tra

- Tìm kiếm Nhật ký Sysmon trong Splunk:
  
`index=sysmon_logs EventCode=11 TargetFilename="*System32*" OR TargetFilename="*Program Files*" | stats count by TargetFilename, Image, User`

- Phát hiện các sửa đổi tệp :

`index=sysmon_logs EventCode=15 TargetFilename="*System32*" | stats count by TargetFilename, Image, User`

- Phát hiện xóa tệp :

`index=sysmon_logs EventCode=23 TargetFilename="*System32*" | stats count by TargetFilename, Image, User`

- Liên hệ các sự kiện với quá trình tạo ra quy trình:

`index=sysmon_logs (EventCode=1 OR EventCode=11 OR EventCode=23) | transaction ProcessId maxspan=1m`

- Trực quan hóa hoạt động của tệp trong Splunk:

  - Biểu đồ thanh về các tệp được sửa đổi nhiều nhất trong các thư mục quan trọng.

  - Biểu đồ tròn về người dùng thực hiện thay đổi tệp.

  - Dòng thời gian của các hoạt động lưu trữ để có thông tin điều tra.
