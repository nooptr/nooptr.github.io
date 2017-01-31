---
layout: post
title: Garbage Collector trong Ruby
description: >
  Tối ưu code ruby thông qua garbage collector, cách thức làm việc của garbage collector
tags: [web, ruby, performance, gc, garbage collector]
categories: [web, linux]
date:   2017-01-30 22:30
---

Đến 1 ngày đẹp trời nào đó, trang web mà mình đang vận hành tự nhiên chạy chậm lại. Đa số đều nghĩ ngay đến việc khởi động lại server web xem thế nào.
Và kết quả tốc độ trang web lại nhanh hơn. Wtf???

Đến ngay cả con router của mình cũng thế, cứ khi nào mạng chậm lại chạy đi khởi động lại con router và kết quả là tốc độ mạng cũng nhanh hơn =))

Có 1 số nguyên nhân có thể xảy ra như thiếu memory, mạng chậm ... Có 1 nguyên nhân có thể gây ảnh hưởng đến ứng dụng mà ít ai để ý đó là Garbage Collector (Bộ thu gom rác)

## Garbage Collector (Bộ thu gom rác)

- Garbage Collector là 1 trong những bộ phận xác định và loại bỏ 1 số object không được sử dụng khỏi bộ nhớ Heap. 
- Không gian trống này sẽ được cấp phát cho những object mới.
- Với ngôn ngữ C thì việc khởi tạo và giải phóng bộ nhớ được thực hiện 1 cách thủ công. Nhưng với Ruby và 1 số ngôn ngữ khác thì được thực hiện 1 cách tự động.

## Ruby sử dụng memory như thế nào?

#### Objects

Trong Ruby, bất cứ thứ gì đều là object và được biểu diễn như cấu trúc `RVALUE`. `sizeof(RVALUE)` được tính như sau:

- 20 bytes in 32-bit architecture when double is 4-byte aligned
- 24 bytes in 32-bit architecture when double is 8-byte aligned
- 40 bytes in 64-bit architecture

Chúng ta có thể xem được thông tin về object thông qua 1 số tools debugger. Ví dụ như `gdb` với Linux, và `lldb` với Mac OS.


~~~
$ gdb `rbenv which ruby`
(gdb) p sizeof(RVALUE)
$1 = 40
(gdb)
~~~


### Ruby Objects Heap

- Ruby cấp phát vùng nhớ cho object trong heap space - cái này bao gồm nhiều heap pages.
- Mỗi heap page bao gồm nhiều slots, mỗi slot dành cho 1 object.

Khi Ruby muốn cấp phát vùng nhớ cho object, nó sẽ tìm xem slot nào chưa được sử dụng từ heap. Nếu không có free slot nào thì khi đó Ruby sẽ tạo thêm heap page. Số lượng heap page được thêm vào phụ thuộc vào tình trạng của heap page hiện tại, version ruby, heap growth algorithm.

#### Ruby 1.8

- Ruby 1.8 sẽ add 1 heap page 1 lần
- First page sẽ được tạo khi mà khởi động ứng dụng, bao gồm `HEAP_MIN_SLOTS` slots. Mặc định giá trị này là 10.000
- Số slot của trang sau sẽ bằng slot trang trước * 1.8
- Ví dụ: Trang đầu có 10.000 slots, trang 2 sẽ có 18.000 slots, trang 3 sẽ có 32.400 slots...

#### Ruby 1.9 and 2.0

- Từ Ruby 1.9 trở về sau sẽ có thuật toán heap growth khác với 1.8
- Thay cho việc add mỗi page 1 lần thì nó sẽ thực hiện việc pre-allocate 1 vài page.
- Khi startup, Ruby 1.9 sẽ pre-allocate `HEAP_MIN_SLOTS / HEAP_OBJ_LIMIT` heap pages. Mặc định là `10.000 / 408 = 24 heap pages`
- Khi interpreter mà cần nhiều page hơn thì nó sẽ lấy số lượng page hiện tại đã dùng, nhân với 1.8. Ví dụ: Với 24 pages được tạo lúc startup, Ruby sẽ tăng số lượng page lên `24 * 1.8 = 43`, sẽ add `43 - 24 = 19` new heap. 43 heaps này sẽ có `43 * HEAP_OBJ_LIMIT = 43 * 408 = 17,544` object slots.

