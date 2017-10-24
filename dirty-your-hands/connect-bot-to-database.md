# ตัวอย่างการเชื่อมต่อฐานข้อมูล Postgresql

เป้าหมายของโค้ดตัวอย่างนี้ ต้องการแสดงให้เห็นว่า หลังจากเราได้ message มาจาก LINEBot API แล้วเราจะเอาไปทำอะไรก็เรื่องของเรา เราสามารถเขียนโปรแกรมได้อย่างอิสระ ผมเลือกติดต่อกับฐานข้อมูลเพราะว่าเป็นงานที่เรามักจะต้องเจอกันประจำ

ก่อนอื่นให้ไปสร้างฐานข้อมูลชื่อ  logs ภายในมีฟิลด์ 2 ฟิลด์คือ id, log ชื่อฟิลด์จะต้องตั้งเป็นตัวเล็กทั้งหมดนะครับ

```php
<?php

require_once('./vendor/autoload.php');

use \LINE\LINEBot\HTTPClient\CurlHTTPClient;
use \LINE\LINEBot;
use \LINE\LINEBot\MessageBuilder\TextMessageBuilder;

$channel_token = '2MCOyCeaBipmw3ZzJG8BrsiO4KzCoaoPddMgbZtEu5HHVeIaWU+PDKcCZRJEY76zqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdciLCuKUgV6aFrvAAuuG1mMWe7DCzfEW9FfHQhJR4F/m0AdB04t89/1O/w1cDnyilFU=';
$channel_secret = 'd4afd7da941ac195c155fe67dcb5a338';

// Get message from Line API
$content = file_get_contents('php://input');
$events = json_decode($content, true);

if (!is_null($events['events'])) {

    // Loop through each event
    foreach ($events['events'] as $event) {

        // Line API send a lot of event type, we interested in message only.
        if ($event['type'] == 'message' && $event['message']['type'] == 'text') {

            // Get replyToken
            $replyToken = $event['replyToken'];

            $host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
            $dbname = 'd74bjtc28mea5m';
            $user = 'eozuwfnzmgflmu';
            $pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
            $connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass);

            $params = array(
                'log' => $event['message']['text'],
            );

            $statement = $connection->prepare("INSERT INTO logs (log) VALUES (:log)");
            $result = $statement->execute($params);

            if($result){
                $respMessage = 'Log:'.$event['message']['text'].' Success';
            }else{
                $respMessage = 'Log:'.$event['message']['text'].' Fail';
            }

            $httpClient = new CurlHTTPClient($channel_token);
            $bot = new LINEBot($httpClient, array('channelSecret' => $channel_secret));

            $textMessageBuilder = new TextMessageBuilder($respMessage);
            $response = $bot->replyMessage($replyToken, $textMessageBuilder);

        }
    }
}

echo "OK";
```

โหลดคลาสที่จำเป็นต้องใช้เข้ามา

```php
require_once('./vendor/autoload.php');
```

เรียกใช้ namespace เพื่อว่าเวลาเรียกใช้งานคลาส คำสั่งจะได้ไม่ยาว

```
use \LINE\LINEBot\HTTPClient\CurlHTTPClient;
use \LINE\LINEBot;
use \LINE\LINEBot\MessageBuilder\TextMessageBuilder;
```

channel token และ secret token เอาได้จาก LINE ตรงที่เราสร้างบอท

```
$channel_token = '1v2OUa9tuMIiDhEg57ANbsRaBDbBGP9nlCC+Dpvt5HrsQ+LqcrImWPUBkH8re/pwqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdcjbFurwXGOyHLt8MdgzLfAe7r0BsQV5cATlUanW3OgJewdB04t89/1O/w1cDnyilFU=';
$channel_secret = '9b2c7349ea939ef723a3cb453d774c86';
```

get ข้อมูลที่ทาง LINE API ส่งมาให้แล้วแปลงเป็นอะเรย์ เพราะข้อมูลที่ LINE API ส่งมาให้นั้นอยู่ในรูปแบบของ JSON

```
// Get message from Line API
$content = file_get_contents('php://input');
$events = json_decode($content, true);
```

เพราะ LINE API ส่ง events มาเป็นอะเรย์ เราจึงต้อง loop แยกออกมาทีละ event

```
foreach ($events['events'] as $event)
```

จากนั้นเราจะตรวจสอบอีกครั้งหนึ่งว่า event นั้นๆ type เป็น message และ message type เป็น tex หรือเปล่า

```
if ($event['type'] == 'message' && $event['message']['type'] == 'text')
```

ถ้าหากว่า event type เป็น message และ message type เป็น text \(ถ้าเขาส่ง sticker หรือ รูปภาพมาเราไม่สนใจ\) เราก็เก็บ replyToken ไว้ในตัวแปรก่อน จากนั้นทำการเชื่อมต่อฐานข้อมูล เมื่อเชื่อมต่อได้แล้ว ก็เอา log ไปเก็บไว้ เมื่อเสร็จแล้วก็ส่งคำว่า Success, Fail กลับไปให้ยูสเซอร์

```
// Line API send a lot of event type, we interested in message only.
if ($event['type'] == 'message' && $event['message']['type'] == 'text') {

    // Get replyToken
    $replyToken = $event['replyToken'];

    $host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
    $dbname = 'd74bjtc28mea5m';
    $user = 'eozuwfnzmgflmu';
    $pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
    $connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass);

    $params = array(
        'log' => $event['message']['text'],
    );

    $statement = $connection->prepare("INSERT INTO logs (log) VALUES (:log)");
    $result = $statement->execute($params);

    if($result){
        $respMessage = 'Log:'.$event['message']['text'].' Success';
    }else{
        $respMessage = 'Log:'.$event['message']['text'].' Fail';
    }

    $httpClient = new CurlHTTPClient($channel_token);
    $bot = new LINEBot($httpClient, array('channelSecret' => $channel_secret));

    $textMessageBuilder = new TextMessageBuilder($respMessage);
    $response = $bot->replyMessage($replyToken, $textMessageBuilder);

}
```



