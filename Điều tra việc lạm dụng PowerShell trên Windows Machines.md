**Mục tiêu**

- Mục tiêu của nhiệm vụ này là phát hiện, điều tra và ứng phó với các hoạt động PowerShell độc hại hoặc trái phép trên máy tính Windows bằng Sysmon và Splunk. Trọng tâm là xác định các lệnh PowerShell đáng ngờ, các tập lệnh được mã hóa và các hoạt động như tải xuống hoặc thực thi tệp từ các đường dẫn không chuẩn.

**Sơ đồ mạng**

<img width="598" height="370" alt="image" src="https://github.com/user-attachments/assets/220173d6-5539-4b00-96e9-ebeeb736a8e2" />

**Thực hành**

- [Tài liệu hướng dẫn](https://github.com/0xrajneesh/90-days-security-challenge/blob/main/Challenge%232/Task%202%3A%20Investigating%20PowerShell%20Abuse%20on%20Windows%20Machines.md)

**Thực hành**


1. ***Thiết lập splunk***

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


2. ***Mô phỏng hoạt động đáng ngờ***

- Tải xuống một file từ internet:

`Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"`

<img width="1547" height="444" alt="image" src="https://github.com/user-attachments/assets/208f578b-cb4e-4b49-b6ca-95fb30fa9e0d" />

- Log được phát hiện trên splunk:

<img width="1197" height="440" alt="image" src="https://github.com/user-attachments/assets/f253ed07-98ba-4556-bd0f-8f9c53c23dd7" />

3. ***Báo cáo sự cố***

a.  Thông tin chung

| Mục                  | Nội dung                                                             |
| -------------------- | -------------------------------------------------------------------- |
| Tiêu đề sự cố        | Nghi ngờ lạm dụng PowerShell được phát hiện trên máy VUTV_B22DCAT318 |
| Ngày phát hiện       | 31/07/2025                                                           |
| Pháy hiện bởi        | Splunk SIEM (Sysmon + PowerShell logs)                               |
| Mức độ nghiêm trọng  | Trung bình -> Cao (Nếu là mã độc)                                    |
| Loại                 | PowerShell Remote Execution Abuse                                    |
| Trạng thái           | Đang đuọc điều tra                                                   |

b. Tóm tắt sự cố

  - Hệ thống phát hiện một phiên bản PowerShell bị lạm dụng để thực thi dòng lệnh tải file EICAR (mã giả lập mã độc) về thư mục Downloads. Đây là dấu hiệu phổ biến trong kiểm thử khả năng phát hiện của hệ thống AV/EDR hoặc hành vi của attacker khi kiểm tra phản ứng của hệ thống bảo vệ.

c. Nguồn phát hiện 

| Loại nguồn |	Chi tiết|
| ----------- | --------|
| SIEM Alert |	Splunk |
| Log hệ điều hành |	EventCode: 4104 (PowerShell Operational)|
| Event Source |	Microsoft-Windows-PowerShell/Operational|
| Tên máy	| VUTV_B22DCAT318.B22DCAT318.ptit |

d. Dữ liệu kỹ thuật

- User: NOT_TRANSLATED

- Command được thực thi:

`Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"`

- EventCode: 4104

- ScriptBlock ID: 8350b569-33a4-4307-beb7-38a7445a8cc8

- Nguồn log: Microsoft-Windows-PowerShell/Operational

- Thời gian: 2025-07-31 10:39:03

- TaskCategory: Execute a Remote Command

e. Phân tích sự cố

****Diễn biến:****

- Vào lúc 10:39:03 ngày 31/07/2025, trên máy VUTV_B22DCAT318, hệ thống log PowerShell ghi nhận một dòng lệnh Invoke-WebRequest được thực thi, tải tệp EICAR từ Internet về máy tính người dùng.

- File EICAR là mã giả lập virus tiêu chuẩn dùng để kiểm tra phản ứng của phần mềm chống virus và giám sát.

****Chi tiết kỹ thuật:****

- Lệnh này thường không xuất hiện trong hành vi thông thường của người dùng không chuyên kỹ thuật.

- ScriptBlock được đánh dấu là thực thi từ xa (TaskCategory: Execute a Remote Command).

- Event ID 4104 thường là dấu hiệu đầu tiên trong việc abuse PowerShell.

****Mục tiêu:****

- Máy tính của người dùng nội bộ (VUTV_B22DCAT318).

- Không xác định được tài khoản người dùng cụ thể (User=NOT_TRANSLATED), cần kiểm tra thêm Windows Security logs.

****Hậu quả:****

- Hiện tại, chỉ thấy hành vi tải file EICAR về máy, chưa ghi nhận dấu hiệu thực thi hoặc lây nhiễm.

- Nếu hệ thống không phát hiện hoặc phản ứng với file này, nguy cơ từ các file thực sự độc hại sẽ cao hơn.

f. Đánh giá tác động

| Thành phần bị ảnh hưởng	| Mức độ ảnh hưởng	| Mô tả |
| ------------------- | -------- | ---------------- |
| Máy VUTV_B22DCAT318	| Trung bình | Có thể bị khai thác hoặc đang bị kiểm thử |
| Phần mềm bảo mật	| Thấp	| Cần xác minh phản ứng của AV/EDR |

g. Biện pháp đã thực hiện

- Gửi cảnh báo đến quản trị viên hệ thống.

- Quét máy VUTV_B22DCAT318 bằng phần mềm EDR/AV.

- Kiểm tra tài khoản nào thực thi PowerShell.

- Audit thêm EventID 4688 (Process Creation) và 4103 (Module Load) để xác định toàn bộ chuỗi hành vi.

h. Đề xuất xử lý/khắc phục

- Chấm dứt các tiến trình độc hại:

  + Xác định và tiêu diệt các tiến trình PowerShell độc hại: `Stop-Process -Name powershell -Force`
 
- Chặn các IP độc hại:

  + Chặn các kết nối đi đáng ngờ:

    `New-NetFirewallRule -DisplayName "Block Malicious IP" -Direction Outbound -Action Block -RemoteAddress <malicious-ip>`

- Thiết lập cảnh báo:

  + Tạo cảnh báo Splunk cho các lệnh PowerShell đáng ngờ:

    `index=sysmon_logs EventCode=1 CommandLine="*Invoke-WebRequest*" OR CommandLine="*EncodedC`
