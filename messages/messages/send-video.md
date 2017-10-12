# เขียนโค้ดส่ videoให้ยูสเซอร์

ก่อนอื่นเรามาดูข้อกำหนดที่ LINE กำหนดไว้เกี่ยวกับการส่ง video ก่อน

![](/assets/2017-10-12_1108-video.png)

originalContentUrl : URL ไฟล์วิดีโอ กำหนดความยาวไว้ไม่เกิน 1000 ตัวอักษร, ต้องเป็น https, เฉพาะไฟล์ mp4 เท่านั้น, ความยาวต้องน้อยกว่า 1 นาที และขนาดไฟล์ไม่เกิน 10 MB

previewImageURL : URL ไฟล์ภาพสำหรับแสดงเป็นตัวอย่าง ความยาวไม่เกิน 1000 ตัวอักษร, เป็น https, ไฟล์ JPEG เท่านั้น, กว้างยาวไม่เกิน 240 x 240 pixel และขนาดไม่เกิน 1 MB

## เปิดไฟล์ index.php ขึ้นมาแล้วเขียนโค้ดดังนี้

```php
<?php

require_once('./vendor/autoload.php');

$channel_token = '1v2OUa9tuMIiDhEg57ANbsRaBDbBGP9nlCC+Dpvt5HrsQ+LqcrImWPUBkH8re/pwqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdcjbFurwXGOyHLt8MdgzLfAe7r0BsQV5cATlUanW3OgJewdB04t89/1O/w1cDnyilFU=';
$channel_secret = '9b2c7349ea939ef723a3cb453d774c86';

// Get message from Line API
$content = file_get_contents('php://input');
$events = json_decode($content, true);

if (!is_null($events['events'])) {

    // Loop through each event
    foreach ($events['events'] as $event) {

        // Get replyToken
        $replyToken = $event['replyToken'];

        // Video
        $originalContentUrl = 'https://www.select2web.com.com/the-fuji.mp4';
        $previewImageUrl = 'https://www.select2web.com.com/the-fuji.jpg';

        $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
        $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

        $textMessageBuilder = new \LINE\LINEBot\MessageBuilder\VideoMessageBuilder($originalContentUrl, $previewImageUrl);
        $response = $bot->replyMessage($replyToken, $textMessageBuilder);

    }
}

echo "OK";
```

## อธิบายโค้ด

```php
require_once('./vendor/autoload.php');
```

ทำการ include ไฟล์ LINEBot SDK เข้ามา

```php
$channel_token = '1v2OUa9tuMIiDhEg57ANbsRaBDbBGP9nlCC+Dpvt5HrsQ+LqcrImWPUBkH8re/pwqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdcjbFurwXGOyHLt8MdgzLfAe7r0BsQV5cATlUanW3OgJewdB04t89/1O/w1cDnyilFU=';
$channel_secret = '9b2c7349ea939ef723a3cb453d774c86';
```

ป้อน Channel token กับ Channel secret ที่ได้มาจาก LINE API เข้าไป อันนี้ของใครของมันไม่เหมือนกัน

```php
$content = file_get_contents('php://input');
$events = json_decode($content, true);
```

get ค่าที่ LINE Server ส่งมาให้แล้วแปลงจาก JSON ไปเป็น Array

```php
// Loop through each event
foreach ($events['events'] as $event) {

    // Get replyToken
    $replyToken = $event['replyToken'];
```

loop เพื่อหา replyToken



```php
// Video
$originalContentUrl = 'https://www.select2web.com.com/the-fuji.mp4';
$previewImageUrl = 'https://www.select2web.com.com/the-fuji.jpg';

```

เตรียมข้อมูลสำหรับส่ง video ให้ยูสเซอร์





```php
$textMessageBuilder = new \LINE\LINEBot\MessageBuilder\VideoMessageBuilder($originalContentUrl, $previewImageUrl);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);
```

ใช้คลาส VideoMessageBuilder สร้างข้อความตอบกลับยูสเซอร์ โดยคลาส VideoMessageBuilder ต้องการพารามิเตอร์ 2 ตัวคือ ที่อยู่ไฟล์วิดีโอ และ ที่อยู่รูปภาพที่จะเอาไปทำ preview

P

