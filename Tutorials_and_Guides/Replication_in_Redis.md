## Bayesian statistics and fuzzy storage replication with multi-instance Redis backend
- Tutorial này trình bày hướng dẫn từng bước về cách thiết lập số liệu thống kê vả fuzzy storage replication trên FreeBSD. Quy trình config cho các hệ điều hành khác khá giống nhau.
- Tutorial tập trung vào mô hình centralized trong đó quá trình học phân loại bayes và fuzzy storage diễn ra trên một host duy nhất và sau đó được phân phối giữa các cài đặt rspamd ở remote locations. Để đơn giản, hướng dẫn này cover việc sao chép tới một `replica` database duy nhất cho mỗi `masters`
- Để đạt được điều này, chúng ta cần replicate bayes và fuzzy storage backend data để remote host. 
Ví chúng ta không muốn sao chép toàn bộ cache redis nên chúng ta nên sử dụng các phiên bản redis chuyên dụng. Nó sẽ là khôn ngoan nếu tách biệt bayes và fuzzy storage.
- Chúng ta sẽ tạo 3 phiên bản Redis trên cả `master` và `replica`: `bayes`, `fuzzy`, `redis` cho cache còn lại
![image](https://github.com/DinhHa1011/Rspamd/assets/119484840/79b73dbd-c2a5-41e1-b9e2-92ea5d5e85f6)
## Installation
- Để bắt đầu, install `databases/redis` package bằng cách thực hiện command:
```
pkg install redis
```
- Tiếp theo, tạo thư mục riêng cho từng trường hợp
```
cd /var/db/redis && mkdir bayes fuzzy && chown redis bayes fuzzy
```
- Để enable redis và trường hợp cụ thể của nó, add dòng dưới đây vào file `/etc/rc.conf`
```
redis_enable="YES"
redis_profiles="redis bayes fuzzy"
```
- Để enable log rotation cho Redis, tạo một newsyslog configuration file named `/usr/local/etc/newsyslog.conf.d/redis.newsyslog.conf`:
```
logfilename          [owner:group]    mode count size when  flags [/pid_file] [sig_num]
/var/log/redis/redis.log    redis:redis    644  5       100    *  J
/var/log/redis/bayes.log    redis:redis    644  5       100    *  J
/var/log/redis/fuzzy.log    redis:redis    644  5       100    *  J
```
## Configuration
- Tạo default config trên cả `master` và `replica` host, điều này sẽ phổ biến cho mọi trường hợp:
```
cp /usr/local/etc/redis.conf.sample /usr/local/etc/redis.conf
```
- Do lo ngại về bảo mật, không disable để exposse Redis to external interfaces. Thay vì, config Redis để listen trên loopback interfaces và sử dụng stunnel để thiết lập TLS tunnels giữa `replica` và `master` hosts. Tuy nhiên, phương pháp này cũng có lỗ hổng bảo mật riêng. Do đó, nó set up replication nếu bạn không thể tin tưởng user của replica host
- Config listening socket và memory limit (optional) như dưới:
```
# diff -u1 /usr/local/etc/redis.conf.sample /usr/local/etc/redis.conf
--- /usr/local/etc/redis.conf.sample    2016-11-03 06:30:49.000000000 +0300
+++ /usr/local/etc/redis.conf   2016-11-27 13:10:43.671584000 +0300
@@ -60,3 +60,3 @@
 # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
-bind 127.0.0.1
+bind 127.0.0.1 ::1

@@ -537,2 +537,3 @@
 # maxmemory <bytes>
+maxmemory 200M
```
- Config redis trên cả `master` và `replica` hosts trên một way chứa khả năng tương thích với cấu hình phiên bản riêng. Điều này đảm bảo rằng nếu bạn thực sự có một instance database riêng, nó sẽ tiếp tục để hoạt động đúng cách.
```
/usr/local/etc/redis-redis.conf
```
```
include /usr/local/etc/redis.conf
```
## Master instances configuration
```
/usr/local/etc/redis-bayes.conf
```
```
include /usr/local/etc/redis.conf

port 6378

pidfile /var/run/redis/bayes.pid
logfile /var/log/redis/bayes.log
dbfilename bayes.rdb
dir /var/db/redis/bayes/

maxmemory 600M
```
