## Mục tiêu

- Mục tiêu của nhiệm vụ này là phát hiện, điều tra và ứng phó với các cuộc tấn công brute-force nhắm vào dịch vụ Giao thức Máy tính Từ xa (RDP) trên máy tính Windows. Nhiệm vụ này sử dụng Sysmon và Splunk để giám sát các lần đăng nhập thành công và thất bại, đồng thời xác định các mẫu đáng ngờ cho thấy một cuộc tấn công.

## Sơ đồ mạng

<img width="919" height="518" alt="image" src="https://github.com/user-attachments/assets/2f04033f-c505-4859-8204-9bc3db4116cb" />

## Lý thuyết

- RDP (Remote Desktop Protocol) là một giao thức độc quyền của Microsoft cho phép bạn điều khiển từ xa một máy tính Windows thông qua mạng. Nó hoạt động như thể bạn đang ngồi trực tiếp trước máy tính đó.

- Công cụ Hydra (thường gọi là THC-Hydra) là một trong những công cụ mạnh mẽ nhất và phổ biến nhất trong lĩnh vực tấn công brute-force mật khẩu cho các giao thức mạng. Nó được sử dụng rộng rãi trong các hoạt động kiểm thử xâm nhập (penetration testing) để kiểm tra độ mạnh của mật khẩu.

## Thực hành

**Bước 1:** Mở dịch vụ RDP trên winsows 10

<img width="1164" height="682" alt="image" src="https://github.com/user-attachments/assets/04b7f62b-4ae5-4671-a127-8afa611c618b" />

**Bước 2:** Sử dụng nmap trên kali  quét máy mục tiêu

<img width="899" height="284" alt="image" src="https://github.com/user-attachments/assets/6ae12072-5556-44d8-9fe7-b030d8412fcd" />

- Ta thấy có cổng 3389 là dịch vụ RDP đang được mở

**Bước 3:** Tiến hành sử dụng hydra để brute-porce

<img width="1338" height="148" alt="image" src="https://github.com/user-attachments/assets/9ad2049f-3441-444c-accb-72ee8bf06381" />

**Buớc 4:** Giám sát và phát hiện trên splunk

<img width="1070" height="790" alt="image" src="https://github.com/user-attachments/assets/ae953ea7-df0a-4158-b632-32d00016952a" />

<img width="1178" height="954" alt="image" src="https://github.com/user-attachments/assets/a1d85115-6188-4e77-916e-ac3e7ecca93c" />

  ****Thời gian (UTC):**** 2025-07-30 04:06:09.597

  ****Thời gian (giờ địa phương):**** 07/30/2025 11:06:11 AM

  ****Hostname:**** VUTV_B22DCAT318

  ****RuleName:**** RDP

  ****Image:**** C:\Windows\System32\svchost.exe

  ****SourceIp:**** 192.168.239.134

  ****SourcePort:**** 54904

  ****DestinationIp:**** 192.168.239.133
  
  ****DestinationPort:**** 3389

  ****DestinationPortName:**** ms-wbt-server
  
  ****ProcessId:**** 1736

  ****ProcessGuid:**** {90fe1516-8b46-6889-7302-000000008f00}

Có rất nhiều kết nối tới cổng 3389 từ địa chỉa ip 192.168.239.134 được thực hiện bởi tiến trình svchost.exe.

***Logs đăng nhận thất bại từ Windows_security_logs***

<img width="1227" height="930" alt="image" src="https://github.com/user-attachments/assets/69114f9e-43ab-4606-b260-157fb94e7f29" />

Đây là log đăng nhập thất bại vào tài khoản administrator từ máy sử dụng tài khoản kali.

**Bước 5:** Phản ứng sự cố

- Sử dụng Tường lửa Windows để chặn các địa chỉ IP 192.168.239.134

- Đặt chính sách khóa để giới hạn số lần đăng nhập sai

- Tắt RDP nếu không cần thiết hoặc kết hợp các biện pháp như VPN, MFA, giám sát log và hạn chế IP sẽ giúp bạn an toàn hơn.

- Tạo alert Splunk tự động cho pattern brute-force RDP: `index="windows_sysmon_logs" DestinationPort=3389 | stats c by SourceIp, DestinationIp, _time | where c > 5`

