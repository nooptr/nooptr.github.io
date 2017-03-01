---
title: Amazon SQS là gì
layout: post
description: 'Amazon SQS làm được gì và khi nào nên sử dụng nó?

'
tags:
- sqs
categories:
- aws
date: 2017-03-01 21:40
---

Mình cũng đã nghe qua khá nhiều về cái tên SQS (Simple Queue Service) nhưng mà chưa từng lần nào tìm hiểu hẳn hoi tử tế. Hôm nay quyét tâm dành thời gian ra tìm hiểu nó.
Thực ra mình đang định thi chứng chỉ Amazon Web Service mà trong đó thấy nó nói nhiều về SQS quá nên không tìm hiểu sâu không được.
Thôi không nói lan man nữa. Vào vấn đề chính thôi.

### 1. SQS là gì?

- `Amazon Simple Queue Service` là 1 dịch vụ về hàng đợi lưu trữ thông điệp (message) 1 cách nhanh chóng, đáng tin cậy và có khả năng mở rộng cao.
- Và đặc biệt là nó khá rẻ ($0.4 với 1 triệu requests)

### 2. SQS làm được gì?

- Với 1 queue, nó có thể gửi, nhận, xoá message.
- Với 1 số tính năng mà không yêu cầu tính realtime thì ta có thể gửi message đến queue và xử lí sau đó thông qua batch.

### 3. Cụ thể SQS được sử dụng như thế nào?

Để dễ hiểu bây giờ mình sẽ lấy 1 ví dụ về logic nạp tiền trong game.  Khi đó với trường hợp không sử dụng SQS với trường hợp sử dụng SQS nó khác nhau thế nào?

#### a. Không sử dụng SQS

- Khi người dùng nạp tiền trong game, đa số chúng ta đều viết logic xử lí payment đồng thời với việc ghi log vào trong DB.
- Giả sử như việc ghi log vào DB là 1 việc cần thiết. Vì 1 lí do nào đó mà logic nào đó mà việc ghi log vào trong DB bị lỗi dẫn đến toàn bộ xử lí bị stop lại và đồng thời thông báo tin nhắn ERROR đến người dùng.

#### b. Sử dụng SQS
- Khi người dùng tiến hành nạp tiền thì sẽ gửi 1 message ghi log đến SQS và sẽ thực hiện việc xử lí ghi log này về sau thông qua batch. 
- Nhờ vào việc xử lí này mà thời gian xử lí toàn bộ logic của hệ thống cũng giảm xuống và nó cũng không gây ảnh hưởng gì đến logic xử lí payment coin.
- Đối với message đang được lưu trên SQS thì sẽ được xử lí thông qua batch từ 1 server khác được chạy định kì.


### 4. Áp dụng thực tế

Sau đây mình xin viết 1 đoạn code demo về việc tương tác với SQS như sau:

※ Mình sẽ không hướng dẫn các bạn cài đặt SQS trên AWS nhé. Các bạn có thể tham khảo về việc cài đặt ở link cuối bài.

`Make a SQS instance`

```
function static get_sqs_client()
{
    $client = new SqsClient([
      'version' => 'latest',
      'region' => Region::TOKYO
    ]);

    return $client;
}
```

`Send message to SQS`

```
public static function send_sqs_message($sqs_name, $data) {
  // Get SQS instance
  $client = self::get_sqs_client();

  // Send message to SQS
  $client->sendMessage(array(
    'QueueUrl'    => "https://sqs.ap-northeast-1.amazonaws.com/[QUEUE_ID]/{$sqs_name}",
    'MessageBody' => json_encode($data),
  ));
}
```

`Receive message from SQS`

```
public static function receive_sqs_message() {
  // Get SQS instance
  $client = self::get_sqs_client();

  // Get Queue URL
  $listqueues = $client->listQueues();
  $queueUrls  = $listqueues['QueueUrls'];

  // Get message from Queue
  foreach ($queueUrls as $queueUrl) {
    $result = $client->receiveMessage(array(
      'QueueUrl' => $queueUrl,
      'MaxNumberOfMessages' => 10,
      'WaitTimeSeconds' => 20
    ));

    foreach ($result->getPath('Messages') as $messageBody) {
      $data = json_decode($messageBody);

      // array('user_id' => '1', 'purchase_date' => '20140501', 'price' => '9800');
      $values = $data['values'];

      // TODO Process log to DB
      $target_table = $data['target_table'];
      Model_Logs::insert_log($target_table, $values);

      // Delete message from SQS
      $ReceiptHandle = $messageBody['ReceiptHandle'];
      $client->deleteMessage([
        'QueueUrl' => $queueUrl,
        'ReceiptHandle' => $ReceiptHandle
      ]);
    }
  }
}
```

Hi vọng bài viết của mình sẽ giúp ích cho các bạn khi tìm hiểu về SQS.

### Nguồn tham khảo

AWS SDK for PHP:

<a target="_blank" href="http://docs.aws.amazon.com/aws-sdk-php/v3/guide/getting-started/installation.html">http://docs.aws.amazon.com/aws-sdk-php/v3/guide/getting-started/installation.html</a>

Guide SQS for AWS SDK PHP: 

<a target="_blank" href="http://docs.aws.amazon.com/aws-sdk-php/v2/guide/service-sqs.html">http://docs.aws.amazon.com/aws-sdk-php/v2/guide/service-sqs.html</a>
