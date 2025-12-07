  
server: 176.34.0.5

client: 176.34.0.7

attacker:176.34.0.3
  
Thử: 
```bash
 sudo systemctl restart logstash kibana elasticsearch
```


# (0.1) bật ssh     
Làm ở 2 terminal attacker và client : 
```bash
sudo systemctl status ssh
sudo systemctl restart ssh
```
Kiểm tra ở cả 2 terminal:
```bash
sudo ss -lntp | grep :22 || echo "Khong co tien trinh nao"
```
hoặc
```bash
sudo ss -lntp | grep :22
```
→ Nhớ phải kiểm tra xem attacker có ssh được vào client không đã
```bash
sudo systemctl stop xinetd
sudo systemctl disable xinetd
sudo systemctl restart ssh
```
- Có thể thử chạy:
``` bash
hydra -l ubuntu -P /opt/wordlist.txt -V -t 8 -f -o hydra.out ssh://176.34.0.7 

```
``` bash
ls -l /opt/wordlist.txt
```
- check log:
```bash
sudo tail -n 200 /var/log/fail2ban.log
sudo tail -n 200 /var/log/auth.log
```
- chạy ssh :
``` bash
ssh ubuntu@176.34.0.7
```


# (1.1) Triển khai trên server +  Cài đặt mật khẩu 
 - Đối với elasticsearch
```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```
+ Bật cơ chế auth  → ghi thêm cái câu dưới =)))))
```bash 
xpack.security.enabled: true
```
       
+  Khởi động lại:
```bash
sudo systemctl restart elasticsearch
sudo systemctl status elasticsearch --no-pager
```
- Trên server, ta sinh mật khẩu ngẫu nhiên
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```
+ rồi nhấn chữ y
+ lưu 2 mật khẩu chính: (pass)

Changed password for user kibana_system

PASSWORD kibana_system = LCAo2rmMwfZRFvuTe0tJ

Changed password for user elastic

PASSWORD elastic = ur36pN06X06KcKwpScnA

- Đôi với kibana: 
```bash
sudo nano /etc/kibana/kibana.yml

```
hoặc
``` bash
sudo cat /etc/kibana/kibana.yml
```
+  QUAN TRỌNG: Kibana PHẢI dùng user kibana_system (không dùng elastic ở đây)
``` bash
elasticsearch.username: "kibana_system"
elasticsearch.password: ""
xpack.security.enabled: true
xpack.encryptedSavedObjects.encryptionKey: "12345678901234567890123456789012"
```

- Rồi restart lại kibana:
```bash
sudo systemctl restart kibana
sudo systemctl status kibana
```
- Đối với logstash: 
Trên server, do Filebeat từ client → Logstash của server→ Elasticsearch

Filebeat gửi Beats tới Logstash 5044 (không cần user/pass).

Logstash phải có user/pass khi ghi vào ES. 
``` bash
ls -l /etc/logstash/conf.d/lab.conf 
```
Mở /etc/logstash/conf.d/lab.conf và đảm bảo đoạn output có:
```bash
sudo nano /etc/logstash/conf.d/lab.conf 
```
 (pass)  Changed password for user elastic
 
PASSWORD elastic = ur36pN06X06KcKwpScnA

- Nhớ đoạn output có :
``` bash
    user  => "elastic"
    password => ""
