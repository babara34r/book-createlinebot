# เขียนโค้ดส่งข้อความตอบกลับผู้ใช้

ก่อนอื่นเรามาดูข้อกำหนดที่ LINE กำหนดไว้เกี่ยวกับการส่งข้อความตอบกลับยูสเซอร์

![](/assets/2017-10-12_1105-text.png)

text : กำหนดความยาวไว้ที่ 2,000 ตัวอักษร และสามารถแทรก emojicon ในระหว่างข้อความได้ ซึ่ง emojicon นั้นก็เป็นแค่ข้อความแหละ ทาง LINE จะเป็นคนแปลงมันให้ออกมาเป็นรูปภาพแสดงอารมณ์เอง emojicon ดูได้จาก Referrence ท้ายหนังสือ

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
        $ask = $event['message']['text'];

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

        $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
        $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

        $textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
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

