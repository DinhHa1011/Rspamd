# Fuzzy Rspamd 
# Table of contents

- [Fuzzy Rspamd ](#fuzzy-rspamd)
  - [Install](#install)
  - [Config](#config)
  - [Command check log & hash fuzzy](#command-check-log--hash-fuzzy)
  - [Upload Hash Fuzzy ](#upload-hash-fuzzy)
  - [Database Hash Fuzzy ](#database-hash-fuzzy)
  - [Notes](#notes)
  - [References](#references)
## Install
- Update
```
sudo apt update
```
- Install
```
sudo apt install sqlite3
sqlite3 --version
````
## Config 

- /etc/rspamd/rspamd.conf : File cấu hình chính để enable fuzzy localhost
    ```
    nano /etc/rspamd/rspamd.conf
    ```
    +  Comment các dòng 
    ```
    worker "fuzzy" {
        bind_socket = "localhost:11335";
        count = 4; # Disable by default
        backend = "sqlite";
        database = "${DBDIR}/fuzzy.db";
        expire = 90d;
        sync = 1min;
        .include "$CONFDIR/worker-fuzzy.inc"
        .include(try=true; priority=1,duplicate=merge) "$LOCAL_CONFDIR/local.d/worker-fuzzy.inc"
        .include(try=true; priority=10) "$LOCAL_CONFDIR/override.d/worker-fuzzy.inc"
    }

    ```

    + Thêm 
    ```
    worker "fuzzy" {
        bind_socket = "localhost:11335";
        count = 4; # Disable by default
        backend = "sqlite";
        database = "${DBDIR}/fuzzy.db";
        expire = 90d;
        sync = 1min;

        allow_update = ["127.0.0.1"];
        .include "$CONFDIR/worker-fuzzy.inc"
        .include(try=true; priority=1,duplicate=merge) "$LOCAL_CONFDIR/local.d/worker-fuzzy.inc"
        .include(try=true; priority=10) "$LOCAL_CONFDIR/override.d/worker-fuzzy.inc"
    }

    ```
- /etc/rspamd/local.d/fuzzy_check.conf : Thêm các flags cho local fuzzy
    + Mở file  
    ```    
    nano /etc/rspamd/local.d/fuzzy_check.conf
    ```
    + Thêm các dòng
    ```
    rule "local" {
        # Fuzzy storage server list
        servers = "localhost:11335";
        # Default symbol for unknown flags
        symbol = "LOCAL_FUZZY_UNKNOWN";
        # Additional mime types to store/check
        mime_types = ["*"];
        # Hash weight threshold for all maps
        max_score = 20.0;
        # Whether we can learn this storage
        read_only = no;
        # Ignore unknown flags
        skip_unknown = yes;
        # Hash generation algorithm
        algorithm = "mumhash";
        # Use direct hash for short texts
        short_text_direct_hash = true;

        # Map flags to symbols
        fuzzy_map = {
            LOCAL_FUZZY_DENIED {
                # Local threshold
                max_score = 20.0;
                # Flag to match
                flag = 11;
            }
            LOCAL_FUZZY_PROB {
                max_score = 10.0;
                flag = 12;
            }
            LOCAL_FUZZY_WHITE {
                max_score = 2.0;
                flag = 13;
            }
        }
    }
    ```
- /etc/rspamd/local.d/fuzzy_group.conf
    + Mở file
    ```
    nano /etc/rspamd/local.d/fuzzy_group.conf
    ```
    + Thêm các dòng
    ```
    max_score = 12.0;
    symbols = {
        "LOCAL_FUZZY_UNKNOWN" {
            weight = 5.0;
            description = "Generic fuzzy hash match";
        }
        "LOCAL_FUZZY_DENIED" {
            weight = 12.0;
            description = "Denied fuzzy hash";
        }
        "LOCAL_FUZZY_PROB" {
            weight = 5.0;
            description = "Probable fuzzy hash";
        }
        "LOCAL_FUZZY_WHITE" {
            weight = -2.1;
            description = "Whitelisted fuzzy hash";
        }
    }
    ```
 - /etc/rspamd/modules.d/fuzzy_check.conf   
     + Mở file
    ```
    nano /etc/rspamd/modules.d/fuzzy_check.conf
    ```
    + Comment các dòng sau 

    ```

      rule "rspamd.com" {
        algorithm = "mumhash";
        servers = "round-robin:fuzzy1.rspamd.com:11335,fuzzy2.rspamd.com:11335";
        encryption_key = "icy63itbhhni8bq15ntp5n5symuixf73s1kpjh6skaq4e7nx5fiy";
        symbol = "FUZZY_UNKNOWN";

    symbol = "LOCAL_FUZZY_DENIED";
        mime_types = ["*"];
        max_score = 20.0;
        read_only = yes;
        skip_unknown = yes;
        short_text_direct_hash = true; # If less than min_length then use direct hash
        min_length = 64; # Minimum words count to consider shingles
        fuzzy_map = {
          FUZZY_DENIED {
            max_score = 20.0;
            flag = 1;
          }
          FUZZY_PROB {
            max_score = 10.0;
            flag = 2;
          }
          FUZZY_WHITE {
            max_score = 2.0;
            flag = 3;
          }
        }
      }

    ```

    + Thêm các dòng 
    ```
    rule "local" {
        # Fuzzy storage server list
        servers = "localhost:11335";
        # Default symbol for unknown flags
        symbol = "LOCAL_FUZZY_UNKNOWN";
        # Additional mime types to store/check
        mime_types = ["*"];
        # Hash weight threshold for all maps
        max_score = 20.0;
        # Whether we can learn this storage
        read_only = no;
        # Ignore unknown flags
        skip_unknown = yes;
        # Hash generation algorithm
        algorithm = "mumhash";
        # Use direct hash for short texts
        short_text_direct_hash = true;

        # Map flags to symbols
        fuzzy_map = {
            LOCAL_FUZZY_DENIED {
                # Local threshold
                max_score = 20.0;
                # Flag to match
                flag = 11;
            }
            LOCAL_FUZZY_PROB {
                max_score = 10.0;
                flag = 12;
            }
            LOCAL_FUZZY_WHITE {
                max_score = 2.0;
                flag = 13;
            }
        }
    }

    ```
- Restart để apply config
```
systemctl restart rspamd
```
## Command check log & hash fuzzy
- Log
```
tail -f /var/log/rspamd/rspamd.log
```
- Hash fuzzy 
    ```
    rspamadm control fuzzystat 
    ```
    + Result
    ```
    root@dinhha:/etc/dovecot#  rspamadm control fuzzystat
    Statistics for storage 37ee21a22cfc0e4c1f5
    fuzzy_stored: 1
    invalid_requests: 0
    delayed_hashes: 0
    fuzzy_checked: (v0.6: 0), (v0.8: 2)
    fuzzy_shingles: (v0.6: 0), (v0.8: 1)
    fuzzy_found: (v0.6: 0), (v0.8: 1)
    fuzzy_expired: 0

    Keys statistics:

    Errors IPs statistics:


    IPs statistics:

    
## Upload Hash Fuzzy 
### Manual 
- Bằng Web 
https://mail.dinhha.online/rspamd/#scan
Paste source mail -> Set flag, weight ->Upload fuzzy
![](https://i.imgur.com/IJWwoUz.png)

- Bằng Command 
```
rspamc -f 1 -w 10 fuzzy_add <file mail>
```
-f : flag trên ví dụ là flag = 1
-w : weight trên ví dụ là weight = 1
```
root@dinhha:/etc/dovecot# rspamc -f 13 -w 1 fuzzy_add mail.mdl
Results for file: mail.mdl (0.064 seconds)
success = true;
hashes [
    "f3dd087c11777ccf6b7629357bb77eeb0f04d2c48450ab72877d9b40feccfe19aaf92c3b50b54d878e338e7e75ec9d2b31ac4cf047d10c44df40063d230d59e9",
]
filename = "mail.mdl";
scan_time = 0.064001;
```
### Auto 

- Edit file Sieve Filter
    + Mở file
    ```
    nano /var/mail/vmail/sieve/global/report-spam.sieve
    ```
    + Thêm các dòng sau  
    ```
    pipe :copy "fuzzy_denied.sh";
    ```
- Create Bash file

    - Tạo file
    ```
    nano /usr/bin/fuzzy_denied.sh
    ```
    - Nội dung
    ```

    #!/bin/bash
    exec rspamc -f 11 -w 1 fuzzy_add

    ```
    - Cấp quyền 
    ```
    cd /usr/bin/
    chown vmail:vmail fuzzy_denied.sh
    chmod u+x fuzzy_denied.sh
    ```
- Logs
Khi user move mail vào Spam => Upload Hash Fuzzy  
```

2023-09-08 01:15:25 #3892999(controller) <96b1e9>; csession; rspamd_controller_check_password: allow unauthorized connection from a trusted IP 127.0.0.1
2023-09-08 01:15:25 #3892999(controller) <0478c9>; task; rspamd_message_parse: loaded message; id: <e47aed05-4e13-4d35-8766-a630bf26679d@pikab.in>; queue-id: <undef>; size: 187243; checksum: <cfa8d6c7e8bedfcf09ddf6cc9f3f750a>
2023-09-08 01:15:25 #3892999(controller) <0478c9>; task; rspamd_mime_part_detect_language: detected part language: en
2023-09-08 01:15:25 #3892999(controller) <0478c9>; task; rspamd_mime_part_detect_language: detected part language: en
2023-09-08 01:15:25 #3892999(controller) <0478c9>; task; fuzzy_controller_io_callback: added fuzzy hash (txt) df0ed3777cf3939007c54b52c93ac402a4380c5227406cbcd6d937d3a16242ca28e96e490ca8184a620d944c42c0997f9257d07f30006dbe3b3cd7c20e6d2c3a, list: LOCAL_FUZZY_DENIED:11 for message <e47aed05-4e13-4d35-8766-a630bf26679d@pikab.in>
2023-09-08 01:15:25 #3892999(controller) <0478c9>; task; fuzzy_controller_io_callback: added fuzzy hash (txt) e5af15df4657ad072a77065456a18d2081d3b88c6c70b469c86332eff9d1ec0c51007607c58013da7bf8b987cc298668cbc237b7113080e20237e09ced7ad74d, list: LOCAL_FUZZY_DENIED:11 for message <e47aed05-4e13-4d35-8766-a630bf26679d@pikab.in>

```
## Database Hash Fuzzy 
- DB được lưu tại `/var/lib/rspamd/fuzzy.db`
- Xem database này bằng lệnh 
```
sqlite3 /var/lib/rspamd/fuzzy.db
```
- Show tables trong DB `.tables`
- Lệnh Select như T-SQL bt
- Quit bằng lệnh `.quit`
- Ví dụ 
```
root@dinhha:/etc/dovecot# sqlite3 /var/lib/rspamd/fuzzy.db
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> .tables
digests   shingles  sources
sqlite> select * from sources;
local|4|1694018021
sqlite> .quit

```
## Notes
- Các biến thư mục trong Rspamd (Rspamd paths)
    * CONFDIR = ${PREFIX}/etc/rspamd - main path for the configuration
    * LOCAL_CONFDIR = ${PREFIX}/etc/rspamd - path for the user-defined configuration
    * RUNDIR = OS specific (/var/run/rspamd on Linux) - used to store volatile runtime data (e.g. PIDs)
    * DBDIR = OS specific (/var/lib/rspamd on Linux) - used to store static runtime data (e.g. databases or cached files)
    * SHAREDIR = ${PREFIX}/share/rspamd - used to store shared files
    * LOGDIR = OS specific (/var/log/rspamd on Linux) - used to store Rspamd logs in file logging mode
    * LUALIBDIR = ${SHAREDIR}/lualib - used to store shared Lua files (included in Lua path)
    * PLUGINSDIR = ${SHAREDIR}/plugins - used to place Lua plugins
    * RULESDIR = ${SHAREDIR}/rules - used to place Lua rules
    * LIBDIR = ${PREFIX}/lib/rspamd - used to place shared libraries (included in RPATH and Lua CPATH)
    * WWWDIR = ${SHAREDIR}/www - used to store static WebUI files

## References
* [Usage of fuzzy hashes](https://rspamd.com/doc/fuzzy_storage.html)
* [Attempting to use local fuzzy storage](https://groups.google.com/g/rspamd/c/ZTmc_HA0W1o)
* [FAQS Rspamd](https://rspamd.com/doc/faq.html#what-are-local-and-override-config-files)
* [IMAPSieve - permission issues (learn from user)](https://forum.directadmin.com/threads/imapsieve-permission-issues-learn-from-user.64215/))
