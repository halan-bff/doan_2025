server: 176.46.0.5
client: 176.46.0.7
attacker:176.46.0.3
=======================================================  
```bash 
labtainer -r idr_splunk_portscanning
```

# (1.1) Ở server: Triển khai cấu hình trên server
Giải nén và cài đặt: 
```bash
sudo tar -xzf splunk-8.2.6-a6fe1ee8894b-Linux-x86_64.tgz -C /opt
```
Cấp quyền: 
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
Mở :
```bash 
firefox http://127.0.0.1:8000  &
```
- Rồi login: admin và mật khẩu là  Admin@123
- Rồi bật cổng 9997
```bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:Admin@123
```
rồi kiểm tra:
```bash
sudo netstat -tulpn | grep 9997
```

# (1.2) Ở client:  Triển khai cấu hình trên client
## 1.2.1. Bật dịch vụ trên client (mở cổng 22, 80, 3306)
Chạy các lệnh này trên client (176.46.0.7):
```bash
sudo service ssh start
sudo service apache2 start
sudo service mysql start
```
Kiểm tra cổng listen:
```bash
sudo ss -ltnp | grep -E ':22|:80|:3306'
```

## 1.2.2. Chạy Splunk
Hãy giải nén và start UF rồi:
```bash
sudo tar -xzf splunkforwarder-8.2.6-a6fe1ee8894b-Linux-x86_64.tgz -C /opt
```
Cấp quyền:
```bash 
sudo chown -R ubuntu:ubuntu /opt/splunkforwarder 
```
Rồi làm tiếp : 
```bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```
Kiểm tra:
```bash 
sudo /opt/splunkforwarder/bin/splunk status
```
kỳ vọng: splunkd is running.

- Rồi login: client và mật khẩu là  Admin@123

- Khai báo server 176.46.0.5:9997
```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 176.46.0.5:9997 -auth client:Admin@123
```

- Rồi Kiểm tra:
```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server -auth client:Admin@123
```
hoặc : sudo /opt/splunkforwarder/bin/splunk list forward-server
→ đợi 1-2 phút để nó ra như dưới:
```bash 
Active forwards:
	176.46.0.5:9997
Configured but inactive forwards:
	None
```

# (0.1) Ở server & client: Thiết lập log + index Splunk từ access.log
## 0.1.1. Tạo log ở client 
Tạo file log (nếu chưa có):
```bash
sudo touch /var/log/portscan_traffic.log
sudo chmod 666 /var/log/portscan_traffic.log
```
Chạy tcpdump bắt gói TCP SYN và ghi vào file:
```bash
sudo tcpdump -i eth0 -nn -tttt 'tcp[13] & 2 != 0' >> /var/log/portscan_traffic.log &
```
Dấu & để tcpdump chạy nền.

Filter 'tcp[13] & 2 != 0' = chỉ bắt gói có cờ SYN, thường là lúc bắt đầu kết nối → rất phù hợp để xem port scan.

Kiểm tra xem tcpdump đang chạy:
```bash
sudo ps aux | grep tcpdump
```
Kì vọng: Thấy một tiến trình tcpdump chạy nền.

File log lúc này có thể trống hoặc chỉ vài dòng.
```bash
sudo tail -n 5 /var/log/portscan_traffic.log
```
Ở attacker thử nghiệm nhỏ:
```bash
sudo nmap -sS -F 176.46.0.7
sudo nmap -sS -p 22,80,3306 176.46.0.7
```

## 0.1.2. Trên Splunk Server (Splunk Enterprise)
Trên server (176.46.0.5):

Tạo index portscan_lab:
```bash
sudo /opt/splunk/bin/splunk add index portscan_lab -auth admin:Admin@123
```
Kiểm tra index đã tồn tại:
```bash
sudo /opt/splunk/bin/splunk list index | grep portscan_lab
```

## 0.1.3. Trên Client (Splunk Universal Forwarder)
### Bước 1 – Mở file inputs.conf
```bash
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```
### Bước 2 – Thêm block monitor mới cho portscan_traffic.log → Dán đúng khối sau:
```bash 
[monitor:///var/log/portscan_traffic.log]
sourcetype = portscan:tcpdump
index = portscan_lab
disabled = false
```

