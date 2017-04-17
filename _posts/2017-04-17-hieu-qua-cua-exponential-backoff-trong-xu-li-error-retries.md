---
title: Hiệu quả của exponential backoff trong xử lí error retries
layout: post
description: >
  Exponential Backoff là gì và nó đã có hiệu quả như nào trong xử lí error?
tags: [exponential backoff, error retries]
categories: [pattern]
date: 2017-04-17 21:30
---

## Lời mở đầu
Khi làm việc với API chắc hẳn ai cũng không lạ lẫm gì với các vấn đề liên quan đến gửi nhận dữ liệu như: đang gửi thì server bị lỗi đến đến gửi thất bại, rồi thì rõ ràng khi thất bại mình đã retries lại mấy lần rồi mà sao vẫn bị lỗi .... vân vân và vân vân ...

Một trong những kĩ thuật khá đơn giản để giải quyết vấn đề này là thực hiện 1 logic retries request đó.
Logic retries mà đa số chúng ta viết thì đều như sau:

```
retries = 0

DO
    wait 1 second

    status = Get the result of response

    IF status = SUCCESS
        retry = false
    ELSE
        retry = true
    END IF

    retries = retries + 1

WHILE (retry AND (retries < MAX_RETRIES))
```

Nhìn thuật toán bên trên dường như `quá perfect???` nhưng mà khi suy nghĩ kĩ hơn thì nó thật sự không hiểu quả trong 1 số trường hợp.
Vậy chúng ta hãy cùng đi xem nó là gì nhé.

OK. Let's Go.

## Exponential Backoff là gì?
- `Exponential Backoff` là 1 thuật toán tính toán thời gian đợi giữa mỗi lần retries theo hàm luỹ kế để việc thực hiện gửi lại request được hiệu quả nhất.

Ví dụ như lần retries thứ 1 sẽ đợi 1s, lần retries thứ 2 sẽ đợi 2s, lần thứ 3 sẽ đợi 4s .... Tức là chúng ta sẽ làm thời gian đợi sau mỗi lần retries kéo dài ra mà không fix cứng thời gian wait như trước nữa và theo thống kê thì làm như này sẽ rất hiệu quả.

## Vấn đề gì nếu không dùng Exponential Backoff?

Vậy với thuật toán ở bên trên chúng ta thấy nó đang gặp vấn đề gì?

- Giả sử như lần đầu tiên chúng ta có N client đồng thời gửi request. Cả N request này đều gửi thất bại và yêu cầu retries lại. 
Như chúng ta thấy bên trên thì sau 1s cả N client này đều gửi lại request đó. Hãy tưởng tượng server lúc đó đang trong trạng thái quá tải. 
Nếu N client lại gửi request đồng thời sau 1s thì chẳng phải điều này là rất tệ đúng không ak? Server không thể nào handle được tất cả các request đó được.

Nếu thời gian gửi lại của các client này mà khác nhau đi thì có lẽ hiệu quả sẽ cao hơn. 
Chính vì lí do này mà `Exponential Backoff` đã ra đời.

## Cài đặt thuật toán Exponential Backoff

```
retries = 0

DO
    wait for random from 0 to (2^retries * 100) milliseconds

    status = Get the result of response

    IF status = SUCCESS
        retry = false
    ELSE
        retry = true
    END IF

    retries = retries + 1

WHILE (retry AND (retries < MAX_RETRIES))
```

Với cài đặt trên thì chúng ta có thể thấy. Nếu request của N client đều thất bại và yêu cầu retries thì lúc này sẽ retries như sau.
Client 1 sẽ đợi 1 khoảng thời gian là 1 số random từ 0 ~ (2^retries * 100) ms
Client 2 sẽ đợi 1 khoảng thời gian là 1 số random từ 0 ~ (2^retries * 100) ms
...

Cứ như thế thì khoảng thời gian mà mỗi client này gửi request lại phía server sẽ khác nhau đi. Nhờ việc tránh gửi đồng thời này dẫn đến hiệu quả cao hơn.


## Ví dụ thực tế

Bây giờ hãy thử xem các ông trùm lớn như Google, Amazon ... đã thực thi nó như thế nào nhé.

#### 1. Google HTTP client

```
retry_interval = retry_interval * multiplier ^ (N - 1)
randomized_interval := retry_interval * (random value in range [1 - randomization_factor, 1 + randomization_factor])
```

- Nếu status code của response là 500 hoặc 503 thì sẽ tiến hành retries.
- Giá trị default của `retry_interval = 0.5s`, `randomization_factor = 0.5`, `multiplier = 1.5`, `max_interval = 1 minute`, `max_elapsed_time = 15s`
- Nếu `retry_interval > max_elapsed_time (15s)` thì stop retries (mặc định là 10 lần)

```
request retry_interval randomized_interval
01      00.50          [0.25, 0.75]
02      00.75          [0.38, 1.12]
03      01.12          [0.56, 1.69]
04      01.69          [0.84, 2.53]
05      02.53          [1.27, 3.80]
06      03.80          [1.90, 5.70]
07      05.70          [2.85, 8.54]
08      08.54          [4.27, 12.81]
09      12.81          [6.41, 19.22]
10      19.22          STOP
```

#### 2. AWS SimpleDB

```
currentRetry = 0
 DO
   status = execute Amazon SimpleDB request
   IF status = success OR status = client error (4xx)
     set retry to false
     process the response or client error as appropriate
   ELSE
     set retry to true
     currentRetry = currentRetry + 1
     wait for a random delay between 0 and (4^currentRetry * 100) milliseconds
   END-IF
 WHILE (retry = true AND currentRetry < MaxNumberOfRetries) 
```

```
Kết quả:

request randomized_interval
1       [0, 400)
2       [0, 1600)
3       [0, 6400)
4       [0, 25600)
5       [0, 102400)
```

## Kết luận

Qua bài giới thiệu của mình hẳn các bạn cũng có cái nhìn khác về thuật toán retries rồi đúng không ak?
Vậy còn chần chừ gì nữa, hãy sửa lại thuật toán để trở thành những pro nào.

### Nguồn tham khảo

Error Retries and Exponential Backoff in AWS: 

<a target="_blank" href="https://docs.aws.amazon.com/general/latest/gr/api-retries.html">https://docs.aws.amazon.com/general/latest/gr/api-retries.html</a>

Exponential Backoff And Jitter:

<a target="_blank" href="https://www.awsarchitectureblog.com/2015/03/backoff.html">https://www.awsarchitectureblog.com/2015/03/backoff.html</a>

Building for Performance and Reliability with Amazon SimpleDB:

<a target="_blank" href="https://aws.amazon.com/articles/Amazon-SimpleDB/1394">https://aws.amazon.com/articles/Amazon-SimpleDB/1394</a>