```
=======================================================  
```bash 
output {

  elasticsearch {
    hosts => ["http://127.0.0.1:9200"]
    user  => "elastic"
    password => "<PASS_ELASTIC>"
    index => "lab-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```
=======================================================  
- Rồi khởi động :
```bash
sudo systemctl restart logstash
sudo systemctl status logstash --no-pager  
sudo cat /etc/logstash/conf.d/lab.conf
```
- Kiểm tra các cổng hết 5044, 9200, 5601 :
``` bash
sudo ss -lnt 'sport = :9200 or sport = :5601 or sport = :5044'  
```


# (1.2) Triển khai trên client   
- Đã cài sẵn công cụ filebeat trong client , bây giờ chỉ làm các lệnh dưới : 
```bash
sudo systemctl restart filebeat
```
- Kiểm tra chạy chưa :
→ để thấy 3 thứ: (1) Filebeat khởi động OK, (2) đã mở Input paths đúng

→ (3)ĐÃ KẾT NỐI tới Elasticsearch@176.34.0.5:9200 thành công.

```bash
sudo tail -n 50 /var/log/filebeat/filebeat
sudo filebeat test config -e          #kiểm tra cấu hình có config OK = đúng
sudo filebeat test output -e         # ko có chữ ERROR là đúng → oke hết rồi
```
``` bash
sudo filebeat test output -e  
```

# (0.2) Tại server, ta tạo Data View trong Kibana (trên firefox của server)
##  a) Xác nhận index đã tạo (trên terminal server) 
Hỏi ES xem đã có index lab-YYYY.MM.DD chưa.

Vì sao? Đây là “dấu hiệu dây chuyền” từ client → server đã thông.
``` bash
curl -sS 'http://127.0.0.1:9200/_cat/indices/lab-*?v'
curl -u elastic:yourpassword -sS 'http://127.0.0.1:9200/_cat/indices/lab-*?v'   
```
(pass)
Changed password for user elastic

PASSWORD elastic = ur36pN06X06KcKwpScnA

``` bash
curl -u elastic:ur36pN06X06KcKwpScnA -sS 'http://127.0.0.1:9200/_cat/indices/lab-*?v' 
```


→ Kết quả trông như:

health status index            uuid ... pri rep docs.count ...

yellow open   lab-2025.10.22   AbCd... 1   1   123

→ thấy ít nhất 1 index lab-<ngày> và docs.count > 0.

##  b) Tạo Data View trong Kibana
Mở firefox ở server ra : 
``` bash
firefox &

```
``` bash
http://176.34.0.5:5601
```
 hoặc 
``` bash
http://localhost:5601
```
→ Vào Stack Management → Index Pattern → Create
 
Name: lab
Pattern: lab-* 
Timestamp: @timestamp
→  Để Discover hiểu dữ liệu theo thời gian. Kết quả trông như: Khi vào Discover, chọn Data View lab, bạn sẽ thấy bảng log tăng dần.

Rồi chọn create index pattern 

Vào 3 gạch → chọn Discover

→ quan sát được giao diện log message

##  c) Kiểm tra xem rule and connection và alert dùng được không (?) 
Trong tab Stack Management --> Alerts and Insights --> Rules and Connectors thì cái "Rules and Connectors"

Còn trong tab Security --> gồm có Alert và Rule như cột bên trái 

→ tất cả phải bật bảo mật hết thì mới lên được

# (0.3) Incident A: Trên terminal Attacker,  ssh brute force thất bại
Dùng hydra với wordlist 5 từ của bạn. → Sinh nhiều lần sai → log Failed password → fail2ban ban. [attacker] brute-force

``` bash
hydra -l ubuntu -P /opt/wordlist.txt -I -t 4 -f ssh://176.34.0.7
```
hoặc :
``` bash
hydra -l ubuntu -P /opt/wordlist.txt -V -t 8 -f -o hydra.out ssh://176.34.0.7
```

để chạy

(1.3) Trên server, phát hiện incident A ở công cụ elk và terminal server 
(pass)
Changed password for user elastic

PASSWORD elastic = ur36pN06X06KcKwpScnA

Đếm số lần ssh brute force fail: 
``` bash
curl -sS -u elastic:ur36pN06X06KcKwpScnA \
  'http://127.0.0.1:9200/lab-*/_count?filter_path=count' \
  -H 'Content-Type: application/json' \
  -d '{
    "query":{"bool":{"filter":[
      {"term":{"host.name.keyword":"client"}},
      {"query_string":{"default_field":"message","query":"sshd AND (Failed OR Invalid)"}},
      {"range":{"@timestamp":{"gte":"now-2h","lte":"now"}}}
    ]}}
  }' \
| jq -c '{count: .count}' | tee -a /home/ubuntu/evidence.json
```

→ {"count":117}

tee -a /home/ubuntu/evidence.json


# (1.4) Containment cho incident A : fail2log → sau đó tắt để thực hiện cho incident B
Mục tiêu: ngăn sự cố lan rộng, giảm thiệt hại tức thời, chưa xử lý tận gốc.

=> Giai đoạn này chỉ “chặn đứng” hành vi, chứ chưa loại bỏ nguyên nhân sâu xa (vẫn có thể quay lại nếu gỡ chặn).

- Cài fail2log và nftables: 
```bash
sudo apt-get update && sudo apt-get install -y fail2ban nftables
```
- Rồi đặt lệnh: (2)
```bash
sudo tee /etc/fail2ban/jail.local >/dev/null <<'CONF'
[sshd]
enabled   = true
port      = 22
filter    = sshd
logpath   = /var/log/auth.log
backend   = auto
maxretry  = 5
findtime  = 60
bantime   = 600
banaction = nftables-multiport
CONF
```
→ jail.local của bạn đang để banaction = dummy → không chặn gì cả (chỉ phát hiện/ghi log). NÊn để phải là nftables-multiport
``` bash
sudo systemctl restart fail2ban nftables
sudo systemctl status fail2ban nftables 
```
và
``` bash
sudo nano /etc/fail2ban/jail.local
```

* Attacker thử lại tấn công
``` bash
hydra -l ubuntu -P /opt/wordlist.txt -V -t 8 -f -o hydra.out ssh://176.34.0.7 
```
→ Được vài giây sau, client đã chặn thành công khi banned Ip đã có ip của attacker 176.34.0.3 khi kiểm tra bằng câu lệnh: 
```bash
sudo fail2ban-client status sshd
```
* Lưu ý: Cần sử dụng lại terminal attacker nên cần ip attacker để thử nghiệm tiếp:
```bash
sudo fail2ban-client set sshd unbanip 176.34.0.3
sudo fail2ban-client set sshd addignoreip 176.34.0.3
```
Lí do: nếu trong thực tế , attacker không chỉ 1 địa chỉ ip được ;v. Chỉ là trong lab đang làm tối giản á nên dùng 1 cái cho tiện

# (0.4) Incident B: attacker ssh brute success, tạo persistence (nhẹ) trong phiên SSH và Cài cron
##  a) Terminal attacker (đang ở máy 176.34.0.3), trước khi bạn đăng nhập ssh vào client.
Lệnh chạy:
``` bash
ssh-keygen -t ed25519 -N '' -f ~/.ssh/lab_ed25519 -C 'attacker@lab'

```
Kết quả mong đợi sẽ Có 2 file:

Private key: ~/.ssh/lab_ed25519 (giữ bí mật)

Public key: ~/.ssh/lab_ed25519.pub


Hiển thị public key (để copy dán sang client ở B2):
``` bash
cat ~/.ssh/lab_ed25519.pub
```

Kết quả là một dòng kiểu:
ssh-ed25519 AAAAC3NzaC1lZDI1<…..> attacker@lab

Kiểu dạng này nè : 

```bash 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB+GUwGnpsfzVyBqPobk/B9ZLEYK04xvUDjabo3bUK4O attacker@lab
```

##  b) Ở Terminal attacker, lúc này chưa dùng key, đăng nhập bằng mật khẩu:
Lúc này , attacker phải đăng nhập bằng mật khẩu đúng: 
```bash 
ssh ubuntu@176.34.0.7
```
### l1) Bật thư mục SSH cho user hiện tại
``` bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh

```
### l2) Thêm *nguyên* dòng public key bạn vừa xem ở bước A:
  Dán đúng một dòng "ssh-ed25519 AAAA... attacker@lab" vào giữa cặp nháy đơn  → NHÁY ĐƠN Ở CUỐI

``` bash
echo 'ssh-ed25519 AAAA attacker@lab' >> ~/.ssh/authorized_keys
```

### l3) Quyền file chặt chẽ
```bash 
ls -l ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Kết quả mong đợi:

```bash 
nl -ba ~/.ssh/authorized_keys
```

Thấy ít nhất 1 dòng chứa attacker@lab.

### l4) Xác nhận “cửa sau” hoạt động (từ attacker, dùng private key):
Giữ nguyên phiên này; mở cửa sổ terminal attacker khác.

Từ attacker ở phiên khác (terminal khác) chạy:
```bash 
ssh -i ~/.ssh/lab_ed25519 ubuntu@176.34.0.7
```

## c)  Tạo biến JOB sau khi attacker ssh thành công
Tiếp tục đóng vai trò là attacker : 
Đầu tiên , cài cron:
```bash 
sudo apt-get install cron -y
```
Bắt buộc :      
```bash
 sudo systemctl restart cron
sudo systemctl status cron
```
Rồi chạy lệnh sau: 
```bash 
JOB='* * * * * test -f "$HOME/.p_once" || { /usr/bin/logger -t persistence "persistence-once $(date +\%FT\%T) $(hostname) [cron]"; echo "persistence-once $(date +\%FT\%T) $(hostname) [cron]" >> /tmp/.once; touch "$HOME/.p_once"; }'
```
### l5) Nạp vào crontab
```bash 
( crontab -l 2>/dev/null; echo "$JOB" ) | crontab -
```
#Kiểm tra kết quả đã có cron chưa
```bash 
crontab -l
```
Ý nghĩa lệnh:
```bash 
test -f "$HOME/.p_once" 
```

— nếu file guard đã tồn tại thì không làm gì.

Nếu chưa có: logger tạo một dòng vào syslog; echo append vào /tmp/.once; touch tạo guard ~/.p_once.
### l6) Xác minh sau 1 phút : 

```bash 
ls -l /tmp/.once
```

```bash 
ls -l ~/.p_once
tail -n 5 /tmp/.once
```

```bash
sudo tail -n 80 /var/log/syslog | egrep -i 'persistence-once|CRON|persistence'
```


# (1.6) Trên server, phát hiện incident B ở công cụ elk và terminal server 
(pass)
Changed password for user elastic

PASSWORD elastic = ur36pN06X06KcKwpScnA

ví dụ :  curl -sS -u elastic:<password_elastic>  

## 1.6.1. SSH → “Accepted publickey?”
Khi tra KQL ở giao diện:

```bash 
host.name:"client" AND message:*sshd*
```
sau đó tra tiếp: 
```bash 
host.name:"client" AND (message:/Accepted/i OR message:"session opened for user")
```
Kết quả cần thấy: các dòng có chữ “Accepted …” hoặc “session opened for user …”.

Kết luận “Accepted publickey” CÓ/KO
```bash 
host.name:"client" AND message:"Accepted publickey"
```
Kết quả cần thấy:

CÓ: xuất hiện dòng kiểu Accepted publickey for ubuntu from <IP> …

KO: Count = 0
→ curl (đếm chính xác):
```bash 
curl -sS -u elastic:ur36pN06X06KcKwpScnA -H 'Content-Type: application/json' \
 'http://127.0.0.1:9200/lab-*/_count' -d '{
  "query": {"bool":{"must":[
    {"match_phrase":{"host.name":"client"}},
    {"match_phrase":{"message":"Accepted publickey"}},
    {"range":{"@timestamp":{"gte":"now-2h"}}}
  ]}}
}' | jq -c '{accepted_publickey: .count}'  | tee -a /home/ubuntu/evidence.json
```


Đọc kết quả:
hits.total.value >= 1 ⇒ CÓ Accepted publickey

= 0 ⇒ KHÔNG có Accepted publickey trong khoảng thời gian đang xem

## 1.6.2. CRON → “persistence-once?”
Khi tra KQL ở giao diện:
```bash 
host.name:"client" AND message:*CRON*
```
sau đó tra tiếp: 
```bash 
host.name:"client" AND message:*CRON* AND message:*CMD* AND NOT message:*run-parts*
```
Kết quả cần thấy: Các dòng giống CRON(...): (ubuntu) CMD (/path/to/script ...) — đây là lệnh cron thực tế được chạy.

Vì run-parts tạo rất nhiều bản ghi “ồn” (cron chạy định kỳ). Nếu mục tiêu là tìm những cron mâu thuẫn/không chuẩn (ví dụ attacker đặt cron manual gọi một script độc hại), bạn có thể bỏ qua các bản ghi chứa run-parts để tập trung trên các CMD khác (ví dụ CMD (/usr/bin/malicious)).

Kết luận“persistence-once”” CÓ/KO
```bash 
host.name:"client" AND message:"persistence-once"
```
Kết quả cần thấy:
CÓ: Dòng CRON (hoặc log khác) chứa “persistence-once”.
KO: Count = 0
curl (tìm đúng chuỗi)
```bash 
curl -sS -u elastic:ur36pN06X06KcKwpScnA -H 'Content-Type: application/json' \
 'http://127.0.0.1:9200/lab-*/_count' -d '{
  "query": {"bool":{"must":[
    {"match_phrase":{"host.name":"client"}},
    {"match_phrase":{"message":"persistence-once"}},
    {"range":{"@timestamp":{"gte":"now-2h","lte":"now"}}}
  ]}}
}' | jq -c '{persistence_once: .count}' | tee -a /home/ubuntu/evidence.json
```

# (1.7) Forensics : Lưu file nghi ngờ để làm forensic && Thu thập log
- Trên client (176.34.0.7):
```bash 
mkdir -p ~/ir-backup
cp -a ~/.ssh/authorized_keys ~/ir-backup/authorized_keys.bak.$(date +%s) 
crontab -l > ~/ir-backup/crontab.bak.$(date +%s) 
sha256sum ~/ir-backup/*  
```
- Mục đích: có bằng chứng/đối chiếu nếu cần.p'
- Nếu muốn kiểm tra xem nó chạy được hay không thì dùng câu lệnh như dưới: 
```bash 
cp -a ~/.ssh/authorized_keys ~/ir-backup/authorized_keys.bak.$(date +%s)   \
  && echo "[OK] Backup done." \
  || echo "[WARN] Not done yet."

```
- Nếu muốn xóa đi làm lại:    rm -rf ir-backup/* 
- Trên server : Thu thập log
```bash 
curl -sS -u elastic:ur36pN06X06KcKwpScnA \
  "http://127.0.0.1:9200/lab-*/_search?size=10000&pretty" \
  -H "Content-Type: application/json" \
  -d '{
        "query": {
          "bool": {
            "must": [
              { "match_phrase": { "message": "persistence-once" }},
              { "range": { "@timestamp": { "gte": "now-2h", "lte": "now" }}}
            ]
          }
        },
        "sort": [{"@timestamp": "asc"}]
      }' \
  | tee -a /home/ubuntu/evidence1.json
