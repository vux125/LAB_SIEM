**Mục tiêu**

- Mục tiêu của nhiệm vụ này là phát hiện, điều tra và ứng phó với các hoạt động PowerShell độc hại hoặc trái phép trên máy tính Windows bằng Sysmon và Splunk. Trọng tâm là xác định các lệnh PowerShell đáng ngờ, các tập lệnh được mã hóa và các hoạt động như tải xuống hoặc thực thi tệp từ các đường dẫn không chuẩn.

**Sơ đồ mạng**

<img width="598" height="370" alt="image" src="https://github.com/user-attachments/assets/220173d6-5539-4b00-96e9-ebeeb736a8e2" />

**Thực hành**

- [Tài liệu hướng dẫn](https://github.com/0xrajneesh/90-days-security-challenge/blob/main/Challenge%232/Task%202%3A%20Investigating%20PowerShell%20Abuse%20on%20Windows%20Machines.md)

**Thực hành**

1. Thiết lập splunk

- Chính sửa file input.conf trên windows cấu hình cho Splunk Universal Forwarder:

<img width="641" height="628" alt="image" src="https://github.com/user-attachments/assets/ef25f7be-8c77-4e38-93a8-6af666c09e1b" />

- Bật PowerShell Script Block Logging:

<img width="1345" height="576" alt="image" src="https://github.com/user-attachments/assets/b535f397-681e-46db-8ab8-9f5f87dc20b3" />

- Thêm index trong splunk trên máy chủ:

<img width="1884" height="217" alt="image" src="https://github.com/user-attachments/assets/d9c57c5e-1440-461b-8b30-57c569eabb50" />

- Khởi động lại Splunk Universal Forwarder:

<img width="978" height="396" alt="image" src="https://github.com/user-attachments/assets/77e4e876-d07c-4f45-aaa0-d2ff3a18215c" />

- Kiểm tra trên Splunk đã nhận được log chưa:

<img width="1874" height="72" alt="image" src="https://github.com/user-attachments/assets/59486c62-f6ea-4c74-9d7f-b6e30d09530f" />

2. Mô phỏng hoạt động đáng ngờ

- Tải xuống một file từ internet:

`Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"`

<img width="1547" height="444" alt="image" src="https://github.com/user-attachments/assets/208f578b-cb4e-4b49-b6ca-95fb30fa9e0d" />

3. 
