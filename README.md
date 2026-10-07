Bài 4: Phân tích Sự cố Mạng và Kiểm tra Cổng Dịch vụ (Troubleshooting Network & Ports)
Mục tiêu
Sử dụng công cụ chẩn đoán mạng CLI (ping, curl, netstat/ss, traceroute/mtr) để phân tích nguyên nhân sự cố mất kết nối hoặc phản hồi chậm.
Phân tích sự cố dịch vụ web phản hồi mã lỗi 502 Bad Gateway hoặc ngắt kết nối.
1. Kịch bản Phân tích Sự cố Mạng & Cổng Dịch vụ
Sự cố: Nginx Web Server phản hồi lỗi 502 Bad Gateway khi người dùng truy cập cổng 80.
2. Các Bước Chẩn đoán & Lệnh Kiểm tra CLI
Bước 1: Kiểm tra kết nối ICMP & HTTP căn bản
# Kiểm tra độ trễ mạng
ping -c 4 103.x.x.x

# Kiểm tra phản hồi HTTP Header
curl -I http://localhost
Kết quả curl -I:

HTTP/1.1 502 Bad Gateway
Server: nginx/1.18.0 (Ubuntu)
Content-Type: text/html
Content-Length: 157
Bước 2: Kiểm tra danh sách các cổng đang LẮNG NGHE (ss -tuln / netstat -tuln)
sudo ss -tuln
Kết quả hiển thị:

Netid  State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port  Process
tcp    LISTEN  0       511            0.0.0.0:80          0.0.0.0:*
tcp    LISTEN  0       4096           0.0.0.0:22          0.0.0.0:*
Phân tích: Cổng 80 (Nginx) đang lắng nghe, nhưng cổng 8082 (Backend Spring Boot app) KHÔNG xuất hiện trong danh sách Lắng nghe.

Bước 3: Kiểm tra nhật ký lỗi Nginx (/var/log/nginx/error.log)
sudo tail -n 20 /var/log/nginx/error.log
Kết quả Log xuất hiện:

2026/10/07 10:50:00 [error] 1500#1500: *1 connect() failed (111: Connection refused) while connecting to upstream, client: 127.0.0.1, server: _, request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8082/", host: "localhost"
3. Nguyên nhân Root Cause & Giải pháp
Nguyên nhân:
Dịch vụ Backend (Spring Boot running on port 8082) bị sập hoặc chưa được khởi động, khiến Nginx Reverse Proxy không thể chuyển tiếp request (upstream connection refused), dẫn đến mã lỗi 502 Bad Gateway.

Giải pháp khắc phục:
Khởi động lại dịch vụ Backend App:

sudo systemctl start spring-app
Kiểm tra lại cổng 8082 và truy cập thành công:

$ sudo ss -tuln | grep 8082
tcp    LISTEN  0       100          0.0.0.0:8082         0.0.0.0:*

$ curl -I http://localhost
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