```

# (1.8) Eradication 1 : Gỡ backdoor key “attacker@lab”

```bash 
nl -ba ~/.ssh/authorized_keys
cp -a ~/.ssh/authorized_keys ~/.ssh/authorized_keys.bak
sed -i '/attacker@lab/d' ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
Kỳ vọng: file hiện tại đã bị xoá dòng attacker nếu có; có file ~/.ssh/authorized_keys.bak giữ bản cũ.

# (1.9) Eradication 2: Xóa cron “persistence-once” và guard files

```bash
sudo grep -REn --color 'persistence-once|\.p_once' /etc/cron* /var/spool/cron
```
```bash 
crontab -l | sed '/persistence-once/d' | crontab -
rm -f ~/.p_once /tmp/.once
```
```bash
sudo systemctl restart cron
```
```bash 
crontab -l && echo "[OK] User crontab found." || echo "[WARN] No user crontab"
```


# (1.10) Eradication 3: Loại bỏ đường xâm nhập ban đầu của attacker

```bash 
NEWPASS="$(openssl rand -hex 16)"
echo "New 32-hex password: $NEWPASS"
echo "ubuntu:${NEWPASS}" | sudo chpasswd
```
- Hoặc tự đặt mật khẩu : 
```bash 
NEWPASS="Matkhaukho1!"
echo "New password will be: $NEWPASS"
echo "ubuntu:${NEWPASS}" | sudo chpasswd
```

