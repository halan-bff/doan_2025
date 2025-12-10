Địa chỉ IP
server: 176.40.0.5
client: 176.40.0.7
attacker:176.40.0.3
=======================================================  
```bash 
labtainer -r idr_splunk_webscan
```
# (1.1) Ở server: Triển khai cấu hình trên server

Giải nén và cài đặt: 
- đợi giải nén 3-4p
```bash
sudo tar -xzf splunk-8.2.6-a6fe1ee8894b-Linux-x86_64.tgz -C /opt  
```
- cấp quyền 
```bash
sudo chown -R ubuntu:ubuntu /opt/splunk
echo "OPTIMISTIC_ABOUT_FILE_LOCKING = 1" | sudo tee -a /opt/splunk/etc/splunk-launch.conf
```
- tạo tài khoản
```bash
sudo /opt/splunk/bin/splunk start --accept-license  
```
``` bash
sudo /opt/splunk/bin/splunk status
```
Mở firefox http://127.0.0.1:8000  &
``` bash
firefox http://127.0.0.1:8000  &
```
- Rồi login: admin và mật khẩu là  Admin@123
- Rồi bật cổng 9997
```bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:Admin@123

```
``` bash
sudo netstat -tulpn | grep 9997

```
# (1.2) Ở client:  Triển khai cấu hình trên client
- Hãy giải nén và start UF rồi:
```bash
sudo tar -xzf splunkforwarder-8.2.6-a6fe1ee8894b-Linux-x86_64.tgz -C /opt
```
- cấp quyền
``` bash
sudo chown -R ubuntu:ubuntu /opt/splunkforwarder 
```
Rồi làm tiếp : 
```bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license 

```
- kiểm tra
``` bash
sudo /opt/splunkforwarder/bin/splunk status
```
kỳ vọng: splunkd is running.

- Rồi login: client và mật khẩu là  Admin@123

- Khai báo server 176.40.0.5:9997
```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 176.40.0.5:9997 -auth client:Admin@123
```

- Rồi Kiểm tra:
```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server -auth client:Admin@123
```
hoặc : sudo /opt/splunkforwarder/bin/splunk list forward-server

→ đợi 1-2 phút để nó ra như dưới:

```bash 
Active forwards:
	176.40.0.5:9997
Configured but inactive forwards:
	None
```

# (0.1) Ở server & client: Thiết lập log + index Splunk từ access.log
## 0.1.1. Trên Splunk Server (Splunk Enterprise)
Tạo index riêng cho web, ví dụ: web_lab
```bash
sudo /opt/splunk/bin/splunk add index web_lab -auth admin:Admin@123
```

## 0.1.2. Trên Client (Splunk Universal Forwarder)
### Bước 1 – Tạo/ sửa inputs.conf để monitor access.log
Tạo file (nếu chưa có):
Bạn mở file inputs.conf thủ công:
```bash
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```
- Sau đó dán nội dung 1 :
``` bash
[monitor:///var/log/apache2/access.log]
sourcetype = access_combined
index = web_lab
disabled = false
```

