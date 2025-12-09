# LAB-7-FUND

Yêu cầu
- 2 server Ubuntu 22.04
- Master: 192.168.215.160
- Replica: 192.168.215.161

## 1. Redis Deployment Guide
### 1.1. Cài đặt Redis
```sh
sudo apt update
sudo apt install redis-server

# Check version of Redis
redis-server --version

# Start Redis and check status
sudo systemctl enable redis
sudo systemctl start redis
sudo systemctl status redis
```

### 1.2. Set up Password để xác thực kết nối
- Chỉnh sửa file config của Redis
```sh
sudo nano /etc/redis/redis.conf

requirepass your_password
```
- Có 2 cách để truy cập vào Redis CLI
```sh
redis-cli -a your_password 
```
Hoặc
```sh
redis-cli

AUTH your_password
```

### 1.3. Cài đặt kiểu lưu dữ liệu (AOF / RDB)
- RDB Snapshot (Redis Database Snapshot)
    - Sửa file config 
```sh
sudo nano /etc/redis/redis.conf
```
```sh
save 3600 1		        #Trong 3600 giây nếu có ít nhất 1 key thay đổi, lưu toàn bộ db
save 300 100	        #Trong 300 giây nếu có ít nhất 100 key thay đổi, lưu toàn bộ db 
save 60 10000	        #Trong 60 giây nếu có ít nhất 10000 key thay đổi, lưu toàn bộ db
dbfilename dump.rdb	    #Tên file RDB
dir /var/lib/redis	    #Vị trí lưu file RDB
```
- AOF (Append-only File)
```sh
appendonly yes				        #Bật tính năng lưu dữ liệu kiểu AOF
appendfsync everysec			    #Thiết lập tần suất ghi file AOF
appendfilename “appendonly.aof”	    #Tên file AOF 
```


