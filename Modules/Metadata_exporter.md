## Metadata exporter
- Metadata exporter operates dựa tren một bộ quy tắc xác định message thú vị, và sau đó gửi thông tin dựa trên những quy tắc này cho một dịch vụ bên ngoài
- Exporter support Redis Pub/Sub, HTTP POST, SMTP như built-in backends, trong khi cũng cho phép người dùng xác định custom backends as desired
- Ứng dụng Potential của Metadata exporter bao gồm quarantining, logging, alerting and feedback loóp
### Theory of operation
- Mỗi rule trong config được xác định:
  - Một `selector` function xác định các message mà chúng tôi muốn export metadata from (default selector selects all message)
  - Một `formatter` function extracts formatted metadata from the message (default formatter returns full message content)
  - Một `pusher` function (xác định bởi `backend` setting) pushes metadata được định dạng ở đâu đó
- Một số functions như vậy được xác định trong plugin có thể được sử dụng ngoài các function do người dùng xác định
