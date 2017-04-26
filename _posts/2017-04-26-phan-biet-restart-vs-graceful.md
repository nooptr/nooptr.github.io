---
layout: post
title: Phân biệt restart vs graceful trong apache
description: >
  Bạn có biết đến graceful trong apache?
tags: [graceful, apache]
categories: [apache]
date: 2017-04-26 21:48
---

1 hôm thấy con web chạy chậm quá, sếp đến gần bảo ê thằng em khởi động lại cho anh con web cái. chạy ì ạch quá. 
Nhớ dùng `httpd -k graceful` chứ đừng dùng `httpd -k restart` nhé. Quái lạ `graceful` là cái gì nhỉ? Lần đầu tiên nghe thấy nó. Mọi lần mình vẫn dùng `restart` khởi động lại apache có thấy sao đâu. 

Sau 1 hồi tìm hiểu cụ thể xem nó là gì thì hôm nay mình xin tóm tắt ngắn gọn về sự khác nhau giữa `restart` và `graceful`.

#### 1. Graceful

- Chờ đến khi các child process xử lí xong request thì sẽ kill và tạo mới child process khác (Do đó mà PID của child process sẽ bị thay đổi)
- Parent process sẽ không bị kill, do đó PID vẫn giữ nguyên.
- Parent process chỉ load lại file cấu hình và mở lại file log.
- Tuy nhiên, trong 1 số trường hợp mà thêm module hoặc thay đổi SSL mà graceful sẽ hoạt động không chính xác. Trong trường hợp đó thì nên dùng restart.

#### 2. Restart

- Kill toàn bộ child process ngay lập tức. Sau đó sẽ kill tiếp parent process. (Trường hợp này thì PID của cả parent lẫn child đều bị thay đổi)

Ok vậy đến đây bạn đã hiểu cụ thể nó là gì rồi đúng không ak? Chúc các bạn vận dụng nó tốt trong công việc. =))