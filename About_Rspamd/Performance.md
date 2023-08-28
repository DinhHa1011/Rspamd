## Mail filtering nowadays
- Rspamd đã được bắt đầu để xử lý mail flows đã tăng hơn 10 lần trong thập kỷ qua. Ngay từ khi bắt đầu project, Rspamd được định hướng trên các hệ thống mail với phát triển tập trung vào hiệu suất và tốc độ quét. Rspamd đươc viết với ngôn ngữ C đơn giản và nó sử dụng một số kỹ thuật để chạy nhanh đặc biệt trên modern hardware. Mặt khác, có thể run Rspamd thậm chí trên một thiết bị nhúng có môi trường rất hạn chế
- Bạn cũng có thể check [bài kiểm tra hiệu suất](https://rspamd.com/misc/2019/05/16/rspamd-performance.html) gần đây để có ấn tượng tốt hơn về độ nhanh của Rspamd
- Rspamd có thể được coi như sự thay thế nhanh hơn cho SpamAssassin mail filter với khả năng scan message gấp 10 lần khi sử dụng cùng rule với Spamassassin. 
- Trong biểu đồ tiếp theo, bạn sẽ thấy việc chuyển sang Rspamd từ SA giúp giảm tải CPU trên scanner machines (để xử lý mail nhanh hơn, rspamd sử dụng một tập hợp các kỹ thuật tối ưu hóa global và local)
![image](https://github.com/DinhHa1011/Rspamd/assets/119484840/7f081a73-0901-4eb6-b86f-235ca1f5d7ac)
## Global optimizations
- Global optimizations được sử dụng để tăng tốc độ xử lý message tổng thể, cải thiện hiệu suất của tất cả filter và sắp xếp các bước ktra theo thứ tự tối ưu
  - Events driven architecture cho phép Rspamd thực thi network và yêu cầu slow khác đồng thời ở chế độ background, cho phép process khác message trong khi đợi replies:
  ![image](https://github.com/DinhHa1011/Rspamd/assets/119484840/1eda4055-e621-4333-965d-1eaf7d3f1de0)
  - Rules reordering được sử dụng để giảm thời gian xử lý message. Rspamd prefer để check rule có quy tắc cao hơn, thời gian thực hiện thấp hơn và hit rate first cao hơn. Hơn nữa, Rspamd sẽ ngừng xử lý khi mail đạt đến ngưỡng spam vì việc kiểm tra thêm có thể sẽ vô nghĩa
## Local optimizations
- Rspamd sử dụng nhiều phương thức để tăng tốc từng giai đoạn xử lý message riêng lẻ. Điều này đạt được bằng cách apply kỹ thuật optimizations local
  - AST optimizations được sử dụng để loại bỏ những kiển tra không cần thiết từ rules
  - Unlike SA, Rspamd sử dụng specific state machines để phân tích các thành phần email: mime structure, HTML parts, URLs, images, received headers,... Cách tiếp cận này cho phép bỏ qua các chi tiết không cần thiết và trích xuất thông tin từ email nhanh hơn so với việc sử dụng một tập hợp lớn các regular expressions cho các mục đích này
  - Hyperscan engine được sử dụng trong Rspamd để xử lý nhanh bộ regular expressions lớn
  - Assembly snippets cho phép optimize thuật toán cụ thể cho targeted architectures
