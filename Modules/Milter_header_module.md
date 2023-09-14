- `milter headers` module (trước đây được biết đến như `rmilter headers`) đã được add trong Rspamd 1.5 để cung cấp một cách tương đối đơn giản để định cấu hình adding/removing header thông qua milter (cách thay thế là sử dụng API)
- Mặc dù nó là tên, module này cũng hoạt động với các mail server khác như Haraka và communigate
## Principles of operation
- `milter headers` module cung cấp một số thói quen để add header chung, nơi mà có thể được kích hoạt và cấu hình có chọn lọc theo nhu cầu cụ thể. Ngoài ra, user 
có thể linh hoạt add các thói quen tùy chỉnh của riêng mình vào config
