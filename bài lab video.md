server: 176.28.0.5 
client: 176.28.0.7 
attacker: 176.28.0.3 
=======================================================  
```bash 
labtainer -r idr_splunk_icmpflood
```
# (1.1) Ở server: Triển khai cấu hình trên server
Giải nén và cài đặt: 
```bash
sudo tar -xzf splunk-8.2.6-a6fe1ee8894b-Linux-x86_64.tgz -C /opt 
```
+ cấp quyền 
```bash 
sudo chown -R ubuntu:ubuntu /opt/splunk
echo "OPTIMISTIC_ABOUT_FILE_LOCKING = 1" | sudo tee -a /opt/splunk/etc/splunk-launch.conf
```
```bash
sudo /opt/splunk/bin/splunk start --accept-license 
```
```bash 
sudo /opt/splunk/bin/splunk status
```
- Rồi login: admin và mật khẩu là  Admin@123
- Rồi bật cổng 9997
```bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:Admin@123
```
➜ rồi kiểm tra 

```bash 
sudo netstat -tulpn | grep 9997
```
# (1.2) Ở client:  Triển khai cấu hình trên client
Hãy giải nén và start UF rồi:
```bash
sudo tar -xzf splunkforwarder-8.2.6-a6fe1ee8894b-Linux-x86_64.tgz -C /opt
```
```bash 
sudo chown -R ubuntu:ubuntu /opt/splunkforwarder 
```
Rồi làm tiếp : 
```bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license  
```
```bash 
sudo /opt/splunkforwarder/bin/splunk status
```
➜ kỳ vọng: splunkd is running.

- Rồi login: client và mật khẩu là  Admin@123

- Khai báo server 176.28.0.5:9997
```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 176.28.0.5:9997 -auth client:Admin@123
```

- Rồi Kiểm tra:
```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server -auth client:Admin@123
```
→ đợi 1-2 phút để nó ra Phải đúng như kì vọng : active forward 176.28.0.5:9997

# (0.1) Trên server: Tạo index riêng cho lab ICMP
Nếu muốn dùng index = icmp_lab , trên server:
```bash
sudo /opt/splunk/bin/splunk add index icmp_lab -auth admin:Admin@123
```
Câu lệnh kiểm tra:
```bash
sudo /opt/splunk/bin/splunk list index -auth admin:Admin@123
```
Kết quả ra được: 
```bash
icmp_lab
	/opt/splunk/var/lib/splunk/icmp_lab/db
	/opt/splunk/var/lib/splunk/icmp_lab/colddb
	/opt/splunk/var/lib/splunk/icmp_lab/thaweddb
```
# (0.2) Trên client: tạo log ICMP bằng tcpdump và cấu hình Splunk Forwarder đọc log
## 0.2.1) Tạo log ICMP bằng tcpdump
### bước 1: Tạo script ghi ICMP vào /var/log/tcpdump_icmp.log
```bash
sudo nano /usr/local/bin/tcpdump_icmp.sh
```
→ Nội dung file:
```bash 
#!/bin/bash
LOG_FILE="/var/log/tcpdump_icmp.log"
: > "$LOG_FILE"
tcpdump -ni any icmp -tttt >> "$LOG_FILE"
```

### bước 2 : Lưu lại, rồi cho phép thực thi:
```bash
sudo chmod +x /usr/local/bin/tcpdump_icmp.sh
```
### bước 3 :  Chạy tcpdump ở nền trước khi attacker bắn ICMP flood
Trong terminal client:
```bash
sudo /usr/local/bin/tcpdump_icmp.sh &
```
sang terminal khác làm tiếp.

### Lưu ý:
Nếu attacker bắn ICMP thì bạn xem nhanh log sẽ thấy các dòng kiểu:
```bash
sudo tail -f /var/log/tcpdump_icmp.log
```
Kì vọng: 

2025-11-20 14:01:23.123456 IP 176.28.0.3 > 176.28.0.7: ICMP echo request, id 1234, seq 1, length 64

Câu lệnh thử nghiệm (Tùy chọn không dùng):
```bash
sudo hping3 --icmp -i u1000 -d 120 176.28.0.7
```
Kiểm tra:
```bash 
tail -n 5 /var/log/tcpdump_icmp.log
```

## 0.2.2) Cấu hình Splunk Forwarder đọc /var/log/tcpdump_icmp.log
###  bước 1 : Tạo inputs.conf để monitor file ICMP
```bash
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```
Thêm (hoặc sửa cho phù hợp) đoạn sau:
```bash 
[monitor:///var/log/tcpdump_icmp.log]
sourcetype = tcpdump_icmp
index = icmp_lab
```
→ sourcetype = tcpdump_icmp: đặt tên sourcetype riêng, sau này search dễ.
→ index = icmp_lab: nếu bạn muốn một index riêng cho lab ICMP.

- Nếu chưa tạo index này trên server thì tạm thời có thể bỏ dòng này, log sẽ vào index mặc định (main).
→ Rồi Lưu file.