#### Ruby 2.1

- Thay thế hằng số `HEAP_MIN_SLOTS` thành `GC_HEAP_INIT_SLOTS`. Mặc định vẫn là 1.8
- Ruby 2.1 sẽ tạo ra 1 page lúc startup trước khi allocating initial 24 pages.
- Ruby từ 2.1 trở đi thì sẽ phân chia heap space thành `eden` và `tomb`. Khi mà allocate object, Ruby sẽ tìm trong `eden` những space đang trống đầu tiên. Nếu mà không có thì sẽ lấy free page từ `tomb`

### Object Memory

- 1 object trong Ruby có thể lưu được 1 lượng hạn chế về dữ liệu, lên đến 40 byte với hệ điều hành Linux 64 bit.
- Tất cả dữ liệu không lưu vừa trong 1 object thì sẽ tự động được cấp phát động ra bên ngoài heap của Ruby. Khi object được quyét bởi GC thì memory sẽ được giải phóng.
- Ví dụ: string trong Ruby chỉ lưu đc 23 byte trong `RSTRING` object với hệ điều hành 64 bit. Khi chiều dài của string lớn hơn 23 byte thì ruby sẽ cấp phát thêm bộ nhớ cho nó. Cụ thể như ví dụ bên dưới:

~~~
irb(main):003:0> require 'objspace'
=> false
irb(main):004:0> str = 'x'
=> "x"
irb(main):005:0> ObjectSpace.memsize_of(str)
=> 40
irb(main):006:0> str = 'x'*23
=> "xxxxxxxxxxxxxxxxxxxxxxx"
irb(main):007:0> ObjectSpace.memsize_of(str)
=> 40
irb(main):008:0> str = 'x'*24
=> "xxxxxxxxxxxxxxxxxxxxxxxx"
irb(main):009:0> ObjectSpace.memsize_of(str)
=> 65
irb(main):010:0>
~~~

Với ví dụ bên trên thì với xâu mà có chiều dài hơn 23 bytes thì nó sẽ lưu ra bên ngoài object. Tổng số size là 65 bytes: 40 bytes cho Ruby object ở trong heap, 24 bytes cấp phát động ra bên ngoài heap, và 1 byte cho vấn đề upkeep (bảo trì)

### Know What Triggers GC

GC sẽ điều khiển 2 việc đó là cấp phát object trong heap space và cấp phát memory cho object bên ngoài Ruby heap. 
trigger GC sẽ hoạt động khi:

- Không đủ free slot trong heap space.
- Cấp phát bộ nhớ hiện tại (current memory allocation) đã quá hạn cho phép.

Do vậy mà cứ khi nào tạo object hoặc cấp phát bộ nhớ đều sẽ gọi GC.

#### GC Triggered by Heap Usage

- Khi mà hết slot trong heap space, GC sẽ được start để giải phóng 1 ít memory. Nếu mà GC không thể giải phóng đủ slots, Ruby sẽ tăng heap space.

#### GC Triggered by Malloc Limit

- Nếu mà cấp phát bộ nhớ nhiều hơn giá trị cho phép thì GC sẽ được start.
- Ruby 2.0 trở về trước thì định nghĩa giới hạn này bằng hằng số `GC_MALLOC_LIMIT`. Giá trị này mặc định là 8 triệu bytes ~ 7.63 MB


## Tài liệu tham khảo:

[Ruby Performance Optimization: Why Ruby is Slow, and How to Fix It](https://www.amazon.com/dp/1680500694)
![80x50](https://images-na.ssl-images-amazon.com/images/I/51nNW87Uv3L._SX415_BO1,204,203,200_.jpg)