### Bước 3 - Restart Splunk Universal Forwarder:
```bash
sudo /opt/splunkforwarder/bin/splunk restart
```
Kiểm tra:

Sau khi lên lại, kiểm tra UF đã monitor file ICMP chưa:
```bash
sudo /opt/splunkforwarder/bin/splunk list monitor -auth client:Admin@123
```

# (0.2) Ở attacker: tấn công port scanning
Trên attacker (176.46.0.3):

Scan toàn bộ port (SYN scan):
```bash
sudo nmap -sS -p- 176.46.0.7
```
Scan có nhận diện dịch vụ:
```bash
sudo nmap -sS -sV -p- 176.46.0.7
```
Scan “aggressive” (tuỳ chọn):
```bash
sudo nmap -A 176.46.0.7
```
Mỗi lần scan xong, Splunk sẽ nhận thêm event trong index=portscan_lab.

nhẹ nhàng thì có:
```bash
sudo nmap -sS -F 176.46.0.7
```

# (1.3) Ở server: Phát hiện sự cố	
## 1.3.1) Cách nhìn bằng giao diện
Mở :
```bash 
firefox http://127.0.0.1:8000   &
```
Login: admin / Admin@123.
Vào Apps → Search & Reporting

Tra bình thường

```bash 
index=portscan_lab sourcetype=portscan:tcpdump
```

- Thống kê theo src_ip, dst_ip, dst_port (trên UI)
✔ Bước 1 — dán nguyên câu SPL dưới đây vào ô Search:
```bash 
index=portscan_lab sourcetype=portscan:tcpdump
| rex "IP (?<src_ip>[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+)\.[0-9]+ > (?<dst_ip>[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+)\.(?<dst_port>[0-9]+):"
| stats count by src_ip, dst_ip, dst_port
| sort - count
```
→ Bấm Search.

✔ Bước 2 — giao diện sẽ hiển thị như sau:

A. Timeline

Bạn sẽ thấy spike tại đúng thời điểm attacker chạy nmap.

B. Bảng kết quả (table)

Cột sẽ gồm:
```bash 
src_ip	dst_ip	dst_port	count
176.46.0.3	176.46.0.7	22	RẤT NHIỀU
176.46.0.3	176.46.0.7	80	RẤT NHIỀU
176.46.0.3	176.46.0.7	3306	RẤT NHIỀU
176.46.0.3	176.46.0.7	1234	...
...	...	...	...
```
→ Đây chính là bằng chứng attacker đang quét rất nhiều port.

Bạn sẽ thấy hàng trăm dòng, mỗi dòng là một port khác nhau.

- Nhận diện IP quét nhiều port (dc(dst_port)) trên UI
- 
✔ Bước 1 — dán câu SPL sau vào ô Search
```bash 
index=portscan_lab sourcetype=portscan:tcpdump
| rex "IP (?<src_ip>[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+)\.[0-9]+ > (?<dst_ip>[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+)\.(?<dst_port>[0-9]+):"
| stats dc(dst_port) AS uniq_ports count AS total_events by src_ip
| where uniq_ports>=50
| sort - uniq_ports
```
→ Bấm Search.

✔ Bước 2 — Kết quả trên giao diện sẽ hiển thị dạng bảng:

## 1.3.2) Cách nhìn bằng màn hình lệnh
```bash
sudo /opt/splunk/bin/splunk search "index=portscan_lab sourcetype=portscan:tcpdump" -maxout 5 -auth admin:Admin@123 | tee -a evidence.txt 
```


# (1.4) Ở client : Kiểm soát sự cố	
Cài đặt: 
```bash
sudo apt-get update && sudo apt-get install -y iptables
```
Chặn IP attacker bằng iptables

Trên client:
```bash
sudo iptables -A INPUT -s 176.46.0.3 -j DROP
```
Kiểm tra rule:
```bash
sudo iptables -L INPUT -v -n | grep 176.46.0.3
```
Kì vọng: Thấy một dòng DROP từ 176.46.0.3.

- Kiểm tra từ attacker sau khi chặn

Trên attacker (176.46.0.3), thử quét lại:
```bash
sudo nmap -sS -p- 176.46.0.7
```
Kì vọng:
Rất nhiều port báo filtered hoặc không trả lời, scan rất lâu.