## Bước 2 – Restart Splunk UF để áp dụng cấu hình → bắt buộc làm
Khởi động lại UF ở client
```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

## 0.1.3. Ở client : Chạy WEB =================================
Cho root MySQL dễ dùng (tùy nhu cầu, có thể đặt mật khẩu khác)
```bash
sudo mysql -u root -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY ''; FLUSH PRIVILEGES;"
```

Import DB
```bash
sudo mysql -u root < ./src/users_account.sql
```

Copy web vào DocumentRoot
```bash
sudo cp ./src/html/* /var/www/html/
```

hoặc nhanh gọn
```bash
sudo mysql -u root -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY ''; FLUSH PRIVILEGES;"
sudo mysql -u root < ./src/users_account.sql
sudo cp ./src/html/* /var/www/html/
```

http://IP_CLIENT/ → chạy index.php

http://IP_CLIENT/login.php → chạy login.php

→ ip client:  176.40.0.7

http://176.40.0.7/ → chạy index.php

http://176.40.0.7/login.php → chạy login.php

``` bash
http://176.40.0.7/
```
Vì UF đang monitor file này, hệ thống sẽ thực hiện chuỗi:
Apache → ghi access.log → UF đọc dòng mới → gửi sang Server → Server lưu trong index web_lab.

# (0.2) Ở attacker, tấn công spam request 
Cách chạy
``` bash
./webscan.sh http://176.40.0.7
```


# (1.3) Ở server: Phát hiện sự cố	
## 1.3.1) Cách nhìn bằng giao diện:	
Web: mở http://176.40.0.5:8000  

→ đăng nhập admin / Admin@123
``` bash
http://176.40.0.5:8000
```
Tra bình thường
``` bash
index=web_lab sourcetype=access_combined
```

- Giờ thử lại 2 câu query:

	+ Lọc 404/403:
``` bash
index=web_lab sourcetype=access_combined status=404 OR status=403
```

 ➜ Sẽ ra toàn event lỗi 404/403.
	+ Đếm theo IP:
``` bash
index=web_lab sourcetype=access_combined
| stats count by clientip
| sort - count
```
➜ Sẽ ra bảng, IP attacker (176.40.0.3) đứng đầu với count rất lớn.

## 1.3.2) Cách nhìn bằng màn hình lệnh	
```bash
sudo /opt/splunk/bin/splunk search index=web_lab sourcetype=access_combined -auth admin:Admin@123 -maxout 20 | tee -a evidence.txt
```

# (1.4) Ở client : Kiểm soát sự cố  
## b1: Tải mod: 
```bash
sudo apt-get update && sudo apt-get install -y libapache2-mod-security2
```
  + bật a2emod lên 
``` bash
sudo a2enmod security2
sudo systemctl reload apache2
```
## b2: sửa file cấu hình chính  
```bash
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
sudo nano /etc/modsecurity/modsecurity.conf
```
Tìm dòng:          SecRuleEngine DetectionOnly
→ Sửa thành:    SecRuleEngine On
→ rồi lưu lại
→ Đảm bảo Apache đã load ModSecurity module → Chạy:
```bash
sudo a2enmod security2
```

## b3:  Thêm luật:
```bash
sudo nano /etc/modsecurity/custom_rules.conf
```
- Nội dung thêm:
``` bash
SecRule REMOTE_ADDR "@ipMatch 176.40.0.3" "id:1000001,phase:1,deny,log,status:403,msg:'Block 176.40.0.3'"
```

## b4: Sửa security2.conf 

→ Load file custom_rules.conf vào Apache
```bash
sudo cat /etc/apache2/mods-enabled/security2.conf
sudo nano /etc/apache2/mods-enabled/security2.conf
```
- Nội dung sửa: 
``` bash
<IfModule security2_module>
	
	SecDataDir /var/cache/modsecurity

	IncludeOptional /etc/modsecurity/modsecurity.conf
            IncludeOptional /etc/modsecurity/custom_rules.conf  
	IncludeOptional /usr/share/modsecurity-crs/*.load
</IfModule>
```

## b5: Test cấu hình Apache
```bash
sudo apache2ctl configtest
```
Kỳ vọng: Syntax OK


-  Restart Apache
```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```

-  Xác nhận ModSecurity thực sự đang chạy
```bash
sudo apache2ctl -M | grep security
```

## b6: Test lại trên attacker
Trên attacker:
```bash 
curl -I http://176.40.0.7
```
Kỳ vọng lần này:  HTTP/1.1 403 Forbidden


## b7: Trên client → kiểm tra log ModSecurity
Trên client:
```bash
sudo tail -n 30 /var/log/apache2/error.log
```
Kì vọng như ảnh:
[Mon Dec 01 12:53:33.446033 2025] [:error] [pid 3070] [client 176.40.0.3:52018] [client 176.40.0.3] ModSecurity: Access denied with code 403 (phase 1). IPmatch: "176.40.0.3" matched at REMOTE_ADDR. [file "/etc/modsecurity/custom_rules.conf"] [line "1"] [id "1000001"] [msg "Block 176.40.0.3"] [hostname "176.40.0.7"] [uri "/"] [unique_id "aS2PzVSGTiU3@@xAm3WcCgAAAAA"]

- kiểm tra: 
```bash
sudo tail -n 2 /var/log/apache2/error.log
```

Lưu ý:  tắt cho attacker tấn công lại

Giờ cần tắt cho attacker đánh lại thì có mấy mức “tắt”, mình đưa cách đơn giản nhất cho lab.

Trên client: disable module security2
```bash
sudo a2dismod security2
sudo systemctl reload apache2
```
a2dismod security2 → Apache không load module ModSecurity nữa.

Nếu muốn bật lại →  Apache đã load ModSecurity module → Chạy:
```bash
sudo a2enmod security2
```

# (1.5) Ở client: Diệt bỏ nguyên nhân	
Ở trên client:
- Sửa file:
``` bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Nội dung thêm vào/chỉnh sửa:
``` bash
<VirtualHost *:80>
    ServerName client-web
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

   <Directory "/var/www/html/backup">
          Require ip 127.0.0.1 176.40.0.7
   </Directory>
    <Location "/backup">
        Require ip 127.0.0.1 176.40.0.7
    </Location>    
    
ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog /var/log/apache2/access.log combined
</VirtualHost>
```

Rồi kiểm tra: 
```bash
sudo cat /etc/apache2/sites-available/000-default.conf
```

 Ở đây:
127.0.0.1 là loopback (curl http://127.0.0.1/backup).

176.40.0.7 là IP của chính client (curl http://176.40.0.7/backup từ client).

✔ Mọi IP khác (ví dụ attacker 176.40.0.3) đều bị 403.

- Reload Apache
```bash
sudo apache2ctl configtest     
sudo service apache2 reload     
```
- Test :
		+ Trên client: Dùng localhost
``` bash
curl -I http://127.0.0.1/backup
```
Hoặc dùng IP của client
``` bash
curl -I http://176.40.0.7/backup
```
→ Kỳ vọng: 200 OK (admin nội bộ vẫn xem được).
		+ Trên attacker (176.40.0.3):
``` bash
curl -I http://176.40.0.7/backup
```
→ Kỳ vọng: 403 Forbidden.
Như vậy: Không cần “chỉ đích danh 1 attacker”,

Mà là chỉ cho phép IP nội bộ → mọi attacker (IP khác) đều bị chặn, còn client thì vẫn truy cập bình thường.

# (1.6) Ở client: Theo dõi hậu sự cố (1)
	
```bash
mkdir -p ~/ir-backup
sudo tar czf ~/ir-backup/webscan_logs.tar.gz /var/log/apache2/access.log /var/log/apache2/error.log
sha256sum ~/ir-backup/webscan_logs.tar.gz | tee ~/ir-backup/webscan_logs.sha256
```

# (1.7) Ở server: Theo dõi hậu sự cố (2)	
## 1.7.1. Bước 1 : Vào trang  Search & Reporting để search dữ liệu	

``` bash
index=web_lab sourcetype=access_combined (status=404 OR status=403)
| stats count AS err_count by clientip
| where err_count >= 200
| sort - err_count
```

→ Time range: Last 5 minutes.

Bấm Save As → Alert…

## 1.7.2. Bước 2: Lưu search này thành Alert	
-  Tab 1: alert

	+ Title:  WEB_SCAN_404_403_Alert
	+ Description:  Cảnh báo khi 1 IP có >=200 HTTP 404/403 trong 5 phút (nghi vấn web directory brute-force).
	+ Alert type: Scheduled
	+ Run every: Cron job
	+ Nếu không có lựa chọn sẵn thì chọn Cron schedule và gõ: */5 * * * *  hoặc * * * * *
	+ Permissions: để Private.


