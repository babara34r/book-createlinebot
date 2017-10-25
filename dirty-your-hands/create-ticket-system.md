# ตัวอย่างระบบลงบันทึกประจำวัน

ระบบนี้จะอนุญาติให้ยูสเซอร์ป้อนข้อมูลว่าเมื่อไร ทำอะไร โดยจะเอาข้อมูลลงไปเก็บในฐานข้อมูล ผมทำตัวอย่างระบบไว้สั้นๆ เพื่อให้ง่ายในการอ่านโค้ด ตัวนี้ถ้าหากจะทำให้สมบูรณ์กว่านี้ เราก็ไปทำหน้าเว็บให้ดึงข้อมูลที่ถูกบันทึกไว้ออกมาแสดง จะจัดเรียง จะทำอย่างไรก็ว่าไป ตามแต่จะจินตนาการ

ในเบื้องต้นให้สร้างฐานข้อมูล และ สร้างตารางชื่อ appointments มีฟิลด์ 3 ฟิลด์ id, time, content

id : serial

time : text

content : text

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

            // Split message then keep it in database.
            $appointments = explode(',', $event['message']['text']);

            if(count($appointments) == 2) {

                $host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
                $dbname = 'd74bjtc28mea5m';
                $user = 'eozuwfnzmgflmu';
                $pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
                $connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass);

                $params = array(
                    'time' => $appointments[0],
                    'content' => $appointments[1],
                );

                $statement = $connection->prepare("INSERT INTO appointments (time, content) VALUES (:time, :content)");
                $result = $statement->execute($params);

                $respMessage = 'Your appointment has saved.';
            }else{
                $respMessage = 'You can send appointment like this "12.00,House keeping." ';
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

```php
use \LINE\LINEBot\HTTPClient\CurlHTTPClient;
use \LINE\LINEBot;
use \LINE\LINEBot\MessageBuilder\TextMessageBuilder;
```

สร้างตัวแปรเก็บ channel token และ secret token เอาไว้จะได้ใช้ง่ายๆ

```
$channel_token = '1v2OUa9tuMIiDhEg57ANbsRaBDbBGP9nlCC+Dpvt5HrsQ+LqcrImWPUBkH8re/pwqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdcjbFurwXGOyHLt8MdgzLfAe7r0BsQV5cATlUanW3OgJewdB04t89/1O/w1cDnyilFU=';
$channel_secret = '9b2c7349ea939ef723a3cb453d774c86';
```

get ข้อมูลที่ทาง LINE API ส่งมาให้แล้วแปลงเป็นอะเรย์

```php
// Get message from Line API
$content = file_get_contents('php://input');
$events = json_decode($content, true);
```

Loop แตก events ที่ LINE API ส่งมาให้ออกมาทีละ event

```php
foreach ($events['events'] as $event)
```

จากนั้นเราจะตรวจสอบอีกครั้งหนึ่งว่า event นั้นๆ type เป็น message และ message type เป็น tex หรือเปล่า

```php
if ($event['type'] == 'message' && $event['message']['type'] == 'text')
```

ถ้าหากว่า event type เป็น message และ message type เป็น text \(ถ้าเขาส่ง sticker หรือ รูปภาพมาเราไม่สนใจ\) เราก็เก็บ replyToken ไว้ในตัวแปรก่อน จากนั้นเราจะระเบิด message ที่ทาง LINE API ส่งมาให้ออกเป็นอะเรย์โดยใช้คอมม่าเป็นตัวแยก  แล้วก็ตรวจสอบว่าอะเรย์ที่เราระเบิดออกนั้นมี 2 ตัวหรือเปล่า ถ้ามีน้อยกว่าแสดงว่าตอนยูสเซอร์คุยกับบอทเขาป้อนผิดฟอร์แมตที่เรากำหนดให้ ถ้าหากว่าฟอร์แมตถูกต้อง เราก็เชื่อมต่อฐานข้อมูลแล้วเอาอะเรย์ตัวแรกใส่ในฟิลด์ time อะเรย์ตัวที่ 2 ใส่ในฟิลด์ content

หลังจากบันทึกข้อมูลลงฐานข้อมูลเรียบร้อยแล้ว ก็ส่ง message กลับไปหายูสเซอร์ว่า "Your appointment has saved." หรือจะเปลี่ยนคำเป็นภาษาไทยก็ได้นะ ไม่เออเร่อ

```php
// Line API send a lot of event type, we interested in message only.
if ($event['type'] == 'message' && $event['message']['type'] == 'text') {

    // Get replyToken
    $replyToken = $event['replyToken'];

    // Split message then keep it in database.
    $appointments = explode(',', $event['message']['text']);

    if(count($appointments) == 2) {

        $host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
        $dbname = 'd74bjtc28mea5m';
        $user = 'eozuwfnzmgflmu';
        $pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
        $connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass);

        $params = array(
            'time' => $appointments[0],
            'content' => $appointments[1],
        );

        $statement = $connection->prepare("INSERT INTO appointments (time, content) VALUES (:time, :content)");
        $result = $statement->execute($params);

        $respMessage = 'Your appointment has saved.';
    }else{
        $respMessage = 'You can send appointment like this "12.00,House keeping." ';
    }

    $httpClient = new CurlHTTPClient($channel_token);
    $bot = new LINEBot($httpClient, array('channelSecret' => $channel_secret));

    $textMessageBuilder = new TextMessageBuilder($respMessage);
    $response = $bot->replyMessage($replyToken, $textMessageBuilder);

}
```

P

