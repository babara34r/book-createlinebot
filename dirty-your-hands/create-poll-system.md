# ตัวอย่างระบบโพล

เงื่อนไขของระบบตัวนี้ก็คือ อนุญาติให้ยูสเซอร์ตอบโพลได้แค่ครั้งเดียวเท่านั้น ไม่อนุญาติให้แก้ไขคำตอบ และหลังจากตอบโพลบอทจะส่งสรุปยอดกลับไปให้ยูสเซอร์ด้วยว่ามีคนตอบเหมือนที่ยูสเซอร์ตอบจำนวนกี่คน

เริ่มแรกให้สร้างฐานข้อมูลและสร้างตารางชื่อ poll มีฟิลด์จำนวน 3 ฟิลด์ id, user\_id, answer

id : serial

user\_id : text

answer : text

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

            try {
                // Check to see user already answer
                $host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
                $dbname = 'd74bjtc28mea5m';
                $user = 'eozuwfnzmgflmu';
                $pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
                $connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass);

                $sql = sprintf("SELECT * FROM poll WHERE user_id='%s' ", $event['source']['userId']);
                $result = $connection->query($sql);

                error_log($sql);

                if($result == false || $result->rowCount() <=0) {

                    switch($event['message']['text']) {

                        case '1':
                            // Insert
                            $params = array(
                                'userID' => $event['source']['userId'],
                                'answer' => '1',
                            );

                            $statement = $connection->prepare('INSERT INTO poll ( user_id, answer ) VALUES ( :userID, :answer )');
                            $statement->execute($params);

                            // Query
                            $sql = sprintf("SELECT * FROM poll WHERE answer='1' AND  user_id='%s' ", $event['source']['userId']);
                            $result = $connection->query($sql);

                            $amount = 1;
                            if($result){
                                $amount = $result->rowCount();
                            }
                            $respMessage = 'จำนวนคนตอบว่าเพื่อน = '.$amount;

                            break;

                        case '2':
                            // Insert
                            $params = array(
                                'userID' => $event['source']['userId'],
                                'answer' => '2',
                            );

                            $statement = $connection->prepare('INSERT INTO poll ( user_id, answer ) VALUES ( :userID, :answer )');
                            $statement->execute($params);

                            // Query
                            $sql = sprintf("SELECT * FROM poll WHERE answer='2' AND  user_id='%s' ", $event['source']['userId']);
                            $result = $connection->query($sql);

                            $amount = 1;
                            if($result){
                                $amount = $result->rowCount();
                            }
                            $respMessage = 'จำนวนคนตอบว่าแฟน = '.$amount;

                            break;

                        case '3':
                            // Insert
                            $params = array(
                                'userID' => $event['source']['userId'],
                                'answer' => '3',
                            );

                            $statement = $connection->prepare('INSERT INTO poll ( user_id, answer ) VALUES ( :userID, :answer )');
                            $statement->execute($params);

                            // Query
                            $sql = sprintf("SELECT * FROM poll WHERE answer='3' AND  user_id='%s' ", $event['source']['userId']);
                            $result = $connection->query($sql);

                            $amount = 1;
                            if($result){
                                $amount = $result->rowCount();
                            }
                            $respMessage = 'จำนวนคนตอบว่าพ่อแม่ = '.$amount;

                            break;
                        case '4':
                            // Insert
                            $params = array(
                                'userID' => $event['source']['userId'],
                                'answer' => '4',
                            );

                            $statement = $connection->prepare('INSERT INTO poll ( user_id, answer ) VALUES ( :userID, :answer )');
                            $statement->execute($params);

                            // Query
                            $sql = sprintf("SELECT * FROM poll WHERE answer='4' AND  user_id='%s' ", $event['source']['userId']);
                            $result = $connection->query($sql);

                            $amount = 1;
                            if($result){
                                $amount = $result->rowCount();
                            }
                            $respMessage = 'จำนวนคนตอบว่าบุคคลอื่นๆ = '.$amount;

                            break;
                        default:
                            $respMessage = "
                                บุคคลที่โทรหาบ่อยที่สุด คือ? \n\r
                                กด 1 เพื่อน \n\r
                                กด 2 แฟน \n\r
                                กด 3 พ่อแม่ \n\r
                                กด 4 บุคคลอื่นๆ \n\r
                            ";
                            break;
                    }

                } else {
                    $respMessage = 'คุณได้ตอบโพลล์นี้แล้ว';
                }

                $httpClient = new CurlHTTPClient($channel_token);
                $bot = new LINEBot($httpClient, array('channelSecret' => $channel_secret));

                $textMessageBuilder = new TextMessageBuilder($respMessage);
                $response = $bot->replyMessage($replyToken, $textMessageBuilder);

            } catch(Exception $e) {
                error_log($e->getMessage());
            }

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

```php
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

จากนั้นตรวจสอบอีกครั้งหนึ่งว่า event นั้นๆ type เป็น message และ message type เป็น tex หรือเปล่า

```php
if ($event['type'] == 'message' && $event['message']['type'] == 'text')
```

ถ้าใช่ เราจะเริ่มด้วยการเชื่อมต่อฐานข้อมูลและไปค้นหาดูในฐานข้อมูลว่า ยูสเซอร์รายนี้ได้ตอบโพลไปแล้วหรือยัง

```php
// Check to see user already answer
$host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
$dbname = 'd74bjtc28mea5m';
$user = 'eozuwfnzmgflmu';
$pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
$connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass);

