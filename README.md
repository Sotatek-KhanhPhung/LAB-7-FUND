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
- Sửa file config 
**NOTE**: Copy file config gốc sang 1 file khác
```sh
sudo nano /etc/redis/redis.conf
```
    - RDB Snapshot (Redis Database Snapshot)
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
## 2. Backup Restore trên cùng 1 server
- Bước 1: Cài đặt Redis và check kết nối đến Redis
```sh
sudo apt update
sudo apt install redis-server
sudo systemctl status redis
```
```sh
redis-cli ping
```

- Bước 2: Sửa file config của Redis
```sh
# Trong 900 giây nếu có ít nhất 1 key thay đổi, tạo snapshot
save 900 1
# Trong 300 giây nếu có ít nhất 10 key thay đổi, tạo snapshot
save 300 10
# Trong 60 giây nếu có ít nhất 10000 key thay đổi, tạo snapshot
save 60 10000
# File lưu snapshot
dbfilename dump.rdb
# Thư mực chứa file RDB
dir /var/lib/redis
```
    - Sau khi thay đổi cấu hình, restart Redis và check status
```sh
sudo systemctl restart redis
sudo systemctl status redis
```

- Bước 3: Tạo dữ liệu để test backup
    - Truy cập vào ```sh redis-cli ```
```sh
FLUSHALL
SET user:1 'Khanh'
SET user:2 'Khai'
SET user:3 'Mike'
SET user:4 'Tin'
LPUSH tasks 'task1' 'task2' 'task3' 'task4' 'task5'
INCR counts
```

    - Tạo snapshot
```sh
BGSAVE
```

    - Lưu file RDB sang thư mục backup
```sh
sudo mkdir -p /var/backups/redis
sudo chown redis:redis /var/backups/redis

sudo cp /var/lib/redis/dump.rdb /var/backups/redis/dump-$(date +%F-%H%M%S).rdb
```

- Bước 4: Test mất dữ liệu và restore trên cùng server
    - Xoá tất cả dữ liệu bằng lệnh ```sh FLUSHALL ``` rồi check xem còn key nào tồn tại không ```sh KEYS * ```
    - Stop Redis
```sh
sudo systemctl stop redis
```
    - Copy lại file dump từ thư mục Backups vào đúng vị trí rồi đổi lại tên thành dumb.rdb
```sh
cp /var/backups/redis/dumb_file_name /var/lib/redis/dumb.rdb
```
    Khời động lại Redis và kiểm tra dữ liệu
```sh
sudo systemctl start redis
```
```sh
redis-cli

KEYS *
```

## 3. Backup Restore trên 2 server và Master Slave với Sentinel
