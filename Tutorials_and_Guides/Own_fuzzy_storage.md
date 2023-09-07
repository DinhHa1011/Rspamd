# Usage of fuzzy hashes
## Introduction
- Fuzzy hashing có thể được sử dụng để search những message tương tự, cho phép chúng ta xác định các message có văn bản giống nhau hoặc được sửa đổi một chút. Kỹ thuật này đặc biệt hữu ích để block spam được gửi tới nhiều user cùng một lúc
- Mục đích của page này là giải thích cách sử dụng fuzzy hashes, không cung cấp chi tiết sâu rộng hoặc hiểu biết thấu đáo về cách chúng hoạt động trong rspamd
- Có 3 high-level steps hướng tới việc sử dụng fuzzy hashes
  - Hash sources selection
  - Config storage
  - Configuring fuzzy_check plugin
- Optional: Hashes relication
- Suggested storage testing
## Step 1: Hash sources selection 
- Điều quan trọng là phải lựa chọn cẩn thận các nguồn mẫu thư rác để đào tạo. Nguyên tắc chung là sử dụng message spam được đông đảo người dùng đón nhận. Có 2 cách tiếp cận chính cho task này:
  - Giải quyết khiếu nại của users
  - Tạo bẫy spam (honeypot)
### Working with user complaints
- Khiếu nại của người dùng có thể là nguồn hữu ích để cải thiện chất lượng lưu trữ hash, nhưng điều quan trọng cần lưu ý là đôi khi người dùng có thể phàn nàn về các email hợp pháp mà họ đã đăng ký, chẳng hạn như store newwsletters, ticket booking notifications, và thậm chí persional email mà họ không thích vì một vài lý do nào đó. NHiều người dùng không phân biệt được giữa `Delete` và `Mask as Spam`
- Một giải pháp cho vấn đề này là nhắc người dùng cung cấp thông tin về complaint, như tại sao họ tin email là spam. Điều này có thể thu hút sự chú ý của người dùng đến thực tế là họ có thể hủy đăng ký khỏi việc nhận những email không mong muốn thay vì đánh dấu chúng là spam. Một cách tiếp cận khác là xử lý thủ công các khiếu nại về spam của người dùng
- Sự kết hợp của các phương pháp này cũng có thể có hiệu quả: gán trọng số lớn hơn cho các email đã được xử lý thủ công và đặt trọng số nhỏ hơn cho tất cả các khiếu nại khác
- Có 2 tính năng trong Rspamd có thể giúp lọc ra các kết quả sai (email bị đánh dấu nhầm là spam). (Note: trong documentation này, FP là viết tắt của "False Positive" và FN là viết tắt của "False Negative")
  - 1. Hash weight
  - 2. Learning filters
