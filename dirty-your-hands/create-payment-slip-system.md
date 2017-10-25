# ตัวอย่างระบบ payment slip

ระบบนี้เป็นระบบแจ้งการโอนเงิน โดยระบบจะอนุญาติให้ยูสเซอร์แจ้งการโอนเงินได้วันละ 1 รายการเท่านั้น ดูให้ดีนะครับไม่จำกัดจำนวนครั้งแต่จำกัดจำนวนรายการ หากมีการส่งสลิปมา 2 ครั้งเราจะเขียนทับสลิปเดิม หากมีการส่งชื่อมา 2 ครั้งเราก็เขียนทับเรคคอร์ดเดิม

ขั้นแรกให้สร้างฐานข้อมูลและสร้างตารางชื่อ slips ประกอบด้วยฟิลด์ id, name, user-id, slip-name, image

id : serial

name : text

user-id : text

slip-name : text

image : text

```
<?php

require_once('./vendor/autoload.php');

use \LINE\LINEBot\HTTPClient\CurlHTTPClient;
use \LINE\LINEBot;
use \LINE\LINEBot\MessageBuilder\TextMessageBuilder;

// Token
$channel_token = '2MCOyCeaBipmw3ZzJG8BrsiO4KzCoaoPddMgbZtEu5HHVeIaWU+PDKcCZRJEY76zqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdciLCuKUgV6aFrvAAuuG1mMWe7DCzfEW9FfHQhJR4F/m0AdB04t89/1O/w1cDnyilFU=';
$channel_secret = 'd4afd7da941ac195c155fe67dcb5a338';

// Create bot
$httpClient = new CurlHTTPClient($channel_token);
$bot = new LINEBot($httpClient, array('channelSecret' => $channel_secret));

// Database connection
$host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
$dbname = 'd74bjtc28mea5m';
$user = 'eozuwfnzmgflmu';
$pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
$connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass);

// Get message from Line API
$content = file_get_contents('php://input');
$events = json_decode($content, true);

if (!is_null($events['events'])) {

    // Loop through each event
    foreach ($events['events'] as $event) {

          // Line API send a lot of event type, we interested in message only.
        if ($event['type'] == 'message') {

                switch($event['message']['type']) {
                     case 'text':

                          $sql = sprintf(
                                "SELECT * FROM slips WHERE slip_date='%s' AND user_id='%s' ",
                                date('Y-m-d'),
                                $event['source']['userId']);
                          $result = $connection->query($sql);

                          if($result !== false && $result->rowCount() >0) {
                                // Save database
                                $params = array(
                                     'name' => $event['message']['text'],
                                     'slip_date' => date('Y-m-d'),
                                     'user_id' => $event['source']['userId'],
                                );
                                $statement = $connection->prepare('UPDATE slips SET name=:name WHERE slip_date=:slip_date AND user_id=:user_id');
                                $statement->execute($params);
                          } else {
                                $params = array(
                                     'user_id' => $event['source']['userId'] ,
                                     'slip_date' => date('Y-m-d'),
                                     'name' => $event['message']['text'],
                                );
                                $statement = $connection->prepare('INSERT INTO slips (user_id, slip_date, name) VALUES (:user_id, :slip_date, :name)');

                                $effect = $statement->execute($params);
                          }

                          // Bot response
                          $respMessage = 'Your data has saved.';
                          $replyToken = $event['replyToken'];
                          $textMessageBuilder = new TextMessageBuilder($respMessage);
                          $response = $bot->replyMessage($replyToken, $textMessageBuilder);

                          break;
                     case 'image':

                          // Get file content.
                          $fileID = $event['message']['id'];

                          $response = $bot->getMessageContent($fileID);
                          $fileName = md5(date('Y-m-d')).'.jpg';

                          if ($response->isSucceeded()) {
                                // Create file.
                                $file = fopen($fileName, 'w');
                                fwrite($file, $response->getRawBody());

                                $sql = sprintf(
                                                "SELECT * FROM slips WHERE slip_date='%s' AND user_id='%s' ",
                                                date('Y-m-d'),
                                                $event['source']['userId']);
                                $result = $connection->query($sql);

                                if($result !== false && $result->rowCount() >0) {
                                     // Save database
                                     $params = array(
                                          'image' => $fileName,
                                          'slip_date' => date('Y-m-d'),
                                          'user_id' => $event['source']['userId'],
                                     );
                                     $statement = $connection->prepare('UPDATE slips SET image=:image WHERE slip_date=:slip_date AND user_id=:user_id');
                                     $statement->execute($params);

                                } else {
                                     $params = array(
                                          'user_id' => $event['source']['userId'] ,
                                          'image' => $fileName,
                                          'slip_date' => date('Y-m-d'),
                                     );
                                     $statement = $connection->prepare('INSERT INTO slips (user_id, image, slip_date) VALUES (:user_id, :image, :slip_date)');
                                     $statement->execute($params);
                                }
                          }

                          // Bot response
                          $respMessage = 'Your data has saved.';
                          $replyToken = $event['replyToken'];
                          $textMessageBuilder = new TextMessageBuilder($respMessage);
                          $response = $bot->replyMessage($replyToken, $textMessageBuilder);

                          break;
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

สร้างตัวแปร bot ไว้ใช้งาน

```php
// Create bot
$httpClient = new CurlHTTPClient($channel_token);
$bot = new LINEBot($httpClient, array('channelSecret' => $channel_secret));
```

สร้างการเชื่อมต่อฐานข้อมูลรอไว้

```php
// Database connection
$host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
$dbname = 'd74bjtc28mea5m';
$user = 'eozuwfnzmgflmu';
$pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
$connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass);
```

get ข้อมูลที่ทาง LINE API ส่งมาให้แล้วแปลงเป็นอะเรย์

```php
// Get message from Line API
$content = file_get_contents('php://input');
$events = json_decode($content, true);
```

ถ้าหากว่ามีการส่งข้อมูลมาจาก LINE API

```php
if (!is_null($events['events']))
```

loop มันออกมาทีละตัว

```php
foreach ($events['events'] as $event)
```

ตรวจสอบ event type ว่าเป็น message ใช่ไหม เพราะว่า event type ที่ LINE API ส่งมาให้นั้นไม่ได้มีแต่ message อย่างเดียว แต่เราต้องการแค่ message เท่านั้น

```php
// Line API send a lot of event type, we interested in message only.
if ($event['type'] == 'message')
```

เมื่อเรารู้แล้วว่ามันเป็น message เราก็เอามันมาแยกประเภทต่อ

```php
switch($event['message']['type'])
```

ถ้าหากว่าที่ยูสเซอร์ส่งมาให้บอทนั้นเป็นข้อความ เราจะตีความไปเลยว่าเขาส่งชื่อผู้โอนเงินเข้ามา แต่ก่อนที่จะเอาชื่อไปเก็บในฐานข้อมูล เราต้องตรวจดูในฐานข้อมูลเสียก่อนว่ามีเรคคอร์ดของวันนี้หรือยัง ถ้ามีแล้วเราจะใช้คำสั่ง update ถ้าหากยังไม่มีเราจะใช้คำสั่ง insert ลงไปในฟิลด์

หลังบันทึกข้อมูลเสร็จก็ส่ง message ไปบอกยูสเซอร์ว่า "Your data has saved."

```php
case 'text':

    $sql = sprintf(
        "SELECT * FROM slips WHERE slip_date='%s' AND user_id='%s' ",
        date('Y-m-d'),
        $event['source']['userId']);
    $result = $connection->query($sql);

    if($result !== false && $result->rowCount() >0) {
        // Save database
        $params = array(
                'name' => $event['message']['text'],
                'slip_date' => date('Y-m-d'),
                'user_id' => $event['source']['userId'],
        );
        $statement = $connection->prepare('UPDATE slips SET name=:name WHERE slip_date=:slip_date AND user_id=:user_id');
        $statement->execute($params);
    } else {
        $params = array(
                'user_id' => $event['source']['userId'] ,
                'slip_date' => date('Y-m-d'),
                'name' => $event['message']['text'],
        );
        $statement = $connection->prepare('INSERT INTO slips (user_id, slip_date, name) VALUES (:user_id, :slip_date, :name)');

        $effect = $statement->execute($params);
    }

    // Bot response
    $respMessage = 'Your data has saved.';
    $replyToken = $event['replyToken'];
    $textMessageBuilder = new TextMessageBuilder($respMessage);
    $response = $bot->replyMessage($replyToken, $textMessageBuilder);

    break;
