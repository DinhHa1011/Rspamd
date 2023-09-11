# Rspamd connect to  other server
# Table of contents
## Rspamd Server

- Config cổng 11334 (worker_controller) của Rspamd chạy public
    + Mở file
    ```
    vim /etc/rspamd/rspamd.conf
    ```
    + Tìm phần   
    ```
    ...
    worker "controller" {
        bind_socket = "localhost:11334";
    ...
    ```
    + Sửa  
    ```
    worker "controller" {
        bind_socket = "*:11334";

    ```
- Thêm IP của IMAP server vào danh sách IP trusted. 
    + Mở file 
    ```
    vim /etc/rspamd/worker-controller.inc
    ```
    + Thêm dòng 
        ```
        secure_ip = "<ip_imap_server";
        ```
        + Ví dụ 
            ```
            secure_ip = "45.124.93.39";
            ```
- Restart rspamd để apply cấu hình 
```
systemctl restart rspamd
```
- Check 
    - Command 
    ``` 
    lsof -i -n -P | grep rspamd
    ```
    + Check log có dòng `*11334` là OK 
    ```
    rspamd    1363413         _rspamd   78u  IPv4 13321847      0t0  TCP *:11334 (LISTEN)
    ```

## IMAP Server

### Bayes Learn Spam/Ham
- Chỉnh sửa sieve script 
    + Chuyển tới thư mục lưu sieve script
    ```
    cd /var/mail/vmail/sieve/global
    ```
    + Sửa file sieve learn spam
        + Mở file 
        ```
        vim report_spam.sieve
        ```
        + Comment dòng để learn localhost
        ```
        pipe :copy "rspamc"  ["learn_spam"];
        ```
        + Thêm dòng learn để sang server khác 
        ```
        pipe :copy "learnspam_external.sh";
        ```
    + Sửa file sieve learn ham
        + Mở file 
        ```
        vim report_ham.sieve
        ```
        + Comment dòng để learn localhost
        ```
        pipe :copy "rspamc"  ["learn_ham"];
        ```
        + Thêm dòng learn để sang server khác 
        ```
        pipe :copy "learnham_external.sh";
        ```
    + Compile sieve script
        ```
        sievec report_spam.sieve
        sievec report_ham.sieve
        ```
- Tạo file bash run lệnh learn
    + Chuyển đến thư mục lưu file bash
    ```
    cd /usr/bin
    ```
    + Tạo file learnspam
        + Tạo file 
        ```
        nano learnspam_external.sh
        ```
        + Thêm dòng
        ```
        #!/bin/bash
        exec rspamc -h <IP_Server_RSPAMD>:11334 learn_spam
        ```

    + Tạo file learnspam
        + Tạo file 
        ```
        nano learnham_external.sh
        ```
        + Thêm dòng
        ```
        #!/bin/bash
        exec rspamc -h <IP_Server_RSPAMD>:11334 learn_ham
        ```
+ Change owner cho u:g `vmail:vmail` để dovecot có quyền run  
```
chown vmail:vmail learnspam_external.spam
chown vmail:vmail learnham_external.spam
```
### Fuzzy 
- Chuyển đến thư mục lưu file bash 
```
cd /usr/bin
```
- Sửa file `fuzzy_denied.sh`
    + Mở file 
    ```
    vim fuzzy_denied.sh
    ```
    + Comment dòng
    ```
    exec rspamc -f 11 -w 1 fuzzy_add
    ```
    + Thêm dòng
    ```
    exec rspamc -h <IP_Server_RSPAMD>:11334 -f 11 -w 1 fuzzy_add
    ```
- Sửa file `fuzzy_whitelist.sh`
    + Mở file
    ```
    vim fuzzy_whitelist.sh
    ```
    + Comment dòng
    ```
    exec rspamc -f 13 -w 1 fuzzy_add
    ```
    + Thêm dòng
    ```
    exec rspamc -h <IP_Server_RSPAMD>:11334 -f 13 -w 1 fuzzy_add
    ```
## References
* [RSPAMC](https://manpages.debian.org/testing/rspamd/rspamc.1.en.html#h)
* [Controller Worker](https://rspamd.com/doc/workers/controller.html)
