**Mục tiêu**

- Mục tiêu của nhiệm vụ này là phát hiện, điều tra và phản hồi các tác vụ được lên lịch trái phép hoặc đáng ngờ trên máy tính Windows có thể được sử dụng để duy trì hoạt động, thực thi các tập lệnh độc hại hoặc di chuyển ngang. Nhiệm vụ này sử dụng Sysmon để giám sát các tác vụ được lên lịch và Splunk để phân tích và trực quan hóa dữ liệu.

**Sơ đồ mạng**

<img width="676" height="450" alt="image" src="https://github.com/user-attachments/assets/cac5ce25-aed7-4853-82e4-cd9dd245aeea" />

**Lý thuyết**

- Scheduled Task là tác vụ được lên lịch, cho phép bạn tự động hóa việc chạy chương trình, script hoặc lệnh vào các thời điểm cụ thể, hoặc khi xảy ra một sự kiện nào đó.

**Thực hành**

1. Mô phỏng hoạt động tác vụ theo lịch trình độc hại

- Tạo một tác vụ theo lịch trình mới:

<img width="1148" height="196" alt="image" src="https://github.com/user-attachments/assets/4e56751a-5e6e-4596-8dce-33c7fc4d83cf" />

- Sửa đổi tác vụ đã lên lịch hiện có:

<img width="987" height="171" alt="image" src="https://github.com/user-attachments/assets/cd0f64e4-b96a-4cc2-9884-1ec839a6ba59" />

- Xóa tác vụ đã lên lịch:

<img width="992" height="73" alt="image" src="https://github.com/user-attachments/assets/5e59a27b-903e-44ae-b878-dfaaaf5f57a5" />

- Thực hiện tác vụ theo lịch trình:

<img width="970" height="79" alt="image" src="https://github.com/user-attachments/assets/e36e4f81-6ba7-41f1-8964-466f7b77d023" />

2. Phát hiện

- Phát hiện việc tạo tác vụ :
  
`index=sysmon_logs EventCode=1 Image="*schtasks.exe" | stats count by Image, CommandLine, User`

<img width="1909" height="686" alt="image" src="https://github.com/user-attachments/assets/33e7c0a1-2e3d-4f84-9ec3-b076e244ce69" />

- Phát hiện sửa đổi tác vụ :

`index=sysmon_logs EventCode=1 CommandLine="*change*" Image="*schtasks.exe"`

<img width="1909" height="686" alt="image" src="https://github.com/user-attachments/assets/fd56d165-fd69-4840-b021-63a879c2157d" />

- Phát hiện thực thi tác vụ :

`index=sysmon_logs EventCode=1 CommandLine="*run*" Image="*schtasks.exe"`

<img width="1564" height="986" alt="image" src="https://github.com/user-attachments/assets/d11b4e23-11a2-4463-8e76-b82b98c0b8b9" />

3. Báo cáo

a. Thông tin chung

- Hệ điều hành: Windows 10 (Build 19041.1503)

- Nguồn phát hiện: Splunk + Sysmon (EventCode=1 – Process Create)

- Máy tính liên quan: VUTV_B22DCAT318.B22DCAT318.ptit

- User liên quan: bblack (SID: S-1-5-18)

- Process đáng ngờ: `schtasks.exe` với command line `/run /tn MaliciousTask`                                         |

b. Tóm tắt sự cố

- Quá trình theo dõi log hệ thống qua Sysmon cho thấy việc thực thi nhiều tác vụ theo lịch trình thông qua schtasks.exe. Một trong các lệnh đặc biệt đáng ngờ là:

`C:\Windows\System32\schtasks.exe /run /tn MaliciousTask`

c. Nguồn phát hiện

- Sysmon Log (EventCode 1 – Process Create)

- Splunk Query: `index=windows_sysmon_logs EventCode=1 Image="*schtasks.exe"`

d. Dữ liệu kỹ thuật

- CommandLine đáng ngờ:
`schtasks.exe /run /tn MaliciousTask`

- Đường dẫn Image:
`C:\Windows\System32\schtasks.exe`

- ParentCommandLine:
`Không xác định (chưa cung cấp)`

- Tệp XML chứa các nhiệm vụ khác:
`"C:\ProgramData\Microsoft\ClickToRun\..."`

- Hash file đáng ngờ:

  + MD5: `76C0662E6D0834BD4A24EA6561B4DC42`
 
  + SHA256: `01C30E1EDF01C3980FAD548187ACA8365E951A5CCEFFDB910F7D8B0AD2BE8F`

e. Phân tích sự cố

- Việc chạy một Scheduled Task tên "MaliciousTask" là cực kỳ nghi ngờ – không trùng với bất kỳ scheduled task hợp lệ nào của hệ điều hành hoặc ứng dụng đã biết.

- Dấu hiệu cho thấy khả năng attacker sử dụng tác vụ theo lịch trình để:

- Tự động hóa thực thi mã độc.

- Duy trì sự hiện diện lâu dài (persistence).

- Tránh bị phát hiện thông qua các kỹ thuật Windows hợp pháp.

- Việc này có thể là hậu quả của việc khai thác ban đầu hoặc kỹ thuật living-off-the-land (LOLBins).

f. Đánh giá tác động

- Mức độ nghiêm trọng: Cao

- Ảnh hưởng tiềm tàng:

  + Tác vụ tự động chạy mã độc vào thời gian nhất định.

  + Gây nguy cơ mã hóa dữ liệu, đánh cắp thông tin, kết nối với C2.

  + Có thể cho thấy giai đoạn tiếp theo của tấn công (privilege escalation hoặc persistence).

g. Biện pháp đã thực hiện

- Xác định và ghi lại toàn bộ lệnh schtasks.exe trong 24 giờ qua.

- Phân tích các scheduled tasks bằng công cụ như:

  + schtasks /query /fo LIST /v

  + Get-ScheduledTask

- Xóa tác vụ "MaliciousTask".

- Thực hiện hash checking trên các binary liên quan.

- Quét hệ thống bằng phần mềm MalwareByteMalwareBytes.s

h. Đề xuất khắc phục

- Tăng cường giám sát hành vi schtasks.exe, đặc biệt là với các tham số /create, /run, /change.

- Thiết lập cảnh báo trên Splunk với điều kiện:
`CommandLine=*schtasks.exe* AND CommandLine IN ("*run*", "*create*", "*change*") AND NOT task_name IN whitelist`

- Kiểm tra scheduled task registry:
`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks`

- Nếu có nghi ngờ APT, thực hiện memory dump và forensic phân tích sâu hơn.

- Triển khai AppLocker để kiểm soát việc thực thi schtasks.exe.
