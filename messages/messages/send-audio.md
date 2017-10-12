# เขียนโค้ดส่ง audio ให้ยูสเซอร์

ก่อนอื่นเรามาดูข้อกำหนดที่ LINE กำหนดไว้เกี่ยวกับการส่ง audio![](/assets/2017-10-12_1109-audio.png)

originalContentUrl : URL ชี้ไปยังไฟล์ audio ซึ่งความยาวต้องไม่เกิน 1000 ตัวอักษร, ต้องเป็น https, ไฟล์ m4a เท่านั้น, ความยาวไม่เกิน 1 นาที และขนาดไฟล์ต้องไม่เกิน 10 MB

duration : ความยาวไฟล์เสียง \(หน่วยเป็นมิลลิเซคคั่น\)

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

        // Audio
        $originalContentUrl = 'https://www.select2web.com.com/netural-sound.m4a';
        $duration = 124510;

        $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
        $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

        $textMessageBuilder = new \LINE\LINEBot\MessageBuilder\AudioMessageBuilder($originalContentUrl, $duration);
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

loop เอา replyToken



```php
// Audio
$originalContentUrl = 'https://www.select2web.com.com/netural-sound.m4a';
$duration = 124510;

```

เตรียมข้อมูลไฟล์เสียง







```php
$textMessageBuilder = new \LINE\LINEBot\MessageBuilder\AudioMessageBuilder($originalContentUrl, $duration);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);


```

ใช้คลาส AudioMessageBuilder ส่งไฟล์เสียงไปให้ยูสเซอร์ โดยคลาส AudioMessageBuilder นี้ต้องการพารามิเตอร์ 2 ตัวคือ URL ที่เก็บไฟล์กับความยาวการเล่นไฟล์ เป็นตัวเลขมิลลิเซกคัน

P

