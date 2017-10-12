# เขียนโค้ดส่งรูปภาพตอบกลับผู้ใช้

ก่อนอื่นเรามาดูข้อกำหนดที่ LINE กำหนดไว้เกี่ยวกับการส่งรูปภาพตอบกลับยูสเซอร์

![](/assets/2017-10-12_1107-image.png)

originalContentUrl : URL ที่เก็บรูปภาพ ความยาวไม่เกิน 1000 ตัวอักษร, ต้องเป็น https, ไฟล์ JPEG, กว้างยาวสูงสุด 1024 x 1024 พิกเซล และขนาดไม่เกิน 1 MB

previewImageUrl : URL ภาพที่ใช้สำหรับทำ preview ความยาวไม่เกิน 1000 ตัวอักษร, ต้องเป็น https, ไฟล์ JPEG, กว้างยาวสูงสุด 240 x 240 พิกเซล และขนาดต้องไม่เกิน 1MB

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

        // Image
        $originalContentUrl = 'https://cdn.shopify.com/s/files/1/1217/6360/products/Shinkansen_Tokaido_ShinFuji_001_1e44e709-ea47-41ac-91e4-89b2b5eb193a_grande.jpg?v=1489641827';
        $previewImageUrl = 'https://cdn.shopify.com/s/files/1/1217/6360/products/Shinkansen_Tokaido_ShinFuji_001_1e44e709-ea47-41ac-91e4-89b2b5eb193a_grande.jpg?v=1489641827';

        $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
        $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

        $textMessageBuilder = new \LINE\LINEBot\MessageBuilder\ImageMessageBuilder($originalContentUrl, $previewImageUrl);
        $response = $bot->replyMessage($replyToken, $textMessageBuilder);
        
	}
}

echo "OK";

```

ตัวอย่างที่กำหนดเขียนนี้ เราจะให้บอทถามยูสเซอร์ว่า คุณเพศอะไร? ถ้าหากเขาตอบมาว่า m เราก็ให้บอทตอบไปว่า "What sup man. Go away" แต่ถ้ายูสเซอร์ตอบมาว่า f เราจะให้บอทตอบกลับไปว่า Love you lady.

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
switch(strtolower($ask)) {
    case 'm':
        $respMessage = 'What sup man. Go away!';
        break;
    case 'f':
        $respMessage = 'Love you lady.';
        break;
    default:
        $respMessage = 'What is your sex? M or F';
        break;    
}
```

ถ้าหากว่ายูสเซอร์พิมพ์คุยกับบอทมา ไม่ใช่ m/f ให้บอทถามไปว่า What is your sex? M or F

ถ้าหากว่ายูสเซอร์ตอบบอทมาว่า m ให้บอทตอบกลับไปว่า What sup man. Go away!

ถ้าหากว่ายูสเซอร์ตอบบอทมาว่า f ให้บอทตอบกลับไปว่า Love you lady.

```php
$textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);
```

ใช้คลาส TextMessageBuilder สร้างข้อความตอบกลับยูสเซอร์

P