#### Hash weight
- Một phương thức để lọc ra kết quả FP là gán trọng số cho từng khiếu nại và add weight này để lưu trữ giá trị hash trong mỗi learning step tiếp theo.
- Khi query storage, chúng tôi sẽ bỏ qua các giá trị hash có trọng số nhỏ hơn ngưỡng xác định. Ví dụ, nếu weight của một complaint là `w=1` và threshold (khiếu nại) là `t=20`, sau đó chúng ta sẽ bỏ qua hash này trừ khi chúng tôi nhận được ít nhất 20 khiếu nại của người dùng
- Hơn nữa, rspamd không ấn định điểm tối đa ngay khi đạt đến giá trị ngưỡng. Thay vào đó, điểm tăng dần từ 0 đến mức tối đa (liên giá trị metric) khi weight của hash từ gí trị ngưỡng lên gấp đôi giá trị ngưỡng (t..2*t)
![image](https://github.com/DinhHa1011/Rspamd/assets/119484840/972902c4-17f3-4256-8e28-63e1719bd5bc)
#### Learning filters
- Phương pháp thứ 2 để lọc FP dựa trên khiếu nại của người dùng liên quan đến việc viết các điều kiện bằng ngôn ngữ Lua, chẳng hạn như có thể bỏ qua quá trình học họăc sửa đổi gía trị của hash cho email từ các domain cụ thể. Những bộ lọc này cung cấp nhiều khả năng, nhưng chúng yêu cầu viết và cấu hình thủ công
### Configuring spam straps
- Phương thức `honeypot` nhằm nâng cao giá trị của bộ lưu trữ hash liên quan đến việc sử dụng hộp thư chỉ nhận email spam và không nhận email hợp pháp. Ý tưởng là một khối lượng lớn thư rác mới, được đảm bảo (có thể là 100%) sẽ được tiếp tục nhận, theo các mẫu hiện tại, cung cấp một lượng lớn dữ liệu băm mờ để so sánh với email nhận được từ các hộp thư trực tiếp. Như đã đề cập trước đó, cách hiểu của người dùng về thư rác có thể dễ xảy ra lỗi. Tập hợp thư rác do người dùng báo cáo không đáng tin cậy bằng bẫy thư rác, trong đó các kết quả trùng khớp rất có thể chỉ ra rằng email mới đến cũng là spam
- Một cách để thiết lập bẫy spam là tiết lộ địa chỉ cho spammer database chứ không phải cho người dùng hợp pháp. Điều này có thể được thực hiện bằng cách đặt địa chỉ email trong phần tử `iframe` ẩn trên một trang web phổ biến chẳng hạn. Phần tử này không hiển thị với người dùng do thuộc tính `headden` hoặc kích thước bằng 0, nhưng nó hiển thị với các chương trình thư rác. Phương pháp này không còn hiệu quả như trước vì những kẻ gửi thư rác đã học được cách tránh những cái bẫy như vậy.
- Một cách khác để tạo bẫy là tìm các tên miền phổ biến trước đây nhưng không còn hoạt động. Những tên miền này có thể được tìm thấy trong nhiều cơ sở dữ liệu thư rác. Mua các miền này và cho phép tất cả thư đến đi đến một địa chỉ nhận toàn bộ thư, nơi nó được xử lý để băm mờ và sau đó bị loại bỏ. Nói chung, việc thiết lập các bẫy của riêng bạn như thế này chỉ thực tế đối với các hệ thống thư lớn vì nó có thể tốn kém về mặt bảo trì và các chi phí trực tiếp như mua tên miền.
## Step 2: Configuring storage
- Qúa trình Rspamd chịu trách nhiệm lưu trữ fuzzy hash được gọi là `fuzzy_storage` worker. Thông tin ở đây sẽ hữu ích cho dù bạn đang sử dụng bộ nhớ local hay remote
- Quá trình này thực hiện các chức năng sau đây sẽ được trình bày chi tiết ở đây:
  - Data storage
  - Hash expiration
  - Access control (read and write)
  - Transport protocol encryption
  - Replication
- Config cho `worker "fuzzy"` bắt đầu trong phần `/etc/rspamd/rspamd.conf`
- Một lệnh `.include` ở đó liên kết đến `/etc/rspamd/local.d/worker-fuzzy.inc`, đây là nơi cài đặt local kích hoạt và định cấu hình quá trình này (tài liệu trc đó đề cập đến `/etc/rspamd/rspamd.conf.local`)
### Sample configuration
- Sau đây là cấu hình mẫu cho quy trình nhân viên lưu trữ mờ này, sẽ được giải thích và đề cập bên dưới. vui lòng tham khảo trang này để biết bất kỳ cài đặt nào không được mô tả ở đây.
```
worker "fuzzy" {
  # Socket to listen on (UDP and TCP from rspamd 1.3)
  bind_socket = "*:11335";

  # Number of processes to serve this storage (useful for read scaling)
  count = 4;

  # Backend ("sqlite" or "redis" - default "sqlite")
  backend = "sqlite";

  # sqlite: Where data file is stored (must be owned by rspamd user)
  database = "${DBDIR}/fuzzy.db";

  # Hashes storage time (3 months)
  expire = 90d;

  # Synchronize updates to the storage each minute
  sync = 1min;
}
```
- Ví dụ này shows một entire section, không phải như bạn sẽ thấy trong file, mà như nó nhìn vào controller khi chi tiết cài đặt được thu thập từ tất cả file (Với tệp .include): hãy đảm bảo đặt các thay đổi trong tệp .inc, không như `worker` wrapper
- Mặc định, tiến trình `fuzzy_storage` không active, với `count=-1` chỉ thị được tìm thấy trong core file. Để active fuzzy storage, local .inc file nhận lệnh `count = 4` như đã thấy ở trên.
- Các giá trị `expire` và `sync` có liên quan đến việc dọn dẹp và hiệu suất cơ sở dữ liệu, như được mô tả bên dưới
- Fuzzy storage làm việc với hashes và không phải với email message. Một worker/scanner process hoặc một controller process chuyển đổi email thành hash trước khi kết nối tới process này cho fuzzy processing. Trong mẫu này, chúng ta sẽ thấy quy trình fuzzy storage hoạt động trên database SQLite được nghe trên socket 11335 cho UDP requests từ processes khác để query hoặc update storage
![image](https://github.com/DinhHa1011/Rspamd/assets/119484840/ff2c53af-1680-4f60-9bea-a9dcfbf7339f)
### Data storage
- Công cụ database, SQLite3, có một số hạn chế về kiến trúc lưu trữ có thể ảnh hưởng đến hiệu suất. Cụ thể, SQLite không thể xử lý tốt các write đồng thời, nơi mà có thể dẫn đến suy giảm đáng kể hiệu suất database
- Để giải quyết vấn đề này, Rspamd hash storage luôn write vào database từ một quy trình duy nhất, fuzzy storage worker. Quá trình này duy trì một update queue, trong khi tất cả process khác chỉ đơn giản là chuyển tiếp yêu cầu write từ client tới quy trình này. Theo mặc định, update queue được viết vào disk mỗi phút một lần, nhưng điều này có thể được cấu hình bằng cách sử dụng cài đặt đồng bộ hóa trong sample config
- Kiến trúc này được tối ưu hóa cho các yêu cầu đọc và ưu tiên chúng
### Hash expiration
- Một chức năng quan trọng khác của fuzzy storage worker là xóa hash lỗi thời bằng cách sử dụng `expire` setting
- Mô hình spam thay đổi khi các chiến thuật nhất định trở nên hiệu quả ít nhiều. Spammers gửi hàng loạt mail spam, sau một khoảng thời gian từ vài ngày đến vài tháng, họ thay đổi mô hình vì họ biết những hệ thống như thế này đang phân tích dữ liệu của họ. Vì " thời gian tồn tại hiệu quả" của spam email thì luôn giới hạn, không có lý do gì để lưu trữ tất cả hash permanently. Dựa trên kinh nghiệm, nó recommended để lưu trữ hash không quá 3 tháng.
- bạn nên so sánh khối lượng hash đã học trong một khoảng thời gian nhất định với ram có sẵn. ví dụ: 400.000 giá trị băm có thể chiếm khoảng 100 mb và 1,5 triệu giá trị băm có thể chiếm khoảng 500 mb. để tránh tình trạng giảm hiệu suất đáng kể, không nên tăng kích thước bộ nhớ vượt quá kích thước ram khả dụng. nghĩa là không dựa vào không gian trao đổi hoặc phân bổ quá nhiều tài nguyên cho các quy trình khác. nếu bạn có một lượng nhỏ băm phù hợp cho việc học, hãy bắt đầu với thời gian hết hạn là 90 ngày. nếu khối lượng dữ liệu trong khoảng thời gian đó dẫn đến lượng ram khả dụng không thể chấp nhận được, chẳng hạn như ram khả dụng trong thời gian cao điểm giảm xuống 20%, bạn có thể muốn giảm thời gian hết hạn xuống 70 ngày và xem liệu dữ liệu từ các bản phát hành bộ lưu trữ có hết hạn hay không một lượng ram dễ chấp nhận hơn.
### Access Control
- Theo mặc định, Rspamd không cho phép thay đổi fuzzy storage. Bất kì hệ thống nào connect tới tiến trình fuzzy_storage thông qua UDP phải được ủy quyền, và phải cung cấp danh sách các địa chỉ IP và / hoặc mạng đáng tin cậy để hỗ trợ việc học. Trong thực tế, nó tốt hơn là viết từ địa chỉ local (127.0.0.1) vì fuzzy storage sử dụng UDP, không được bảo vệ khỏi việc giả mạo IP nguồn
```
worker "fuzzy" {
  # Same options as before ...
  allow_update = ["127.0.0.1"];

  # or 10.0.0.0/8, for internal network
}
```
- Cài đặt `allow_update` là một mảng gồm các huỗi phân tách bằng dấu `,`, hoặc một map của địa chỉ IP, được phép thực hiện các thay đổi đối với fuzzy storage - bạn cũng nên đặt read_only = no trong plugin matte_check của mình, xem bước 3 bên dưới
### Transport protocol encryption
- fuzzy hash protocol cho phép mã hóa tùy chọn ( opportunistic) hoặc mã hóa bắt buộc dựa trên pulic-key cryotigraphy. Tính năng này được sử dụng cho việc tạo kho lưu trữ hạn chế nơi quyền truy cập chỉ được phép dành cho khách hoàng hoặc đối tác kinh doanh khác có khóa chung được tạo
- How this works:
  - Config được sửa đổi trong `/etc/rspamd/local.d/worker-fuzzy.inc` của hệ thống cục bộ đang chạy fuzzy_storage worker. Một cặp khóa public/private được cài cho mỗi remote UDP client rằng sẽ connect trên port 11335
  - Mỗi public key duy nhất được cấp cho mỗi hệ thống client duy nhất, để chỉ một hệ thống đó có thể sử dụng một khóa đó.
- Kiến trúc mã hóa sử dụng cryptobox construction và nó tương tự thuật toán mã hóa đầu cuối được sử dụng trong giao thức DNSCurve
- Để config transport encryption, tạo một keypair cho storage server, sử dụng dòng lệnh `rspamadm keypair -u`. Mỗi lần lệnh này run, output duy nhất được trả về, như shown trong ví dụ (thứ tự của các cặp name=value có thể thay đổi mỗi lần run):
```
keypair {
    pubkey = "og3snn8s37znxz53mr5yyyzktt3d5uczxecsp3kkrs495p4iaxzy";
    privkey = "o6wnij9r4wegqjnd46dyifwgf5gwuqguqxzntseectroq7b3gwty";
    id = "f5yior1ag3csbzjiuuynff9tczknoj9s9b454kuonqknthrdbwbqj63h3g9dht97fhp4a5jgof1eiifshcsnnrbj73ak8hkq6sbrhed";
    encoding = "base32";
    algorithm = "curve25519";
    type = "kex";
}
```
- Public key nên được sao chép thủ công tới remote host, hoặc publish theo bất kỳ cách nào đảm bảo độ tin cậy. Như mọi khi, private key không nên được publish hoặc share
- Mỗi storage có thể sử dụng số nào của key simultaneously, 1 cho mỗi remote client (hoặc 1 group của client)
```
worker "fuzzy" {
  # Same options as before ...
  keypair {
    pubkey = ...
    privkey = ...
  }
  keypair {
    pubkey = ...
    privkey = ...
  }
  keypair {
    pubkey = ...
    privkey = ...
  }
}
```
- Cơ chế này là tùy chọn, nhưng nó có thể thành bắt buộc nếu add option `encrypted_only`. Trong mode này, client systems không có public key hợp lệ sẽ không thể truy cập vào storage
```
worker "fuzzy" {
  # Same options as before ...
  encrypted_only = true;

  keypair {
    ...
  }
  ...
}
```
### Hashes replication
- Có một bản sao local của remote fuzzy storage có thể được sử dụng trong nhiều tình huống. Để tạo điều kiện thuận lợi cho việc này, Rspamd cung cấp support cho hash replication, được xử lý bởi fuzzy storage worker. Bạn có thể tìm thấy hướng dẫn thiết lập trong step 4 bên dưới
## Step 3: Configuring `fuzzy_check` plugin
- `fuzzy_check` plugin được sử dụng bởi scanner processes để query một storage, và bởi các quy trình điều khiển để learning fuzzy hashes
- Plugin functions:
  - Email processing and hash creation from email part and attachments
  - Quering from and learning to storage
  - Transport Encryption
- Learning được thực hiện bởi dòng lệnh `rspamc fuzzy_add`:
```
rspamc -f 1 -w 10 fuzzy_add <message|directory|stdin>
```
  - `-w`: hash weight
  - `-f`: flag number
- Flags enable lưu trữ hash từ sources khác nhau. Ví dụ, một hash có thể bắt nguồn từ một spam trap, hash khác có thể là kết quả của user complaints (khiếu nại người dùng), và một hash thứ 3 có thể đến từ email trên một whitelist. Mỗi flag có thể được liên kết với symbol riêng và có weight khi check emails:
![image](https://github.com/DinhHa1011/Rspamd/assets/119484840/f82f9e0d-a034-466c-8be0-c05a3a99b1b3)
- symbol name có thể được sử dụng thay vì numeric flag during learning, ví dụ:
```
$ rspamc -S FUZZY_DENIED -w 10 fuzzy_add <message|directory|stdin>
```
- Ký hiệu `FUZZY_DENIED` tương đương với flag=11, như định nghĩa trong `modules.d/fuzzy_check.conf`. Để match ký hiệu với flag tương ứng, bạn có thể sử dụng `rule`
```
local.d/fuzzy_check.conf
```
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
```
local.d/fuzzy_group.conf
```
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
- Đây là một số tùy chọn hữu ích có thể được đặt trong module:
- One option là `max_score`, trong đó chỉ định ngưỡng cho hash weight
![image](https://github.com/DinhHa1011/Rspamd/assets/119484840/69b5e415-420e-48a6-ac08-7134a13d716e)
- Option `mime_types` chỉ định loại attachment nào được kiểm tra (hoặc learned) sử dụng fuzzy rule. Option này lấy danh sách các loại valid theo format dưới đây: ["type/sybtype", "*/subtype", "type/*", "*"], trong đó `*` đại diện cho bất kì loại hợp lệ nào. Trong thực tế, việc lưu các giá trị hash cho tất cả `fuzzy_check` plugin, vì vậy không cần add `image/*` trong list của scanned attachment. Chú ý rằng attachments và images thì được search cho một exact match, trong khi text thì được match sử dụng thuật bán gần đúng (shingles)
  - `read_only`: là một option khá quan trọng cần thiết cho việc storage learning. Nó set `read_only = true` là mặc định, do đó hạn chê khả năng storage's learning:
```
read_only = true; # disallow learning
read_only = false; # allow learning
```
- Tham số `encryption_key` chỉ định public key của storage và enable encryption cho tất cả request
- Tham số `algorithm` chỉ định thuật toán để tạo hash từ text part của email
- Ban đầu, rspamd chỉ support thuật toán `siphash`. Tuy nhiên, thuật toán này có một vài vấn đề hiệu năng, đặc biệt là trên hardware cũ hơn (CPU models up to Intel Haswell). Sau đó, support đã được thêm vào cho các thuật toán sau:
  - mumhash
  - xxhash
  - fasthash
- Đối với phần lớn các config chúng tôi khuyên dùng mumhash hoặc fasthash (còn gọi nhanh là fast). Cấc thuật toán n ay hoạt động tốt trên nhiều nền tảng và mumhash hiện là mặc định cho tất cả cá bộ storage mới. `siphash` (còn gọi là `old` chỉ được support cho các mục đích cũ.
- Bạn có thể đánh giá hiệu suất của các thuật toán khác nhau bằng [compiling the tests set
](https://rspamd.com/doc/tutorials/writing_tests.html) từ rspamd sources:
```
make rspamd-test
```
- Run test suite các biến thế khác nhau của thuật toán hash trên specific platform:
```
test/rspamd-test -p /rspamd/shingles
```
- Important note: Việc thay đổi tham số này sẽ dẫn đến mất tất cả dữ liệu trong fuzzy hash storage, vì mỗi lần lưu trữ chỉ có thể dùng một thuật toán cho mỗi bộ storage. Không thể chuyển đổi loại hash này sang loại hash khác vì hash được thiết kế để không thể đảo ngược.
### Condition scripts for the learning
- `fuzzy_check` plugin chịu trách nhiệm cho learning, chúng ta tạo một script với comfig của nó. Script này quyết định xem một mail có phù hợp để học không. Script nên return một function Lua với một đối số duy nhất thuộc loại `rspamd_task`. The function nên return một boolean value (`true`: learn, `false`: skip learning), hoặc một pair chứa một boolean value và một numeric value ( để modify hash flag value, nếu necessary). Tham số `learn_conditions` được sử dụng để setup learn script. Cách thuận tiện nhất để đặt script là viết nó như một chuỗi nhiều dòng được support bởi `UCL`:
```
# Fuzzy check plugin configuration snippet
learn_condition = <<EOD
return function(task)
  return true -- Always learn
end
EOD;
```
- Đây là một số ví dụ thực tế của useful script. Ví dụ, nếu chúng ta mong muốn hạn chế learning với các message đến từ một số domain nhất định:
```
return function(task)
  local skip_domains = {
    'example.com',
    'google.com',
  }

  local from = task:get_from()

  if from and from[1] and from[1]['addr'] then
    for i,d in ipairs(skip_domains) do
      if string.find(from[1]['addr'], d) then
        return false
      end
    end
  end


end
```
- Nó có thể cũng được sử dụng để chia hash thành các flags khác nhau dựa trên source của họ. Ví dụ, như sources có thể encode trong `X-Source` title. Chẳng hạn, chúng ta có sự match giữa flags và sources:
  - `honeypot`: `black` list: 1
  - `users_unfiltered` - `gray` list: 2
  - `user_filtered` - `black` list: 1
  - `FP` - `while` list: 3
- Script cung cấp logic này có thể như dưới đây:
```
return function(task)
  local skip_headers = {
    ['X-Source'] = function(hdr)
      local sources = {
        honeypot = 1,
        users_unfiltered = 2,
        users_filtered = 1,
        FP = 3
      }
      local fl = sources[hdr]

      if fl then return true,fl end -- Return true + new flag
      return false
    end
  }

  for h,f in pairs(skip_headers) do
    local hdr = task:get_header(h) -- Check for interesting header
    if h then
      return f(hdr) -- Call its handler and return result
    end
  end

  return false -- Do not learn if specified header is missing
end
```
## Hashes replication
- Người ta thường mong muốn có một local copy của remote storage. Rspamd supports replication cho mục đích này được triển khai trong hash storage từ version 1.3:
![image](https://github.com/DinhHa1011/Rspamd/assets/119484840/708ebfdf-f8cd-40a5-92db-8a0eac0caddb)
- The hashes transfer được khởi xướng bởi replication master. Nó gửi hash update commands, như adding, modifying hoặc deleting, tới tất cả specified slaves. Do đí slaves phải có thể chấp nhận connect từ master. Điều này cần đực tính đến khi config firewall
- Theo mặc định slave listens port 11335 trên TCP để accept connections. Đồng bộ hóa giữa master và slave được perform qua HTTP protocol với HTTPCrypt transport encryption. Để cập nhật lặp đi lặp lại hoặc thao tác k hợp lệ, slave check update version. Nếu version của master nhỏ hơn hoặc bằng local version, update bị reject. Nếu master đi trước slave bởi nhiều hơn 1 version, message sau sẽ thông báo dưới log của slave:
```
rspamd_fuzzy_mirror_process_update: remote revision: XX is newer more than 1 revision than ours: YY, cold sync is recommended
```
- Trong trường hợp này chúng tôi recommand tạo lại database thông qua `cold` synchronization
## The cold synchronization
- Procedure này được sử dụng để khởi tạo một slave mới hoặc recover một slave sau khi liên lạc với master bị gián đoạn
- Để sync master host bạn cần stop rspamd service và tạo một dump của hash database. Về lý thuyết, bạn có thể skip step này, tuy nhiên, nếu một version của master tạo bởi nhiều hơn một trong database cloning, nó sẽ được yêu cầu lặp lại thủ tục:
```
sqlite3 /var/lib/rspamd/fuzzy.db ".backup fuzzy.sql"
```
- Sau đó, copy output file `fuzzy.sql` cho tất cả slaves (nó có thể thực hiện mà không cần stop dịch vụ rspamd trên slaves):
```
sqlite3 /var/lib/rspamd/fuzzy.db ".restore fuzzy.sql"
```
- Sau tất cả, bạn có thể run rspamd trên slaves và sau đó switch trên master
## Replication setup
- Bạn có thể set replication trong file config hash storage, tên là `worker-fuzzy.inc`. Master replication được config như dưới đây:
```
# Fuzzy storage worker configuration snippet
# Local keypair (rspamadm keypair -u)
sync_keypair {
    pubkey = "xxx";
    privkey = "ppp";
    encoding = "base32";
    algorithm = "curve25519";
    type = "kex";
}
# Remote slave
slave {
        name = "slave1";
        hosts = "slave1.example.com";
        key = "yyy";
}
slave {
        name = "slave2";
        hosts = "slave2.example.com";
        key = "zzz";
}
```
- Hãy tập trung vào việc config encryptions keys. Thông thường, rspamd tự động tạo một cặp khóa cho client và không yêu cầu một thiêt lập chuyên dụng nào. Tuy nhiên, trong replication case, master hoạt động như client, vì vậy bạn có thể set specific (public) key trên slaves cho better access control. Slave sẽ chỉ cho phép update đối với host đang sử dụng key này. Nó cũng khả thi để set cho phép IP-address của master, nhưng public key dựa trên protection có thể đáng tin cậy hơn. Ngoài ra, bạn có thể kết hợp phương thức này
- Slave setup trông như:
```
# Fuzzy storage worker configuration snippet
# We assume it is slave1 with pubkey 'yyy'
sync_keypair {
    pubkey = "yyy";
    privkey = "PPP";
    encoding = "base32";
    algorithm = "curve25519";
    type = "kex";
}

# Allow update from these hosts only
masters = "master.example.com";
# Also limit updates to this specific public key
master_key = "xxx";
```
- Để tránh xung đột với local hashes, bạn có thể set một flag translation từ master tớ slave. Ví dụ, config sau có thể được sử dụng để dịch flag 1,2,3 thành 10,20,30
```
# Fuzzy storage worker configuration snippet
master_flags {
  "1" = 10;
  "2" = 20;
  "3" = 30;
};
```
## Storage testing
- Để test storage bạn có thể sử dụng command `rspamadm control fuzzystat`
```
Statistics for storage 73ee122ac2cfe0c4f12
invalid_requests: 6.69M
fuzzy_expired: 35.57k
fuzzy_found: (v0.6: 0), (v0.8: 0), (v0.9: 0), (v1.0+: 20.10M)
fuzzy_stored: 425.46k
fuzzy_shingles: (v0.6: 0), (v0.8: 41.78k), (v0.9: 23.60M), (v1.0+: 380.87M)
fuzzy_checked: (v0.6: 0), (v0.8: 95.29k), (v0.9: 55.47M), (v1.0+: 1.01G)

Keys statistics:
Key id: icy63itbhhni8
        Checked: 1.00G
        Matched: 18.29M
        Errors: 0
        Added: 1.81M
        Deleted: 0

        IPs stat:
        x.x.x.x
                Checked: 131.23M
                Matched: 1.85M
                Errors: 0
                Added: 0
                Deleted: 0

        x.x.x.x
                Checked: 119.86M
                ...
```
- Về cơ bản, số liệu thống kê lưu trữ chung được trình diễn, như số của store và hash lỗi thời, và phân phối các yêu cầu cho client Protocol version:
  - `v0.6` - requests from rspamd 0.6 - 0.8 (older versions, compatibility is limited)
  - `v0.8` - requests from rspamd 0.8 - 0.9 (partially compatible)
  - `v0.9` - unencrypted requests from rspamd 0.9+ (fully compatible)
  - `v1.1` - encrypted requests from rspamd 1.1+ (fully compatible)
- Sau đó số liệu thống kê được hiển thị cho mỗi config key trong storage và để biết địa chỉ IP của client. Trong phần kết luận, chúng ta thấy số liệu thống kê tổng thể về IP-addresses
- Để thay đổi output từ command này, bạn có thể sử dụng options dưới dây:
  - `-n`: hiển thị raw numbers without reduction
  - `--short`: không hiển thị thống kê chi tiết trên key và IP-address
  - `--no-keys`: không show statistics trên keys
  - `--no-íp`: không show statistic trên IP-addresses
  - `--sort`: sort:
    - `checked`: theo số lượng hash đáng tin cậy (default)
    - `matched`: theo số lượng hash tìm thấy
    - `error`: theo số lượng request failed
    - `ip`: theo IP-address theo từ điển
- Ví dụ:
```
rspamadm control fuzzystat -n
```
# Config
# Fuzzy Rspamd Basic 
# Table of contents

- [Table of contents](#table-of-contents)
  - [Update](#update)
  - [Install](#install)
  - [Config](#config)
  - [Restart](#restart)
  - [Check log](#check-log)
  - [Check hash fuzzy](#check-hash-fuzzy)
  - [Add hash](#add-hash)
  - [DB Fuzzy](#db-fuzzy)
  - [Notes](#notes)
  - [References](#references)

## Update
```
sudo apt update

```
## Install
```
sudo apt install sqlite3
sqlite3 --version
````
## Config 

- /etc/rspamd/rspamd.conf
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
- /etc/rspamd/local.d/fuzzy_check.conf
    ```    
    nano /etc/rspamd/local.d/fuzzy_check.conf
    ```
    + Thêm 
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
    ```
    nano /etc/rspamd/local.d/fuzzy_group.conf
    ```
    + Thêm 
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
    ```
    nano /etc/rspamd/modules.d/fuzzy_check.conf
    ```
    + Comment 

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

    + ADD
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
## Restart 
```
systemctl restart rspamd
```
## Check log
```
tail -f /var/log/rspamd/rspamd.log
```
## Check hash fuzzy 
```
rspamadm control fuzzystat 
```

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

```
## Add hash 
### Web 
https://mail.dinhha.online/rspamd/#scan
Paste source mail -> Set flag, weight ->Upload fuzzy
![](https://i.imgur.com/IJWwoUz.png)

### Command 
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
## DB Fuzzy 
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
Các biến thư mục trong rspamd 
Rspamd paths
There are several variables defined in UCL configuration parser and they are also exported via rspamd_paths global table in Lua code (available everywhere). Here are the default meanings and values for those paths (by ${PREFIX} we denote the default installation prefix, e.g. /usr):

CONFDIR = ${PREFIX}/etc/rspamd - main path for the configuration
LOCAL_CONFDIR = ${PREFIX}/etc/rspamd - path for the user-defined configuration
RUNDIR = OS specific (/var/run/rspamd on Linux) - used to store volatile runtime data (e.g. PIDs)
DBDIR = OS specific (/var/lib/rspamd on Linux) - used to store static runtime data (e.g. databases or cached files)
SHAREDIR = ${PREFIX}/share/rspamd - used to store shared files
LOGDIR = OS specific (/var/log/rspamd on Linux) - used to store Rspamd logs in file logging mode
LUALIBDIR = ${SHAREDIR}/lualib - used to store shared Lua files (included in Lua path)
PLUGINSDIR = ${SHAREDIR}/plugins - used to place Lua plugins
RULESDIR = ${SHAREDIR}/rules - used to place Lua rules
LIBDIR = ${PREFIX}/lib/rspamd - used to place shared libraries (included in RPATH and Lua CPATH)
WWWDIR = ${SHAREDIR}/www - used to store static WebUI files

## References
* [Usage of fuzzy hashes](https://rspamd.com/doc/fuzzy_storage.html)
* [Attempting to use local fuzzy storage](https://groups.google.com/g/rspamd/c/ZTmc_HA0W1o)
* [FAQS Rspamd](https://rspamd.com/doc/faq.html#what-are-local-and-override-config-files)
