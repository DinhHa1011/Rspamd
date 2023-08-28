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
- Một lệnh `.include` ở đó liên kết đến `/etc/rspamd/local.d/worker-fuzzy.inc`, đây là nơi cài đặt local kích hoạt và định cấu hình quá trình này (tài liệu trc đó đề cập đến `/etc/rspamd/rspamd.conf.local`
