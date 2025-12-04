```bash
sudo mysql -u root -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY ''; FLUSH PRIVILEGES;"
sudo mysql -u root < ./src/users_account.sql
sudo cp ./src/html/* /var/www/html/
```


thử 

```bash

```bash


```

```bash

2.2 Xây dựng và quản lý bài thực hành trên Labtainer
2.2.1 Cấu trúc bài thực hành
Nền tảng thực hành số cung cấp chức năng tạo bài thực hành dựa trên chương trình Lab Editor của Labtainer. Chương trình này cung cấp giao diện đồ họa để tạo bài thực hành. Từ đường dẫn bất kỳ trên terminal gõ lệnh labedit nếu không mở được giao diện, như hình vẽ:
 Câu lệnh khởi động labedit
Cần update file update-designer.sh (sử dụng lệnh ./update-designer.sh để
update)
 
File update-designer.sh dùng để cập nhật
Giao diện sau khi update sử dụng lệnh labedit để hiển thị giao diện:
  Giao diện của Labtainer
Từ trên thanh điều khiển chọn mục File -> New Lab, đặt tên bài lab
 
- Đặt tên cho bài lab
Xây dựng bài lab mới gồm 2 container gồm client và server với địa chỉ IP của 2 máy:
-	Client: 172.30.0.10
-	Server: 172.30.0.20
Bài lab mới gồm 2 yêu cầu:
	Sử dụng ssh từ client đến server
	Sử dụng ping từ server đến client
-	Đổi tên container gitlab bằng cách chuột phải vào phần Containers gitlab chọn
Rename, chuyển demo-lab    client.
 
Giao diện sau khi đã đổi tên bài lab
2.2.2 Mô hình container và cấu hình mạng cho bài thực hành
Tạo container thứ 2 với tên server bằng cách Add ở mục Containers
 
- Tạo container trong Labtainer
Cấu hình mạng cho bài lab
Cấu hình mạng cho bài Lab, ở phần Networks chọn Add, cấu hình chi tiết mạng bằng cách điền các thông tin mạng.
 
- Cấu hình dải mạng cho bài lab
Cấu hình mạng cho client và cấu hình mạng tương tự với server
 
- Cấu hình mạng cho client
-	Cấu hình file dockerfile, qua đường dẫn labtainer -> trunk -> labs -> demo-lab2 ->dockerfiles trong folder gồm các file ở dạng: Dockerfile. <tên bài lab>.<têncontainer>.student
 
- Cấu hình Dockerfile cho client
Cấu hình tương tự với container server:
 
- Cấu hình Dockerfile cho server
2.2.3 Cấu hình chấm điểm tự động cho bài thực hành
Cấu hình file result
 
- Cấu hình file result
 
- Thực thi lệnh trên client
 
- Thực thi lệnh trên server
Kết quả chương trình
 
- Kết quả chương trình
2.2.4 Quản lý và lưu trữ bài thực hành
Lưu trữ bài lab
Tạo tài khoản trên trang https://hub.docker.com trên giao diện labtainer chọn mục Edit/Config (registry) đănh nhập tên tài khoản hiển thị trên dockerhub và Build only lại.
 
- Đăng nhập trong Labtainer
-	Khởi tạo quá trình lưu trữ trên dockerhub từ đường dẫn ~labtainer/trunk/labs với lệnh: 	git init
 
- Khởi tạo quá trình lưu trữ trên dockerhub
-	Khai báo các thông số để kết nối đến dockerhub với lệnh:
git config --global user.name “your_username”
git config --global user.email “your_email@example.com”
 
- Khai báo thông tin tài khoản
-	Thêm bài lab muốn lưu trữ vào git theo lệnh:
git add <tên bài lab>
git commit <tên bài lab> -m “Adding an Imodule”
 
- Thêm bài lab muốn lưu trữ
-	Chuyển đến thư mục ~trunk/distrib, sử dụng lệnh docker login để đăng nhập docker trên Labtainer
 
- Đăng nhập tài khoản dockerhub
-	Đẩy bài lab lên dockerhub bằng cách sử dụng lệnh: 
./publish.py -d -l <tên bài lab>
 
- Quá trình tải bài lab lên dockerhub
-	Quá trình thành công các bài lab đã được đẩy lên quản lý trên docker hub
 
- Lưu trữ bài lab trên dockerhub
Github
-	https://github.com/hoangkienlc02/demo-lab2
-	Nhập lệnh create-imodules.sh
 
- Tạo file Imodule.tar
Lấy các bài lab được lưu trữ về
-	Sinh viên tải bài lab theo đường dẫn cung cấp và kéo vào môi trường máy ảo để thực hiện
 
- File imodule.tar chứa bài thực hành
-	Tạo repo mới để đẩy imodule.tar lên và tạo phần release mới
 
- Đẩy file imodule.tar lên github
-	Sinh viên tiến hành tải bài lab theo đường dẫn được cung cấp theo lệnh
wget <đường dẫn>
 
- Tải bài lab về

```
