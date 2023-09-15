- Multimap module được thiết kế đặc biệt để xử lý rule dựa trên nhiều loại danh sách khác nhau được Rspamd cập nhật và được gọi là `map`.
- Module này đặc biệt hữu ích cho tổ chức whitelists, blacklist, và lists khác thông qua files
- Ngoài ra, nó có khả năng load remote list sử dụng HTTP và HTTPS protocol hoặc RESP (REdis Serialization Protocol)
- Bài viết này cung cấp giải thích chi tiết về tất cả các tùy chọn config vfa tính năng có sẵn trong module này
## Nguyên tắc hoạt động
- Module này xác định các quy tắc cho phép trích xuất nhiều loại dữ liệu (xác định bằng `type`)
- Dữ liệu được trích xuất được chuyển đổi theo cách mong muốn (được xác định bởi `filter`) và nó được kiểm tra dựa trên danh sách các chuỗi thường được gọi là `map`:
![image](https://github.com/DinhHa1011/Rspamd/assets/119484840/047e3743-f316-4f0c-ab77-c14398ebd730)
- Một lỗi phổ biến là sử dụng `type` thay vì `filter` và ngược lại. Để tránh nhầm lẫn, xin lưu ý rằng `type` là thuộc tính của bản đồ xác định dữ liệu chính xác nào được trích xuất
- Maps trong Rspamd tham khảo các file hoặc HTTP links được tự động theo dõi và tải lại nếu có bất kỳ thay đổi nào xảy ra. Dưới đây là ví dụ về cách xác định bản đồ:
```
map = "http://example.com/file";
map = "file:///etc/rspamd/file.map";
map = "/etc/rspamd/file.map";
```
- Rspamd cung cấp option để lưu traffic cho HTTP map bằng cached maps, đồng thời tôn trọng `304 Not modified responses`, Cache-Control headers, and ETags
- Ngoài ra, maps data được chia sẻ giữa các workers, và chỉ first controller worker mới được phép fetch remote maps
- Theo mặc định, config của module này tích cực sử dụng comppound maps, xác định map là một array của sources với một local fallback location. Trong khi redundancy này có thể không cần thiết đối với các maps do người dùng xác định, nhưng thông tin chi tiết hơn có sẵn trong phần FAQ section