### bước 2 : Restart SF để nhận cấu hình mới
Restart là bắt buộc để nó lên index : 
```bash
sudo /opt/splunkforwarder/bin/splunk restart
```
Sau khi lên lại, kiểm tra UF đã monitor file ICMP chưa:
```bash
sudo /opt/splunkforwarder/bin/splunk list monitor -auth client:Admin@123
```
Kì vọng: Kết quả lệnh ra là cuối dòng có dòng: /var/log/tcpdump_icmp.log


# (0.3) Ở attacker: Tạo ICMP Flood ở attacker
Đây mới ICMP flood thật sự
```bash
sudo hping3 --icmp -c 100000 -d 120 176.28.0.7
```
```bash 
sudo hping3 --icmp -i u1000 -d 120 176.28.0.7
```
```
hoặc
```bash
sudo hping3 --icmp --flood -d 120 176.28.0.7
```

# (1.3) Ở server: Phát hiện sự cố
## 1.3.1) Cách nhìn bằng giao diện: 
Mở firefox http://127.0.0.1:8000  
Login: admin / Admin@123.

--> nhớ phóng to màn hình

Tôi chỉ cần đơn giản là index=icmp_lab "ICMP" như thế này với thêm thời gian và địa chỉ IP tấn công là được
```bash 
index=icmp_lab "ICMP"
```
```bash 
index=icmp_lab sourcetype=tcpdump_icmp
```
hoặc
```bash 
index=icmp_lab "ICMP" 
| rex "IP (?<src_ip>\d+\.\d+\.\d+\.\d+) > (?<dst_ip>\d+\.\d+\.\d+\.\d+)"
| table _time src_ip dst_ip
```

## 1.3.2) Cách nhìn bằng màn hình lệnh
In ra số lượng lệnh ra được: 
```bash
sudo /opt/splunk/bin/splunk search 'index=icmp_lab "ICMP"' -auth admin:Admin@123 -maxout 20 | tee -a evidence.txt
```

* Kì vọng ra được:

2025-11-29 18:15:33.466673 IP 176.28.0.3 > 176.28.0.7: ICMP echo request, id 34817, seq 52480, length 128

2025-11-29 18:15:33.454139 IP 176.28.0.7 > 176.28.0.3: ICMP echo reply, id 34817, seq 52224, length 128

2025-11-29 18:15:33.454126 IP 176.28.0.3 > 176.28.0.7: ICMP echo request, id 34817, seq 52224, length 128



# (1.4) Ở client : Kiểm soát sự cố
+ Cài iptables (nếu thiếu) :
```bash
sudo apt-get update && sudo apt-get install -y iptables
```
+ Kiểm tra iptables hoạt động :
```bash
sudo iptables -L
```
Kỳ vọng: Thấy các chain INPUT, FORWARD, OUTPUT với policy ACCEPT.

+ Thêm rule containment kiểu:
```bash
sudo iptables -A INPUT -s 176.28.0.3 -p icmp -j DROP
```
+ Cách loại bỏ để attacker tấn công lại được:
```bash
sudo iptables -D INPUT 1
sudo hping3 --icmp --flood -d 120 176.28.0.7
```

# (1.5) Ở client: Diệt bỏ nguyên nhân
→ Thêm rule limit ICMP
## Trên client 176.28.0.7:
### b1) Cho phép ICMP nhưng giới hạn tốc độ (VD: tối đa 10 packet/giây, burst 20)
```bash
sudo iptables -A INPUT -p icmp -m limit --limit 10/s --limit-burst 20 -j ACCEPT
```

### b2) Mọi ICMP vượt ngưỡng ở trên sẽ bị DROP
```bash
sudo iptables -A INPUT -p icmp -j DROP
```
Giải thích nhanh:

Khi attacker flood hàng trăm/thousands packet/s → vượt limit → rơi vào rule DROP.

Người dùng bình thường (ping vài lần) không sao, vì không vượt 10/s.

### b3) Kiểm tra lại rule để ghi vào báo cáo/checkwork
```bash
sudo iptables -L INPUT -n -v --line-numbers | grep icmp
```
Kì vọng ra được:
```bash 
num  pkts bytes target  prot opt in out source      destination
1      ...  ... ACCEPT  icmp ...  0.0.0.0/0  0.0.0.0/0  limit: up to 10/sec burst 20
2      ...  ... DROP    icmp ...  0.0.0.0/0  0.0.0.0/0
```

# (1.6) Ở client: Theo dõi hậu sự cố (1)
## k0) nhanh gọn:
```bash 
mkdir -p ~/ir-backup
sudo tar czf ~/ir-backup/icmp_lab_logs.tar.gz /var/log/tcpdump_icmp.log
sha256sum ~/ir-backup/icmp_lab_logs.tar.gz
```

## k1) Tạo thư mục chứa bằng chứng
```bash 
mkdir -p ~/ir-backup
```

## k2) Backup log ICMP vào file tar.gz
```bash
sudo tar czf ~/ir-backup/icmp_lab_logs.tar.gz /var/log/tcpdump_icmp.log
```
(Thông báo “Removing leading /” là bình thường.)

## k3) Sinh hash SHA-256 cho file backup
```bash 
sha256sum ~/ir-backup/icmp_lab_logs.tar.gz
```
✔ Lệnh này tạo file:
~/ir-backup/icmp_lab_logs.sha256 

→ chứa giá trị SHA-256 của file backup.


# (1.7) Ở server: Theo dõi hậu sự cố (2)
## 1.7.1. Bước 1 : Vào trang  Search & Reporting để search dữ liệu 
Đăng nhập Splunk Web trên server:
Trình duyệt vào http://127.0.0.1:8000  (hoặc IP server nếu bạn dùng remote).
Mở :
```bash 
firefox http://127.0.0.1:8000  &
```
- Rồi login: admin và mật khẩu là  Admin@123
- Ở thanh trên cùng, bên trái bấm vào Search & Reporting để vào app search.
- Ở góc trên bên phải có Time range picker (bắt buộc chọn là “Last 15 minutes”):
→  Chọn Last 15 minutes (tạm thời để xem có log ICMP trong 15p gần nhất).

- Trong ô Search, nhập đúng chuỗi sau (dán nguyên khối):
```bash 
index=icmp_lab "ICMP"
| rex "IP (?<src_ip>\d+\.\d+\.\d+\.\d+) > (?<dst_ip>\d+\.\d+\.\d+\.\d+)"
| bin _time span=10s
| stats count AS icmp_count BY _time src_ip dst_ip
| where icmp_count > 200
```

## 1.7.2. Bước 2: Lưu search này thành Alert
Rất quan trọng: trước khi bấm “Save As → Alert”, bạn nên chỉnh lại Time range phù hợp cho alert:
→  bắt buộc chọn là “Last 15 minutes”
Trong cửa sổ “Save As Alert”:

### Tab 1:  Phần Alert:
- Title : ICMP Flood from Attacker
- Description : Alert khi một địa chỉ src_ip gửi hơn 200 gói ICMP trong 10 giây
- Permissions
    + Để Private (mặc định). Nếu sau này muốn dùng chung cho người khác thì bạn có thể chỉnh thành “Shared in App”.
    + Alert type:    Chọn Scheduled (để Splunk kiểm tra theo chu kỳ).
    + Không chọn Real-time vì bản dùng thử và bản miễn phí đều bị hạn chế.

Ngay dưới phần Alert type:
- Có dòng “Run every hour / day / …”. Bấm vào đó.
→ Chọn tùy chọn Run on Cron Schedule.
→ Sẽ xuất hiện ô để nhập Cron Expression. Điền:
```bash 
* * * * *
```
Ý nghĩa là: chạy mỗi 1 phút.

Phần “At XX minutes past the hour” lúc này không còn ý nghĩa nữa, vì đã dùng cron.

Bạn có thể giữ Expires = 24 hours mặc định (Splunk sẽ giữ kết quả trigger trong 24 giờ).

### Tab 2 : Phần Trigger:
Vẫn trong cửa sổ “Save As Alert”, kéo xuống phần Trigger Conditions:
- Ô Trigger alert when:
    + Dòng đầu tiên: chọn Number of Results.
    + Dòng thứ hai: chọn is greater than.
- Ô số cuối cùng: nhập 0.
      ➜ Nghĩa là: nếu search trả về hơn 0 kết quả (tức có ít nhất một nguồn bị coi là flood) thì alert sẽ bắn.
- Ô Trigger: Chọn Once.
  → Nghĩa là mỗi lần Splunk chạy search (mỗi phút), nếu có kết quả thì bắn một lần.

### Tab 3:  Phần Actions:
Kiểm tra lại 5 ô:
+ Event: ICMP flood detected
```bash 
ICMP flood detected: src_ip=$result.src_ip$, dst_ip=$result.dst_ip$, icmp_count=$result.icmp_count$
```
+ Source: icmp_flood_alert (hoặc alert:$name$)
```bash 
icmp_flood_alert
```
+ Sourcetype: icmp_flood_alert
+ Host: trống hoặc splunk_server
+ Index: icmp_lab

→ Bấm Save ở dưới cùng của màn hình Alert.

## 1.7.3. Bước 3 : Kiểm tra  alert đã có trên server chưa
Trên máy attacker (container attacker trong Labtainer), chạy một trong các lệnh ví dụ:
```bash
sudo hping3 --icmp -i u1000 -d 120 176.28.0.7
```
Đợi khoảng 1–2 phút sau khi bắt đầu flood.

Vì bạn chọn hành động Log Event, Splunk sẽ ghi thông điệp vào index đã ghi

Vào Search & Reporting.

Chạy câu:
```bash 
index=icmp_lab sourcetype=icmp_flood_alert
```
Hoặc đơn giản:
```bash 
index=icmp_lab "ICMP flood detected"
```
Lệnh rõ ràng ở màn hình lệnh terminal: 
```bash
sudo /opt/splunk/bin/splunk search \
'index=icmp_lab sourcetype=icmp_flood_alert "ICMP flood detected"' \
-auth admin:Admin@123 -maxout 50 | tee -a evidence.txt
```