# (1.11) Post-Incident Monitoring :Theo dõi hậu sự cố “incident A”
## A) Cấu hình lại logstash
- Sửa /etc/logstash/conf.d/lab.conf để thêm filter (giữ nguyên input/output hiện có):
```bash 
input {
  beats { port => 5044 }
}
filter {
  if "sshd" in [message] {
    grok {
      match => {
        "message" => [
          "Failed password for %{DATA:user.name} from %{IP:source.ip} port %{NUMBER} ssh2",
          "Invalid user %{DATA:user.name} from %{IP:source.ip}",
          "authentication failure%{DATA} rhost=%{IP:source.ip}%{GREEDYDATA}"
        ]
      }
      tag_on_failure => []
    }
    mutate { add_field => { "event.category" => "authentication" } }
  }
}
output {
  elasticsearch {
    hosts => ["http://127.0.0.1:9200"]
    index => "lab-%{+YYYY.MM.dd}"
  }
  # stdout { codec => rubydebug }  # tùy bật để debug
}
```
- Rồi sau đó khởi động lại:
```bash
sudo systemctl restart logstash
sudo systemctl status logstash --no-pager
sudo ss -lnt 'sport = :9200 or sport = :5601 or sport = :5044'  
```
```bash 
http://176.34.0.5:5601  
```

