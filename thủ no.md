# ĐỀ TÀI: Nghiên cứu và xây dựng các bài thực hành Labtainer về kỹ thuật phát hiện và phản ứng sự cố mạng

---

## MỤC LỤC

- [Lời nói đầu](#loi-noi-dau)
- [Lời cảm ơn](#loi-cam-on)
- [Lời cam đoan](#loi-cam-doan)
- [Danh mục hình vẽ, bảng biểu](#danh-muc-hinh-bang)
- [Chương 1. Tổng quan về phát hiện và phản ứng sự cố mạng](#chuong-1)
  - [1.1. Khái niệm về sự cố an ninh mạng](#1-1)
  - [1.2. Quy trình phát hiện và phản ứng sự cố](#1-2)
  - [1.3. Nhu cầu đào tạo thực hành](#1-3)
- [Chương 2. Môi trường Labtainer và thiết kế bài thực hành](#chuong-2)
  - [2.1. Giới thiệu Labtainer](#2-1)
  - [2.2. Kiến trúc container và mạng trong Labtainer](#2-2)
  - [2.3. Cách xây dựng một bài lab mới](#2-3)
- [Chương 3. Xây dựng các bài thực hành phát hiện và phản ứng sự cố](#chuong-3)
  - [3.1. Lab 1 – Tấn công SSH brute-force và persistence](#3-1)
  - [3.2. Lab 2 – ICMP flood và phát hiện bất thường](#3-2)
  - [3.3. Lab 3 – Web directory brute-force và log Apache](#3-3)
  - [3.4. Lab 4 – Port scanning và giám sát trên Splunk](#3-4)
- [Chương 4. Đánh giá, kết luận và hướng phát triển](#chuong-4)
  - [4.1. Đánh giá kết quả đạt được](#4-1)
  - [4.2. Hạn chế](#4-2)
  - [4.3. Hướng phát triển](#4-3)

---

<a name="loi-noi-dau"></a>
<details open>
  <summary><strong>Lời nói đầu</strong></summary>

Viết lời mở đầu ở đây...

</details>

---

<a name="loi-cam-on"></a>
<details>
  <summary><strong>Lời cảm ơn</strong></summary>

Nội dung lời cảm ơn...

</details>

---

<a name="loi-cam-doan"></a>
<details>
  <summary><strong>Lời cam đoan</strong></summary>

Nội dung lời cam đoan...

</details>

---

<a name="danh-muc-hinh-bang"></a>
<details>
  <summary><strong>Danh mục hình vẽ, bảng biểu</strong></summary>

Liệt kê hình/bảng nếu cần.

</details>

---

<a name="chuong-1"></a>
<details open>
  <summary><strong>Chương 1. Tổng quan về phát hiện và phản ứng sự cố mạng</strong></summary>

<a name="1-1"></a>
<details>
  <summary><strong>1.1. Khái niệm về sự cố an ninh mạng</strong></summary>

### 1.1.1. Sự cố an ninh mạng là gì?

Tôi là text ví dụ...

### 1.1.2. Phân loại sự cố

Tôi là text2...

</details>

<a name="1-2"></a>
<details>
  <summary><strong>1.2. Quy trình phát hiện và phản ứng sự cố</strong></summary>

### 1.2.1. Các giai đoạn: Preparation, Detection, Containment, Eradication, Recovery

Nội dung...

### 1.2.2. Ví dụ kịch bản sự cố

Nội dung...

</details>

<a name="1-3"></a>
<details>
  <summary><strong>1.3. Nhu cầu đào tạo thực hành</strong></summary>

### 1.3.1. Khó khăn của sinh viên mới

Nội dung...

### 1.3.2. Vai trò của bài lab mô phỏng

Nội dung...

</details>

</details>

---

<a name="chuong-2"></a>
<details>
  <summary><strong>Chương 2. Môi trường Labtainer và thiết kế bài thực hành</strong></summary>

<a name="2-1"></a>
<details>
  <summary><strong>2.1. Giới thiệu Labtainer</strong></summary>

### 2.1.1. Mục tiêu của Labtainer

Nội dung...

### 2.1.2. Ưu điểm so với lab truyền thống

Nội dung...

</details>

<a name="2-2"></a>
<details>
  <summary><strong>2.2. Kiến trúc container và mạng trong Labtainer</strong></summary>

### 2.2.1. Mô hình nhiều container (server, client, attacker)

Nội dung...

### 2.2.2. Cấu hình mạng và dải IP

Nội dung...

</details>

<a name="2-3"></a>
<details>
  <summary><strong>2.3. Cách xây dựng một bài lab mới</strong></summary>

### 2.3.1. Cấu trúc thư mục của một lab

Nội dung...

### 2.3.2. Cấu hình chấm điểm tự động (results.config, treataslocal)

Nội dung...

</details>

</details>

---

<a name="chuong-3"></a>
<details>
  <summary><strong>Chương 3. Xây dựng các bài thực hành phát hiện và phản ứng sự cố</strong></summary>

<a name="3-1"></a>
<details>
  <summary><strong>3.1. Lab 1 – Tấn công SSH brute-force và persistence</strong></summary>

### 3.1.1. Mục tiêu học tập

Nội dung...

### 3.1.2. Mô hình lab (server, client, attacker)

Nội dung...

### 3.1.3. Kịch bản tấn công và thu thập log (ELK/Splunk)

Nội dung...

</details>

<a name="3-2"></a>
<details>
  <summary><strong>3.2. Lab 2 – ICMP flood và phát hiện bất thường</strong></summary>

### 3.2.1. Mục tiêu học tập

Nội dung...

### 3.2.2. Cấu hình Suricata/iptables/log

Nội dung...

</details>

<a name="3-3"></a>
<details>
  <summary><strong>3.3. Lab 3 – Web directory brute-force và log Apache</strong></summary>

### 3.3.1. Mục tiêu học tập

Nội dung...

### 3.3.2. Xây dựng web mồi (admin, backup, phpmyadmin)

Nội dung...

### 3.3.3. Phát hiện brute-force qua log và dashboard

Nội dung...

</details>

<a name="3-4"></a>
<details>
  <summary><strong>3.4. Lab 4 – Port scanning và giám sát trên Splunk</strong></summary>

### 3.4.1. Mục tiêu học tập

Nội dung...

### 3.4.2. Tạo log và index `portscan_lab`

Nội dung...

### 3.4.3. Dashboard/alert phát hiện portscan

Nội dung...

</details>

</details>

---

<a name="chuong-4"></a>
<details>
  <summary><strong>Chương 4. Đánh giá, kết luận và hướng phát triển</strong></summary>

<a name="4-1"></a>
<details>
  <summary><strong>4.1. Đánh giá kết quả đạt được</strong></summary>

Tổng hợp lại các kết quả chính đạt được của đề tài…

</details>

<a name="4-2"></a>
<details>
  <summary><strong>4.2. Hạn chế</strong></summary>

Các hạn chế về:
- Phạm vi lab
- Môi trường triển khai
- Trải nghiệm người học

</details>

<a name="4-3"></a>
<details>
  <summary><strong>4.3. Hướng phát triển</strong></summary>

Đề xuất:
- Mở rộng thêm lab (DNS spoofing, malware, v.v.)
- Tích hợp với hệ thống LMS
- Tự động hóa đánh giá nâng cao

</details>

</details>