-  Tab 2:  Trigger Conditions
	+  Trigger alert when: Number of Results is greater than 0
	+  Trigger: Once
	+  Throttle: có thể bỏ trống (hoặc suppress 5 phút nếu sợ spam).

-  Tab 3 : Trigger Actions → Log Event

Bấm + Add Actions → Log Event
	+ Event:  WEB_SCAN detected: ip=$result.clientip$ count=$result.err_count$
	+ Source: giữ nguyên alert:$name$
	+ Sourcetype: web_scan_alert
	+ Host: splunk-server
	+ Index: web_lab
➜ Bấm Save.

 ```bash 
WEB_SCAN detected: ip=$result.clientip$ count=$result.err_count$
```

## 1.7.3. Bước 3 : Kiểm tra  alert đã có trên server chưa
Sau khi cho attacker quét lại web, chờ > 5 phút, rồi trên server chạy:
```bash
sudo /opt/splunk/bin/splunk search 'index=web_lab sourcetype=web_scan_alert' -auth admin:Admin@123  | tee -a evidence.txt
```
Bạn sẽ thấy dòng kiểu:

WEB_SCAN detected: ip=176.40.0.3 err_count=395

→ Lúc đó có thể dùng chuỗi WEB_SCAN detected hoặc sourcetype=web_scan_alert cho checkwork.

Trên giao diện Splunk cũng search:
``` bash
index=web_lab sourcetype=web_scan_alert
```

→ Thấy event kiểu WEB_SCAN detected: ip=176.40.0.3 ... là OK, dùng luôn cho checkwork.

