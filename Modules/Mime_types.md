- Module này nhằm mục đích thực hiện kiểm tra độ chính xác của một số mime type. Bao gồm những điều dưới đây:
  - Check xem mime type có nằm trong `good` list không (`multipart/alternative` hoặc `text/html`)
  - Check xem mime type có nằm trong `bad` list không (`multipart/form-data`)
  - Check xem một attachment filename extension có đính kèm intended mime type khác
  - Check nội dung lưu trữ (rar and zip được support) và tìm một số bad file bên trong
  - Check một vài bad patterns khác thường được sử dụng bởi spammer, ví dụ extensions hiding (ví dụ `.pdf.exe)
