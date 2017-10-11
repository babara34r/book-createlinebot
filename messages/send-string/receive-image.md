# เขียนโค้ดรับรูปภาพจาก LINE Server

ดูข้อความที่ LINE Server ส่งมาให้เราถ้าหากมีใครส่งรูปภาพให้บอท

```
{
   "events":[
      {
         "replyToken":"nHuyWiB7yP5Zw52FIkcQobQuGDXCTA",
         "type":"message",
         "timestamp":1462629479859,
         "source":{
            "type":"user",
            "userId":"U206d25c2ea6bd87c17655609a1c37cb8"
         },
         "message":{
            "id":"325708",
            "type":"image"
         }
      }
   ]
}
```

จะเห็นว่าตรง message จะมีเฉพาะ id และบอก type ว่าเป็น image เท่านั้น แล้วในกรณีเราต้องการเก็บภาพที่ยูสเซอร์ส่งให้บอท เราจะทำอย่างไร LINE SDK ให้คำสั่งสำหรับดึงเนื้อของรูปภาพนำไปส่งไฟล์เก็บในเซิฟเวอร์ของเรา ผมเขียนไว้ให้แล้วในตัวอย่าง

\*\*\* สิ่งหนึ่งจะต้องจำก็คือ ภาพที่ใครก็ตามส่งให้บอทนั้นจะถูกเก็บไว้บนเซิฟเวอร์ของ LINE เพียงชั่วคราวเท่านั้น ระยะหนึ่งจะถูกลบทิ้งไป LINE เองก็บอกเวลาที่แน่ชัดไม่ได้ว่าจะเก็บไว้นานเท่าไร \(ถ้าคุณอยากจะเก็บภาพนั้นไว้ใช้ภายหลัง คุณต้องสร้างไฟล์ไว้บนเซิฟเวอร์ตัวเอง\)

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

                case 'image':
                    $messageID = $event['message']['id'];

                    /*                 
                    $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
                    $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));
                    $response = $bot->getMessageContent($messageID);

                    if ($response->isSucceeded()) {
                        $tempfile = tmpfile();
                        fwrite($tempfile, $response->getRawBody());
                    }

                    // Returns status code 200 and the content in binary.
                    // Note: Content is automatically deleted after a certain period from when the message was sent.
                    // There is no guarantee for how long content is stored.
                    */

                    // Reply message
                    $respMessage = 'Hello, your image ID is '. $messageID;

                    break;
                default:
                    // Reply message
                    $respMessage = 'Please send image only';
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
$messageID = $event['message']['id'];
$respMessage = 'Hello, your image ID is '. $messageID;
```

ให้ส่งข้อความว่า Hello, your image ID is ต่อท้ายด้วย ID ของรูปภาพไปให้

```
/*                 
$httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
$bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));
$response = $bot->getMessageContent($messageID);

if ($response->isSucceeded()) {
    $tempfile = tmpfile();
    fwrite($tempfile, $response->getRawBody());
}

// Returns status code 200 and the content in binary.
// Note: Content is automatically deleted after a certain period from when the message was sent.
// There is no guarantee for how long content is stored.
*/
```

โค้ดนี้เป็นโค้ดดึงเอา content ของรูปภาพออกมาจากที่เก็บไว้ในเซิฟเวอร์ของ LINE แล้วเขียนเป็นไฟล์บนเซิฟเวอร์ของเรา เพื่อเก็บไว้ใช้ \(ผมคอมเม้นท์เอาไว้ หากจะใช้งานก็เปิดคอมเม้นท์ และ ต้องมั่นใจว่าโฟลเดอร์ permission อนุญาติให้สร้างไฟล์\)

```php
$httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
$bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

$textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);
```

LINE SDK สำหรับส่งข้อความกลับไปหายูสเซอร์

P



















