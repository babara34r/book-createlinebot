# เขียนโค้ดรับ Sticker จาก LINE Server

ข้อความที่ LINE Server ส่งมาให้เราถ้าหากมีใครส่ง Sticker ให้บอท

```
{
   "events":[
      {
        "replyToken": "nHuyWiB7yP5Zw52FIkcQobQuGDXCTA",
        "type": "message",
        "timestamp": 1462629479859,
        "source": {
          "type": "user",
          "userId": "U206d25c2ea6bd87c17655609a1c37cb8"
        },
        "message": {
          "id": "325708",
          "type": "sticker",
          "packageId": "1",
          "stickerId": "1"
        }
      }
   ]
}
```

จะเห็นว่าตรง message จะมี packageId และ stickerId ถูกส่งมาให้  มีคนถามทางบริษัท LINE ว่าสามารถดึงภาพ Sticker มาเก็บไว้ที่เครื่องได้ไหม LINE ตอบว่าไม่ได้ โดยความคิดเห็นส่วนตัวผมก็คิดว่าไม่ได้ เพราะว่าสติ๊กเกอร์บางอันลูกค้าซื้อมา

\*\*\* สิ่งที่ต้องจำ เราไม่สามารถดึงภาพ Sticker มาเก็บที่เครื่องได้

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

        // Line API send a lot of event type, we interested in message only.
        if ($event['type'] == 'message') {

            // Get replyToken
            $replyToken = $event['replyToken'];

            switch($event['message']['type']) {

                case 'sticker':
                    $messageID = $event['message']['packageId'];

                    // Reply message
                    $respMessage = 'Hello, your Sticker Package ID is '. $messageID;

                    break;
                default:
                    // Reply message
                    $respMessage = 'Please send Sticker only';
                    break;
            }

            $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
            $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

            $textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
            $response = $bot->replyMessage($replyToken, $textMessageBuilder);
        }
    }
}

echo "OK";
```

เสร็จแล้ว push โค้ดขึ้น repository รอสักแป้บให้ทาง Heroku ดึงโค้ดส่งขึ้นโฮสต์ก่อน ลองส่งภาพอะไรก็ได้ให้บอทดู

## อธิบายโค้ด

```php
require_once('./vendor/autoload.php');
```

ทำการ include ไฟล์ LINEBot SDK เข้ามา

```php
$channel_token = '1v2OUa9tuMIiDhEg57ANbsRaBDbBGP9nlCC+Dpvt5HrsQ+LqcrImWPUBkH8re/pwqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdcjbFurwXGOyHLt8MdgzLfAe7r0BsQV5cATlUanW3OgJewdB04t89/1O/w1cDnyilFU=';
$channel_secret = '9b2c7349ea939ef723a3cb453d774c86';
```

ป้อน Channel token กับ Channel secret ที่ได้มาจาก LINE API เข้าไป อันนี้ของใครของมันไม่เหมือนกัน ถ้าหากคุยกับบอทแล้วบอทไม่ตอบอะไร ให้ดูตรงนี้ให้มั่นใจว่าถูกต้อง

```php
$content = file_get_contents('php://input');
$events = json_decode($content, true);
```

get ค่าที่ LINE Server ส่งมาให้แล้วแปลงจาก JSON ไปเป็น Array

```php
switch($event['message']['type']) {
```

เหตุที่ต้องมีการตรวจเช็กว่าเมสเสจที่ส่งมาเป็นอะไรนั้นเพราะว่า โครงสร้างข้อมูลแตกต่างกันไปในแต่ละประเภท

```php
$messageID = $event['message']['packageId'];

// Reply message
$respMessage = 'Hello, your Sticker Package ID is '. $messageID;
```

ถ้าหากยูสเซอร์ส่ง Sticker หาบอทให้บอทส่งข้อความตอบไปว่า  Hello, your Sticker Package ID is

```php
$httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
$bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

$textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);
```

LINE SDK สำหรับส่งข้อความกลับไปหายูสเซอร์

P

