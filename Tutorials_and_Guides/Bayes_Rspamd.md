# Bayes Spam Autolearn |  Rspamd & Dovecot
# Table of contents

- [Bayes Spam Autolearn |  Rspamd & Dovecot](#bayes-spam-autolearn---rspamd--dovecot)
  - [Update](#update)
  - [Install](#install)
  - [Config RSPAMD](#config-rspamd)
  - [Config Dovecot](#config-dovecot)
  - [Check Log](#check-log)
  - [References](#references)

## Update
```
sudo apt update
```
## Install 
```
sudo apt install dovecot-sieve dovecot-managesieved
```
## Config RSPAMD
- Decrease min learn limit 
    + Mở file 
    ```
    vim /etc/rspamd/statistic.conf
    ```
    + Sửa dòng `min_learns = 200` về giá trị thấp hơn ví dụ `min_learns = 7`
## Config Dovecot
- Enable plugins
    + LMTP 
    ```
    vim /etc/dovecot/conf.d/20-lmtp.conf
    ```

    ```
    protocol lmtp {
      mail_plugins = $mail_plugins sieve
    }
    ```
    + IMAP
    ```
    vim /etc/dovecot/conf.d/20-imap.conf
    ```

    ```
    protocol imap {
      mail_plugins = $mail_plugins imap_quota imap_sieve
    }
    ```

- Manage Sieve
```
vim /etc/dovecot/conf.d/20-managesieve.conf
```
```
service managesieve-login {
  inet_listener sieve {
    port = 4190
  }
}

service managesieve {
  process_limit = 1024
}
```
- Sieve
```
vim /etc/dovecot/conf.d/90-sieve.conf
```

```
plugin {
    #sieve = file:~/sieve;active=~/.dovecot.sieve
    sieve_plugins = sieve_imapsieve sieve_extprograms
    sieve_before = /var/mail/vmail/sieve/global/spam-global.sieve
    sieve = file:/var/mail/vmail/sieve/%d/%n/scripts;active=/var/mail/vmail/sieve/%d/%n/active-script.sieve
    imapsieve_mailbox1_name = Spam
    imapsieve_mailbox1_causes = COPY
    imapsieve_mailbox1_before = file:/var/mail/vmail/sieve/global/report-spam.sieve
    imapsieve_mailbox2_name = *
    imapsieve_mailbox2_from = Spam
    imapsieve_mailbox2_causes = COPY
    imapsieve_mailbox2_before = file:/var/mail/vmail/sieve/global/report-ham.sieve
    sieve_pipe_bin_dir = /usr/bin
    sieve_global_extensions = +vnd.dovecot.pipe
}
```

- Write Sieve Filters 
    + Create folder
    ```
    mkdir -p /var/mail/vmail/sieve/global
    ```
    + Sieve để di chuyển các email bị đánh dấu là SPAM vào thư mục SPAM:
        + Mở file 
        ```
        vim /var/mail/vmail/sieve/global/spam-global.sieve
        ```
        + Nội dung 
        ```
        require ["fileinto","mailbox"];

        if anyof(
            header :contains ["X-Spam-Flag"] "YES",
            header :contains ["X-Spam"] "Yes",
            header :contains ["Subject"] "*** SPAM ***"
            )
        {
            fileinto :create "Spam";
            stop;
        }
        ```
    + Sieve chạy khi imap action move mail vào SPAM
        + Mở file
        ```
        vim /var/mail/vmail/sieve/global/report-spam.sieve
        ```
        + Nội dung : Pipe copy mail sang learn_spam của rspamd
        ```
        require ["vnd.dovecot.pipe", "copy", "imapsieve"];
        pipe :copy "rspamc" ["learn_spam"];
        ```
    + Sieve chạy khi imap action move mail ra khỏi SPAM
        + Mở file
        ```
        vim /var/mail/vmail/sieve/global/report-ham.sieve
        ```
        + Nội dung: Pipe copy mail sang learn_ham của rspamd
        ```
        require ["vnd.dovecot.pipe", "copy", "imapsieve"];
        pipe :copy "rspamc" ["learn_ham"];
        ```
- Restart Dovecot
```
sudo systemctl restart dovecot
```
- Compile Sieve Scripts 
```
sievec /var/mail/vmail/sieve/global/spam-global.sieve
sievec /var/mail/vmail/sieve/global/report-spam.sieve
sievec /var/mail/vmail/sieve/global/report-ham.sieve
```
- Cấp quyền cho thư mục chứa Sieve Filters
```
sudo chown -R vmail: /var/mail/vmail/sieve/
```
## Check Log
- Command
```
tail -f /var/log/rspamd/rspamd.log
```
- Learn Spam 
```
2023-09-08 00:18:36 #3892998(rspamd_proxy) <de4623>; proxy; proxy_accept_socket: accepted milter connection from 127.0.0.1 port 40222
2023-09-08 00:18:41 #3892999(controller) <9f5d12>; csession; rspamd_controller_check_password: allow unauthorized connection from a trusted IP 127.0.0.1
2023-09-08 00:18:41 #3892999(controller) <9f5d12>; csession; rspamd_message_parse: loaded message; id: <aba30f57-ea42-4413-81fe-d2b12e1f6724@anthanh264.id.vn>; queue-id: <undef>; size: 184445; checksum: <8469e25782e3fb573cff3088525bd481>
2023-09-08 00:18:41 #3892999(controller) <9f5d12>; csession; rspamd_mime_part_detect_language: detected part language: en
2023-09-08 00:18:41 #3892999(controller) <9f5d12>; csession; rspamd_mime_part_detect_language: detected part language: en
2023-09-08 00:18:41 #3892999(controller) <9f5d12>; csession; rspamd_controller_learn_fin_task: <127.0.0.1> learned message as spam: aba30f57-ea42-4413-81fe-d2b12e1f6724@anthanh264.id.vn

```
- Learn Ham 
```

2023-09-08 01:03:21 #3892999(controller) <115a78>; csession; rspamd_controller_check_password: allow unauthorized connection from a trusted IP 127.0.0.1
2023-09-08 01:03:21 #3892999(controller) <115a78>; csession; rspamd_message_parse: loaded message; id: <ace2d1c5-3300-4802-827a-dc9d2bcb0a99@gmail.com>; queue-id: <undef>; size: 98409; checksum: <5ef2acf10e37bb4200d36b6c851c1ffb>
2023-09-08 01:03:21 #3892999(controller) <115a78>; csession; rspamd_mime_part_detect_language: detected part language: en
2023-09-08 01:03:21 #3892999(controller) <115a78>; csession; rspamd_mime_part_detect_language: detected part language: en
2023-09-08 01:03:21 #3892999(controller) <115a78>; csession; rspamd_controller_learn_fin_task: <127.0.0.1> learned message as ham: ace2d1c5-3300-4802-827a-dc9d2bcb0a99@gmail.com

```
- Score khi bị learn SPAM (BAYES_SPAM)
```
2023-09-08 00:27:11 #3892999(controller) <cf1261>; csession; rspamd_task_write_log: id: <157c12f9-9200-4ad3-9e73-89167b94006b@gmail.com>, ip: 10.3.53.226, from: <anthanh264@gmail.com>, (default: T (add header): [6.10/25.00] [BAYES_SPAM(5.10){100.00%;},URI_COUNT_ODD(1.00){5;},MIME_GOOD(-0.10){multipart/alternative;text/plain;},RCVD_NO_TLS_LAST(0.10){},XM_UA_NO_VERSION(0.01){},ARC_NA(0.00){},FREEMAIL_ENVFROM(0.00){gmail.com;},FREEMAIL_FROM(0.00){gmail.com;},FROM_EQ_ENVFROM(0.00){},FROM_HAS_DN(0.00){},MID_RHS_MATCH_FROM(0.00){},MIME_TRACE(0.00){0:+;1:+;2:~;},PREVIOUSLY_DELIVERED(0.00){ah@dinhha.online;},RCPT_COUNT_ONE(0.00){1;},RCVD_COUNT_THREE(0.00){4;},RCVD_VIA_SMTP_AUTH(0.00){},TO_DN_ALL(0.00){}]), len: 48306, time: 4119.145ms, dns req: 24, digest: <18243eafedab9fa6e721f9d38a5db586>, mime_rcpts: <ah@dinhha.online>, file: stdin

```
- Score khi được leadn HAM (BAYES_HAM)
```
2023-09-07 23:51:44 #3765778(rspamd_proxy) <8d265a>; proxy; rspamd_task_write_log: id: <de0d6a80-18a5-44a3-8f81-04e49dd28609@gmail.com>, qid: <4D1B68FBAF>, ip: 209.85.215.169, from: <anthanh264@gmail.com>, (default: F (no action): [-5.09/25.00] [BAYES_HAM(-3.00){100.00%;},RCVD_DKIM_ARC_DNSWL_HI(-1.00){},RCVD_IN_DNSWL_HI(-1.00){2a09:bac5:d45f:16d2::246:3a:received;209.85.215.169:from;},URI_COUNT_ODD(1.00){133;},DMARC_POLICY_ALLOW(-0.50){gmail.com;none;},R_DKIM_ALLOW(-0.20){gmail.com:s=20221208;},R_SPF_ALLOW(-0.20){+ip4:209.85.128.0/17:c;},MIME_GOOD(-0.10){multipart/alternative;text/plain;},RWL_MAILSPIKE_GOOD(-0.10){209.85.215.169:from;},XM_UA_NO_VERSION(0.01){},ARC_NA(0.00){},ASN(0.00){asn:15169, ipnet:209.85.128.0/17, country:US;},DKIM_TRACE(0.00){gmail.com:+;},FREEMAIL_ENVFROM(0.00){gmail.com;},FREEMAIL_FROM(0.00){gmail.com;},FROM_EQ_ENVFROM(0.00){},FROM_HAS_DN(0.00){},MID_RHS_MATCH_FROM(0.00){},MIME_TRACE(0.00){0:+;1:+;2:~;},PREVIOUSLY_DELIVERED(0.00){ah@dinhha.online;},RCPT_COUNT_ONE(0.00){1;},RCVD_COUNT_TWO(0.00){2;},RCVD_TLS_LAST(0.00){},RCVD_VIA_SMTP_AUTH(0.00){},SUBJECT_HAS_QUESTION(0.00){},TO_DN_ALL(0.00){},TO_MATCH_ENVRCPT_ALL(0.00){}]), len: 112132, time: 831.666ms, dns req: 66, digest: <10a468acf5110ad829254ac670b2a742>, rcpts: <ah@dinhha.online>, mime_rcpts: <ah@dinhha.online>
```

## References
* [Install and Integrate Rspamd](https://linuxize.com/post/install-and-integrate-rspamd/)
