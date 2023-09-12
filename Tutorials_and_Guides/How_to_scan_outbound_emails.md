## Why and how to scan outbound mail
- Gửi mail spam có thể ảnh hường nghiêm trọng đến việc gửi email của hệ thống của bạn
- Điều quan trọng là ngăn chặn sự xuất hiện của thư rác và các vấn đề liên quan đến nó
- Trong trường hợp bùng phát thư rác gửi đi, có thể cần phải thực hiện các biện pháp ứng phó sự cố, chẳng hạn như thay đổi mật khẩu của các tài khoản bị ảnh hưởng
- Tiến hành phân tích của con người để xác minh tính xác thực của thư rác cũng có thể có lợi, nhưng điều cần thiết là phải xem xét luật về quyền riêng tư và chính sách của công ty vì phân tích này có thể vi phạm chúng
- Để quyết định hành động phù hợp để xử lý mail spam chuẩn bị gửi đi, nên tìm lời khuyên từ legal experts và có sự tham gia của các bên liên quan
## Scanning outbound with Rspamd
- Rspamd được thiết kế đặc biệt để tạo điều kiện thuận lợi cho việc config cho outbound scanning
- Với sự tích hợp thích hợp, Rspamd có thể nhận dạng user xác thực và địa chỉ IP gửi mail
- Nếu mail được nhận từ một user xác thực hoặc một địa chỉ IP được list trong local_addrs, một số check thì automatically disabled:
  - ASN: checking is disabled for local IPs, unless `check_local` set là `true`
  - DKIM: checking is disabled, ngược lại signign được enabled nếu tìm được key và rule phù hợp
  - DMARC: is disabled
  - Graylist: is disabled
  - HFilter: chỉ URL-checks được applied
  - MX Check: is disabled
  - One Received header policy: is disabled
  - ratelimit: chỉ user ratelimit thì applied (để xác thực user - không xử lý với local_addrs)
  - RBL: RBLs disabled theo `exclude_users` và `exclude_local` setting cho RBL rules ( ví dụ, URL list nên check cho tất cả directions)
  - Replies: action thì không bị ép
  - SPF: disabled
- Ngoài ra, nó thì khả thi để disable/enable check có chọn lọc và/hoặc kiểm tra lại score cho user xác thực của bạn hoặc relay Ips sử dụng setting module

