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


4. 