## B) Trong giao diện UI, tạo alert
Vào tab Security → Rules → Create new rule → chọn Threshold (đừng chọn Query).##
### mục Defination :
  + Index patterns: lab-*   (đã có trước đó và bắt dùng)
  + Custom query:
KQL:
```bash 
host.name:"client" and message:(sshd and ("Failed password" or "Invalid user" or "authentication failure"))
```
  + Group by:
```bash 
source.ip.keyword
```
Threshold: is ≥ 1 within 1 minute (tùy đổi ngưỡng).

### About: 
  + Name:
```bash 
SSH brute-force failed (by IP)
```
### Schedule: 
  + Schedule: Run every 2 minute, Look back 1 minutes.

### Actions: trong môi trường của bạn chỉ có “Index” → dùng luôn: Log
  + Action Frequency: On each rule execution
Chọn Index → tạo connector mới:
  + Connector name:
```bash 
SSH brute-force alerts
```
  + Index:
```bash 
ir_ssh_alerts
```
  + Document to index:
```bash 
{"@timestamp":"{{date}}","rule":"{{rule.name}}","results":"{{context.results}}"}
```
→ Save → Create & enable rule.

Lúc này vào alert ở mục Security sẽ có: (làm 3 lần mới ra ạ =)))) ở alert thì chọn signal.rule.risk_score.
- Sau đó kiểm tra: 
```bash 
curl -sS -u elastic:ur36pN06X06KcKwpScnA \
'http://127.0.0.1:9200/.siem-signals-*/_search' \
-H 'Content-Type: application/json' -d '{
  "size": 0,
  "query": { "bool": { "filter": [
    { "term": { "signal.rule.name.keyword": "SSH brute-force failed (by IP)" } },
    { "term": { "signal.status": "open" } },
    { "range": { "@timestamp": { "gte": "now-1h", "lte": "now" } } }
  ]}},
  "aggs": { "ips": { "terms": { "field": "signal.threshold_result.terms.value.keyword", "size": 50 } } }
}'  | jq --arg time "$(date -Iseconds)" \
  '{rule:"SSH brute-force failed (by IP)",
    alert:.hits.total.value,
    attackers:(.aggregations.ips.buckets|map(.key)),
    time:$time }' | tee -a /home/ubuntu/evidence.json
```

### Nếu lỗi:===================================================
```bash 
curl -sS -u elastic:ur36pN06X06KcKwpScnA -X DELETE 'http://127.0.0.1:9200/.siem-signals-*'
```
Cái này dùng kiểm tra:
```bash 
curl -sS -H 'Content-Type: application/json' \
     -u elastic:ur36pN06X06KcKwpScnA\
'http://127.0.0.1:9200/lab-*/_search' -d '{
  "size":0,
  "query":{"bool":{"filter":[
    {"match_phrase":{"host.name":"client"}},
    {"query_string":{"default_field":"message","query":"sshd AND (Failed OR Invalid OR \"authentication failure\")"}},
    {"range":{"@timestamp":{"gte":"now-5m","lte":"now"}}}
  ]}},
  "aggs":{"by_ip":{"terms":{"field":"source.ip","size":10}}}
}'
```






