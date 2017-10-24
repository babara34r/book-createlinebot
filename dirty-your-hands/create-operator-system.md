# ตัวอย่างระบบโอเปอเรเตอร์

เป็นระบบแบบง่ายๆให้ยูสเซอร์ถามบอทได้ 4 คำถาม

tel - บอทจะตอบเบอร์โทรสำหรับติดต่อกลับไป

address - บอทจะตอบที่อยู่บริษัทกลับไป

boss - บอทจะตอบเบอร์โทรเจ้านายกลับไป

idcard - บอทจะตอบหมายเลขผู้เสียภาษีไปให้

ระบบนี้ถ้าหากเราจะเอาไปต่อยอด ก็อาจจะทำในส่วนของคำถามและคำตอบให้อยู่ในฐานข้อมูล มีระบบหลังบ้านให้ใช้ผ่านเว็บ อยากให้บอทฉลาดขึ้นก็เพิ่มคำตอบใหม่ๆเข้าไป

```php
<?php
/**
 * Use for return easy answer.
 */

require_once('./vendor/autoload.php');

use \LINE\LINEBot\HTTPClient\CurlHTTPClient;
use \LINE\LINEBot;
use \LINE\LINEBot\MessageBuilder\TextMessageBuilder;

$channel_token = '1v2OUa9tuMIiDhEg57ANbsRaBDbBGP9nlCC+Dpvt5HrsQ+LqcrImWPUBkH8re/pwqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdcjbFurwXGOyHLt8MdgzLfAe7r0BsQV5cATlUanW3OgJewdB04t89/1O/w1cDnyilFU=';
$channel_secret = '9b2c7349ea939ef723a3cb453d774c86';

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

            switch($event['message']['text']) {

                case 'tel':
                    $respMessage = '089-5124512';
                    break;
                case 'address':
                    $respMessage = '99/451 Muang Nonthaburi';
                    break;
                case 'boss':
                    $respMessage = '089-2541545';
                    break;
                case 'idcard':
                    $respMessage = '5845122451245';
                    break;
                default:
                    break;
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

จากนั้นเราจะตรวจสอบอีกครั้งหนึ่งว่า event นั้นๆ type เป็น message และ message type เป็น tex หรือเปล่า  หากย้อนกลับไปอ่านช่วงต้นๆของหนังสือผมจะเขียนอธิบายไว้แล้วว่า event ที่เกิดขึ้นนั้นไม่ได้มีเฉพาะการพูดคุยโต้ตอบกับบอทเท่านั้น แต่การที่บอทถูกถีบออกจากห้องแชท LINE API ก็จะส่ง event หาเรา รูปแบบคล้ายๆ message แต่ต่างกันตรงที่ type

```
if ($event['type'] == 'message' && $event['message']['type'] == 'text')
```

ถ้าหากว่าเป็นการพิมพ์ถามคำถามบอทเข้ามา ก็ให้บอทตรวจสอบว่าเขาถามอะไร โดยใช้คำสั่ง switch case เป็นตัวหา เมื่อได้คำถามว่าเขาถามอะไรก็ให้ส่งคำตอบกลับไปให้ ส่วนคำตอบที่อยู่นอกเหนือจากที่เตรียมไว้ ก็ไม่ต้องตอบอะไร

```
// Get replyToken
$replyToken = $event['replyToken'];

switch($event['message']['text']) {

    case 'tel':
        $respMessage = '089-5124512';
        break;
    case 'address':
        $respMessage = '99/451 Muang Nonthaburi';
        break;
    case 'boss':
        $respMessage = '089-2541545';
        break;
    case 'idcard':
        $respMessage = '5845122451245';
        break;
    default:
        break;
}

$httpClient = new CurlHTTPClient($channel_token);
$bot = new LINEBot($httpClient, array('channelSecret' => $channel_secret));

$textMessageBuilder = new TextMessageBuilder($respMessage);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);
```