# (1.5) Ở client: Diệt bỏ nguyên nhân	
Thắt chặt firewall (ví dụ rule đơn giản)

Trên client (chỉ là ví dụ, dùng khi bạn muốn harden):
```bash
sudo iptables -F
```

```bash
sudo iptables -A INPUT -i lo -j ACCEPT
```

```bash
sudo iptables -A INPUT -p tcp -s 176.46.0.5 --dport 22 -j ACCEPT
```

```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

```bash
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

```bash
sudo iptables -A INPUT -j DROP
```

+ Kiểm tra:
```bash
sudo iptables -L INPUT -v -n
```
+ Nhanh gọn
```bash 
sudo iptables -F


sudo iptables -A INPUT -i lo -j ACCEPT

sudo iptables -A INPUT -p tcp -s 176.46.0.5 --dport 22 -j ACCEPT

sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

sudo iptables -A INPUT -j DROP

```
# (1.6) Ở client: Theo dõi hậu sự cố (1)
Trên client:
```bash 
mkdir -p ~/ir-backup
sudo tar czf ~/ir-backup/portscan_logs.tar.gz /var/log/portscan_traffic.log
sha256sum ~/ir-backup/portscan_logs.tar.gz | tee ~/ir-backup/portscan_logs.sha256
```

# (1.7) Ở server: Theo dõi hậu sự cố (2)
## c1. Tra Search dùng cho alert là (đã chạy OK rồi):
Nhập vào ô search
```bash 
index=portscan_lab sourcetype=portscan:tcpdump
| rex "IP (?<src_ip>[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+)\.[0-9]+ > (?<dst_ip>[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+)\.(?<dst_port>[0-9]+):"
| bin _time span=1m
| stats dc(dst_port) AS uniq_ports count AS total_events BY src_ip,_time
| where uniq_ports>=50
| sort - _time
```

→ Save As Alert

Trên Splunk Web, đang ở trang Search, chạy lại search trên.

Góc trên bên phải bấm Save As → Alert.


## c2. Điền phần “Settings”
- Title:  Port scan detected (portscan_lab)
- Description (optional):  Alert khi 1 địa chỉ IP quét >=50 port khác nhau trong 1 phút (log từ portscan_traffic.log).
- Permissions: Để Private nếu chỉ dùng trong lab; nếu muốn share cho cả app thì chọn Shared in App – chọn cái nào cũng được, thường để Private cho đơn giản.
- Alert type: chọn Scheduled.
Dòng bên dưới (run every…): Mở drop-down, chỉnh thành:
- Run every: Ngay dưới phần Alert type:
Có dòng “Run every hour / day / …”. Bấm vào đó. Chọn tùy chọn Run on Cron Schedule. Sẽ xuất hiện ô để nhập Cron Expression. Điền:
```bash 
* * * * *
```
Ý nghĩa là: chạy mỗi 1 phút.

- Trigger Conditions → Trigger alert when:
	+ chọn Number of Results → chọn is greater than
	+ ô số: điền 0
→ Nghĩa là: nếu search trả về ≥1 dòng (có IP nghi port scan) thì bắn alert.

*Action
- Event : Port scan detected: src_ip=$result.src_ip$, uniq_ports=$result.uniq_ports$, total_events=$result.total_events$
- Source: giữ mặc định: alert:$name$
- Sourcetype: portscan_alert
- Host: splunk-server
- Index: portscan_lab

→ View alert
```bash 
Port scan detected: src_ip=$result.src_ip$, uniq_ports=$result.uniq_ports$, total_events=$result.total_events$
```

## c3. Test alert
Từ attacker (176.46.0.3) chạy lại scan:
```bash
sudo nmap -sS -p- 176.46.0.7
```
Đợi khoảng 1–2 phút cho Splunk chạy alert (vì mình để Scheduled mỗi 1 phút).
Trên Splunk Web:

Vào Activity → Triggered Alerts, chọn alert Port scan detected (portscan_lab) → xem đã có bản ghi chưa.

Hoặc search trực tiếp:

```bash 
index=portscan_lab sourcetype=portscan_alert
```

Trên màn hình lệnh CLI : 
```bash
sudo /opt/splunk/bin/splunk search 'index=portscan_lab sourcetype=portscan_alert' -auth admin:Admin@123 -maxout 20 | tee -a evidence.txt
```