```

ทีนี้ถ้าหากว่าที่ยูสเซอร์ส่งมานั้นเป็นรูปภาพ เราจะคิดเลยว่ามันเป็นใบสลิป ก็ให้เรียก LINEBot API ไปดึงเอารูปภาพจาก LINE Server ออกมา เขียนเป็นไฟล์ไว้บนเซิฟเวอร์ของเรา หลังเขียนไฟล์เสร็จก็ให้ตรวจเช็กในฐานข้อมูลก่อนว่ามีเรคอร์ดอยู่แล้วหรือไม่ ถ้าหากมีแล้วเราจะใช้คำสั่ง update ถ้าหากยังไม่มีเราจะใช้คำสั่ง insert บันทึกข้อมูลเก็บไว้ในฐานข้อมูล

หลังบันทึกข้อมูลเรียบร้อยแล้ว ส่งข้อความไปบอกยูสเซอร์ว่า "Your data has saved."

```php
case 'image':

    // Get file content.
    $fileID = $event['message']['id'];

    $response = $bot->getMessageContent($fileID);
    $fileName = md5(date('Y-m-d')).'.jpg';

    if ($response->isSucceeded()) {
        // Create file.
        $file = fopen($fileName, 'w');
        fwrite($file, $response->getRawBody());

        $sql = sprintf(
                        "SELECT * FROM slips WHERE slip_date='%s' AND user_id='%s' ",
                        date('Y-m-d'),
                        $event['source']['userId']);
        $result = $connection->query($sql);

        if($result !== false && $result->rowCount() >0) {
                // Save database
                $params = array(
                    'image' => $fileName,
                    'slip_date' => date('Y-m-d'),
                    'user_id' => $event['source']['userId'],
                );
                $statement = $connection->prepare('UPDATE slips SET image=:image WHERE slip_date=:slip_date AND user_id=:user_id');
                $statement->execute($params);

        } else {
                $params = array(
                    'user_id' => $event['source']['userId'] ,
                    'image' => $fileName,
                    'slip_date' => date('Y-m-d'),
                );
                $statement = $connection->prepare('INSERT INTO slips (user_id, image, slip_date) VALUES (:user_id, :image, :slip_date)');
                $statement->execute($params);
        }
    }

    // Bot response
    $respMessage = 'Your data has saved.';
    $replyToken = $event['replyToken'];
    $textMessageBuilder = new TextMessageBuilder($respMessage);
    $response = $bot->replyMessage($replyToken, $textMessageBuilder);

    break;
```

ระบบที่ทำขึ้นมานี้เพื่อให้เป็นตัวอย่างง่ายๆ ท่านอาจจะต่อยอด สร้างระบบหลังบ้าน เปลี่ยนเงื่อนไขให้ยูสเซอร์สามารถส่งสลิปต์กี่ใบก็ได้ หรือจะทำให้มันเชื่อมต่อกับ woocommerce ให้มันอัปเดตออเดอร์ให้อัตโนมัติ ก็ยังได้

รูปภาพที่บอทเก็บไว้ท่านสามารถเปิดหน้าเว็บใน heroku พิมพ์ชื่อไฟล์เอา

P

