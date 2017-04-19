---
layout: post
title: xip.io domain ma thuật
description: >
  xip.io is a magic domain name that provides wildcard DNS for any IP address.
tags: [xip.io, xip]
categories: [dns]
date: 2017-04-19 19:58
---

Các bạn trong làng IT đã từng nghe thấy cái tên `xip.io` bao giờ chưa ak? Thực chất nó cũng chỉ là 1 cái domain thôi cũng không có gì kinh khủng cả. 
Nhưng mà bên trong nó lại có 1 ma lực khá kinh khủng đấy. Vậy cùng đi vào bên trong xem nó là cái gì nhé.

#### 1. Xip.io là gì?

`xip.io` là domain cung cấp cho ta 1 cái DNS cho bất kì 1 địa chỉ IP nào. Hình như có cái gì đó sai sai chỗ này???
Bình thường khi cài đặt domain chúng ta thấy là 1 domain chỉ có thể trỏ đến 1 địa chỉ IP duy nhất thôi đúng ko ak?
Vậy tại sao cái này nó lại trỏ đến được nhiều IP được nhỉ?

#### 2. Xip.io đã làm việc như thế nào?

Thực chất `xip.io` đã chạy 1 cái <a target="_blank" href="https://github.com/basecamp/xip-pdns">custom DNS server</a> và có khả năng tách địa chỉ IP từ domain và forward request đến địa chỉ IP đó.

#### 3. Cách dùng xip.io

`xip.io` dùng khá đơn giản. chúng ta chỉ cần gắn địa chỉ ip vào trước xip.io là OK. Ví dụ:

```
    10.0.0.1.xip.io       resolves to   10.0.0.1
www.10.0.0.1.xip.io       resolves to   10.0.0.1
mysite.10.0.0.1.xip.io    resolves to   10.0.0.1
foo.bar.10.0.0.1.xip.io   resolves to   10.0.0.1
```

#### 4. Ứng dụng

`Bài toán`: Mình có 1 con server có địa chỉ IP là `54.54.54.54` và đang chạy apache.
Bây giờ sếp yêu cầu muốn chạy ứng dụng demo trên đó và chạy cả phpmyadmin nữa nhưng mà sếp lại không cho tiền mua domain để dùng. Vậy chúng ta sẽ làm như nào ak? 

Có 1 cách là với mỗi ứng dụng chúng ta sẽ cài đặt để nó chạy trên 1 cổng khác nhau. ứng dụng web chạy cổng 80, còn phpmyadmin chạy cổng 81.

Ví dụ `úng dụng web: http://54.54.54.54/`

`phpmyadmin: http://54.54.54.54:81/`

Ok. cách này cũng được. Nhưng mình sẽ giới thiệu cho các bạn cách khác hay hơn sử dụng `xip.io`
Trong file virualhost sẽ cấu hình như sau:

```
<VirtualHost *:80>
   ServerName app.54.54.54.54.xip.io
   DocumentRoot "/var/www/html/app"
</VirtualHost>

<VirtualHost *:80>
   ServerName phpmyadmin.54.54.54.54.xip.io
   DocumentRoot "/var/www/html/phpmyadmin"
</VirtualHost>
```

Như vậy ta sẽ được 2 domain thực hiện 2 chức năng khác nhau.

App: http://app.54.54.54.54.xip.io

Phpmyadmin: http://phpmyadmin.54.54.54.54.xip.io

Qủa thật là tuyệt phải không ak? Không cần mua domain mà cũng có domain dùng free như 1 nguòi thợ phải ko các bác.

