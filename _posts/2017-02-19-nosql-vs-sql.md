---
layout: post
title: NoSQL vs SQL
description: >
  Sự khác nhau của trường hợp sử dụng NoSQL với sử dụng SQL
tags: [sql, nosql, database]
categories: [database]
date:   2017-02-19 20:10
---

## NoSQL vs SQL

Hầu hết tất cả chúng ta cũng đã từng nghe qua về RDBMS - Relational Database Management System (Hệ thống quản lý cơ sở dữ liệu quan hệ).
Tuy nhiên, với sự tăng trưởng nhanh chóng về dữ liệu thì việc xử lí tốc độ nhanh là điều cực kì quan trọng.
RDBMS mặc dù được nhiều developer ưa chuộng thế nhưng với 1 số bài toán khác có thể hiệu năng của nó không cao và nhiều khi còn khá là chậm, vì chúng ta sẽ phải join nhiều bảng hơn mới có thể lấy được dữ liệu mong muốn.
Để giải quyết 1 số vấn đề đó thì NoSQL đã ra đời.

NoSQL được sử dụng vì múc đích đưa ra hướng quản lí dữ liệu "Not only SQL", hoặc là hệ thống quản lí dữ liệu tương ứng với "Not SQL".
Có rất nhiều kĩ thuật trong NoSQL như Document database, key-value ... và những cái này thường hay được dùng trong ứng dụng game app, social app, IoT app.

![80x50](https://docs.microsoft.com/ja-jp/azure/documentdb/media/documentdb-nosql-vs-sql/nosql-vs-sql-overview.png)

## Trường hợp sử dụng NoSQL

Bây giờ mình sẽ lấy 1 ví dụ về chức năng comment và post bài trong trang blog để mọi người dễ hiểu nhé. Cụ thể như sau:

- Người dùng có thể tạo bài viết. Hơn nữa có thể thêm ảnh, thêm video.
- Người dùng khác có thể comment bài viết, cho điểm đánh giá bài viết.

Vậy dữ liệu của chúng ta sẽ được lưu như thế nào? Với những thanh niên nào đã ưa chuộng MySQL chắc hẳn sẽ đưa ra 1 cấu trúc schema như ở bên dưới.

![80x50](https://docs.microsoft.com/ja-jp/azure/documentdb/media/documentdb-nosql-vs-sql/nosql-vs-sql-social.png)

OK, bây giờ đã có database rồi. Nhìn có vẻ ngon =)) Bây giờ hãy thử suy nghĩ khi hiển thị dữ liệu về bài viết, cả comment, video, ảnh trên web sẽ như thế nào nhé.
Để lấy được hết dữ liệu cần thiết chúng ta phải join đến 8 bảng. ôi fukkk :'(
Với dữ liệu lớn có khi sẽ mất đến mấy chục đến hàng trăm câu query cũng nên. 

Với vấn đề này thì có lẽ NoSQL là 1 giải pháp tốt hơn. Với NoSQL chúng ta có thể lưu toàn bộ thông tin vào trong 1 đối tượng được gọi là DocumentDB, do đó mà có thể cải thiện được hiệu năng, 
chỉ cần 1 câu query không cần join có thể lấy được dữ liệu mong muốn.

~~~
{
    "id":"ew12-res2-234e-544f",
    "title":"post title",
    "date":"2016-01-01",
    "body":"this is an awesome post stored on NoSQL",
    "createdBy":User,
    "images":["http://myfirstimage.png","http://mysecondimage.png"],
    "videos":[
        {"url":"http://myfirstvideo.mp4", "title":"The first video"},
        {"url":"http://mysecondvideo.mp4", "title":"The second video"}
    ],
    "audios":[
        {"url":"http://myfirstaudio.mp3", "title":"The first audio"},
        {"url":"http://mysecondaudio.mp3", "title":"The second audio"}
    ]
}
~~~

## Sự khác nhau giữa NoSQL vs SQL

![80x50](https://docs.microsoft.com/ja-jp/azure/documentdb/media/documentdb-nosql-vs-sql/nosql-vs-sql-comparison.png)

