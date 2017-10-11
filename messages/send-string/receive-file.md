# เขียนโค้ดรับไฟล์จาก LINE Server

ข้อความที่ LINE Server ส่งมาให้เรา

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
            "type":"file",
            "fileName":"file.txt",
            "fileSize":2138
         }
      }
   ]
}
```

จะเห็นว่าตรง message จะมี id ของไฟล์พร้อมทั้งชื่อไฟล์ ถ้าหากเราต้องการเก็บไฟล์ไว้ใช้ภายหลัง เราจะต้องเขียนคำสั่งไปดึงเอาเนื้อของไฟล์ที่ LINE Server เก็บไว้ออกมาแล้วเขียนลงไฟล์เก็บไว้เป็นของเรา เพราะว่าไฟล์ที่ LINE Server เก็บไว้นั้น สักพักมันจะลบทิ้ง

\*\*\* สิ่งที่ต้องจำ LINE Server จะลบไฟล์ทิ้งเมื่อถึงช่วงระยะเวลาหนึ่ง เพราะฉะนั้นถ้าต้องการเก็บไฟล์ไว้ จะต้องเขียนไฟล์เก็บไว้ที่เซิฟเวอร์ตัวเอง

## เปิดไฟล์ index.php ขึ้นมาแล้วเขียนโค้ดดังนี้

```php
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

                case 'file':
                    $messageID = $event['message']['id'];
                    $fileName = $event['message']['fileName'];

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
                    $respMessage = 'Hello, your file ID is '. $messageID . ' and file name is '. $fileName;

                    break;
                default:
                    // Reply message
                    $respMessage = 'Please send file only';
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
$fileName = $event['message']['fileName'];
$respMessage = 'Hello, your file ID is '. $messageID . ' and file name is '. $fileName;
```

เก็บ id กับ fileName ของไฟล์มาเราจะส่งกลับไปให้ยูสเซอร์ชม 

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

ถ้าหากต้องการเก็บไฟล์ไว้ในเครื่องเซิฟเวอร์ ให้เปิดคอมเม้นท์โค้ดชุดนี้ แต่ก็ต้องมั่นใจนะว่าโฟลเดอร์มี permission สามารถเขียนไฟล์ได้

```php
$httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
$bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

$textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);
```

LINE SDK สำหรับส่งข้อความกลับไปหายูสเซอร์

P