$sql = sprintf("SELECT * FROM poll WHERE user_id='%s' ", $event['source']['userId']);
$result = $connection->query($sql);
```

ถ้าหากยังไม่เคยตอบโพลเลย

```
if($result == false || $result->rowCount() <=0)
```

ให้เอา message ที่เขาส่งให้บอทมาดูว่าเป็นอะไร

```
switch($event['message']['text'])
```

ถ้าเขาพิมพ์เลข 1 มา ให้เอาไอดีของเขาพร้อมกับคำตอบลงไปบันทึกในฐานข้อมูล แล้วโชว์ข้อมูลกลับไปว่าคนที่ตอบ 1 นั้นทั้งหมดมีกี่คน

```php
case '1':
    // Insert
    $params = array(
        'userID' => $event['source']['userId'],
        'answer' => '1',
    );

    $statement = $connection->prepare('INSERT INTO poll ( user_id, answer ) VALUES ( :userID, :answer )');
    $statement->execute($params);

    // Query
    $sql = sprintf("SELECT * FROM poll WHERE answer='1' AND  user_id='%s' ", $event['source']['userId']);
    $result = $connection->query($sql);

    $amount = 1;
    if($result){
        $amount = $result->rowCount();
    }
    $respMessage = 'จำนวนคนตอบว่าเพื่อน = '.$amount;

    break;
```

ถ้าเขาพิมพ์เลข 2 มา ให้เอาไอดีของเขาพร้อมกับคำตอบลงไปบันทึกในฐานข้อมูล แล้วโชว์ข้อมูลกลับไปว่าคนที่ตอบ 2 นั้นทั้งหมดมีกี่คน

```php
case '2':
    // Insert
    $params = array(
        'userID' => $event['source']['userId'],
        'answer' => '2',
    );

    $statement = $connection->prepare('INSERT INTO poll ( user_id, answer ) VALUES ( :userID, :answer )');
    $statement->execute($params);

    // Query
    $sql = sprintf("SELECT * FROM poll WHERE answer='2' AND  user_id='%s' ", $event['source']['userId']);
    $result = $connection->query($sql);

    $amount = 1;
    if($result){
        $amount = $result->rowCount();
    }
    $respMessage = 'จำนวนคนตอบว่าแฟน = '.$amount;

    break;
```

ถ้าเขาพิมพ์เลข 3 มา ให้เอาไอดีของเขาพร้อมกับคำตอบลงไปบันทึกในฐานข้อมูล แล้วโชว์ข้อมูลกลับไปว่าคนที่ตอบ 3 นั้นทั้งหมดมีกี่คน

```php
case '3':
    // Insert
    $params = array(
        'userID' => $event['source']['userId'],
        'answer' => '3',
    );

    $statement = $connection->prepare('INSERT INTO poll ( user_id, answer ) VALUES ( :userID, :answer )');
    $statement->execute($params);

    // Query
    $sql = sprintf("SELECT * FROM poll WHERE answer='3' AND  user_id='%s' ", $event['source']['userId']);
    $result = $connection->query($sql);

    $amount = 1;
    if($result){
        $amount = $result->rowCount();
    }
    $respMessage = 'จำนวนคนตอบว่าพ่อแม่ = '.$amount;

    break;
```

ถ้าเขาพิมพ์เลข 4 มา ให้เอาไอดีของเขาพร้อมกับคำตอบลงไปบันทึกในฐานข้อมูล แล้วโชว์ข้อมูลกลับไปว่าคนที่ตอบ 4 นั้นทั้งหมดมีกี่คน

```php
case '4':
    // Insert
    $params = array(
        'userID' => $event['source']['userId'],
        'answer' => '4',
    );

    $statement = $connection->prepare('INSERT INTO poll ( user_id, answer ) VALUES ( :userID, :answer )');
    $statement->execute($params);

    // Query
    $sql = sprintf("SELECT * FROM poll WHERE answer='4' AND  user_id='%s' ", $event['source']['userId']);
    $result = $connection->query($sql);

    $amount = 1;
    if($result){
        $amount = $result->rowCount();
    }
    $respMessage = 'จำนวนคนตอบว่าบุคคลอื่นๆ = '.$amount;

    break;
```

แต่ถ้าหากที่เขาพิมพ์มานั้น ไม่ใช่ 1, 2, 3 หรือ 4 ให้ส่งคำถามของโพลกลับไปให้ใหม่

```php
default:
    $respMessage = "
        บุคคลที่โทรหาบ่อยที่สุด คือ? \n\r
        กด 1 เพื่อน \n\r
        กด 2 แฟน \n\r
        กด 3 พ่อแม่ \n\r
        กด 4 บุคคลอื่นๆ \n\r
    ";
    break;
```

ทีนี้ในกรณีเขาเคยตอบโพลนั้นแล้วล่ะ ก็ให้ส่งข้อความไปหาบอกว่า "คุณได้ตอบโพลล์นี้แล้ว"

```php
} else {
    $respMessage = 'คุณได้ตอบโพลล์นี้แล้ว';
}
```

ส่วนของโค้ดที่ใช้สำหรับส่งข้อความกลับไปหา LINE API

```php
$httpClient = new CurlHTTPClient($channel_token);
$bot = new LINEBot($httpClient, array('channelSecret' => $channel_secret));

$textMessageBuilder = new TextMessageBuilder($respMessage);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);
```

เราจะเห็นว่า ถ้าหากเราต้องการส่งข้อความที่มีการขึ้นบรรทัดใหม่ สามารถส่ง \r\n เข้าไปตรงตำแหน่งที่ต้องการขึ้นบรรทัดใหม่ และข้อความจะต้องคร่อมด้วยเครื่องหมาย \( " \) สองเขา เท่านั้น

P

