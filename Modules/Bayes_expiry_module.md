## Bayes expiry module
- `bayes expriry` module cung cấp expiration thông minh của statistic token cho `new schema` của Redis statistic storage
## Module configuration
- configuration setting cho `bayes expiry` module nên được tích hợp vào session `classifier` phù hợp, như file `local.d/classifier-bayes.conf`
- Ngoài ra, `Bayes expiry` module đòi hỏi sử dụng statistic schema mới, bắt buộc phải kích hoạt nó trong classifier config:
```
new_schema = true;    # Enabled by default for classifier "bayes" in the stock statistic.conf since 2.0
```
- Setting dưới đây hợp lệ:
  - expire: set TTL (time to live) cho tokens. Tokens trong `common` category không bị ảnh hưởng. Xem thêm thông tin bên dưới. Support giá trị:
    - time in seconds (hậu tố đơn vị thời gian được support). Giá trị TTL tối đa có thể có tong Redis là 2147483647 (int32);
    - `-1`: làm cho mã thông báo liên tục;
    - `false`: disable `bayes expiry` cho classifier. Lưu ý rằng điều này không làm thay đổi TTLs của existing tokens, nhưng những token mới học được sẽ tồn tại lâu dài
  - lazy (before 2.0): `true` - enable lazy expiration mode (disable theo mặc định)
- Config example:
```
new_schema = true;    # Enabled by default for classifier "bayes" in the stock statistic.conf since 2.0
expire = 8640000;
#lazy = true;    # Before 2.0
```
## Cơ chế hoạt động
- `bayes expiry` module thực hiện một bước expiry mỗi phút. Trong mõi bước, nó nghiên cứu tính thường xuyên của khoảng 1000 statistic token và điều chỉnh TTLs nếu cần
- Thời lượng mỗi lần lặp đầy đủ khác nhau dựa trên số token, ví dụ 1 chu kỳ đầy đủ cho 10 triệu token mất khoảng 1 tuần để hoàn thành. Mỗi `bayes expiry` modules kết thúc một lần lặp đầy đủ, nó bắt đầu lại
## Tokens classification
- `Bayes expiry` module phân loại token thành 4 group dựa trên tính thường xuyên của họ của tần suất xảy ra trong ham và spam classes:
  - infrequent: xảy ra quá thường xuyên (tổng số lần xuất hiện rất thấp)
  - significant: xảy ra thường xuyên hơn đáng kể trong một class (ham or spam) so với lớp kia
  - common (meaningless): xảy ra cả ham và spam classes với khoảng cùng tần số
  - insignificant: sự khác biệt về sự xuất của chúng trong class nằm ở đâu đó giữa `significant` và `common` tokens
## Expiration modes
### Default expiration mode (before 2.0)
- `default` mode đã bị xóa trong Rspamd 2.0 vì nó không mang lại lợi ích gì so với `lazy` mode
- Hoạt động:
  - Kéo dài `significant` thời gian tồn tại của token: update TTL của token mỗi lần `expire` value
  - Không làm gì với một token `insignificant` hoặc `infrequent`
  - Phân biệt một token `common`: reset TTL về gí trị thấp (10d) nếu token có TTL lớn hơn
- Nhược điểm:
  - Statistic phải không được lưu trữ offile lâu hơn thời gian `expire`. `Bayes Expiry` module phải cập nhật định kỳ TTLs của họ, điều này có nghĩa là cần phải có các thủ tục sao lưu đặc biệt. chỉ cần sao chép tệp *.rdb sẽ khiến tệp hết hạn sau khi hết thời gian.
  - Nếu không có chính sách eviction nào được đặt trong redis nhắm mục tiêu đến significant tokens thì việc cập nhật liên tục các TTLs của chúng là không cần thiết.
### Lazy expiration mode (1.7.4+)
- `lazy` mode là expiration mode duy nhất từ Rspamd 2.0
- Mode này đảm bảo rằng `significant` token có TTL được giữ liên tục (module cài đặt significant token TTLs thành -1, tức là làm cho họ kiên trì nếu họ không) trong khi TTL của `insignificant` hoặc `infrequent` token giảm giá trị `expire` nếu TTL hiện tại của nó vượt quá `expire`. `Common` token được phân biệt bằng cách setting lại TTL của họ xuống giá trị thấp hơn là 10 ngày nếu TTL của họ vượt quá ngưỡng này
- Lợi ích của `lazy` mode bao gồm:
  - Khả năng giữ statistic offile cho thời gian vô hạn mà không có nguy cơ mất `significant` token
  - Giảm thiểu các update TTL không cần thiết càng nhiều càng tốt
- Để active lazy expiration mode trong Rspamd version prior 2.0, simply add `lazy = true`, đến classifier config
### Changing expiration mode (before 2.0)
- expiration mode cho một existing statistics database có thể thay đổi trong config tại any moment. TTLs của token sẽ update theo yêu cầu trong chu kỳ expiration tiếp theo.
### Changing expire value
- Khi một giá trị `expire` được đặt thành giá trị thấp hơn giá trị hiện tại, TTLs vượt giá giá trị `expire` mới sẽ update trong expiration cycle tiếp theo
- Để tăng giá trị `expire`, trước tiên cần thiết làm token kiên trì bởi setting `expire = -1`, va đợi đến tận khi ít nhất một expiration cycle được hoàn thành. Chỉ khi đó giá trị `expire` mới mới có thể cài đặt
## Limiting memory usage to a fixed amount
- Memory usage của statistics dataset có thể được quản lý bằng cách sử dụng Redis `maxmemory` directive và `volatile-ttl` eviction policy. Nếu memory usage vượt quá `maxmemory` limit, Redis sẽ loại bỏ các khóa có TTLs tùy theo policy
-  
