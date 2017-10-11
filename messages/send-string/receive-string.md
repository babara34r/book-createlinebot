# เขียนโค้ดรับข้อความจาก LINE Server

ก่อนอื่นผมให้ดูรูปแบบข้อความที่ทาง LINE Server จะส่งมาให้เรา ซึ่งมันจะอยู่ในรูปแบบของ JSON

```
{
  "events": [
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
             "type": "text",
             "text": "Hello, world"
          }
      }
  ]
}

```

จะเห็นว่า LINE นั้นส่ง events มาเป็นอะเรย์ \(ไลน์จะเรียกแต่ละเมสเสจว่า event\) ฉะนั้นเวลาเราเขียนโค้ดเราต้อง loop  ภายในเมสเสจจะมี replyToken มาให้ด้วย ซึ่ง replyToken นี้จะใช้ส่งข้อความกลับไปให้ LINE Server ได้เพียงครั้งเดียวเท่านั้น และมีอายุจำกัด

type : ประเภทของ event  
userID : ไอดีของผู้ส่งข้อความมา  
message:type : ประเภทของเมสเสจ \(ประกอบไปด้วย 7 ประเภท text, image, sticker, file, video, audio, location \)   
message:text : ข้อความที่คนส่งหาบอท



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

                        switch($event['message']['type']) {
                            
                            case 'text':
                                // Get replyToken
                                $replyToken = $event['replyToken'];
                    
                                // Reply message
                                $respMessage = 'Hello, your message is '. $event['message']['text'];
                        
                                $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
                                $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));
                    
                                $textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
                                $response = $bot->replyMessage($replyToken, $textMessageBuilder);
                                
                                break;
                        }
		}
	}
}

echo "OK";

```

เสร็จแล้ว push โค้ดขึ้น repository รอสักแป้บให้ทาง Heroku ดึงโค้ดส่งขึ้นโฮสต์ก่อน ลองทักบอทไปดูด้วยคำอะไรก็ได้ ถ้ามีคำตอบกลับมา แสดงว่าบอทเราทำงานแล้ว 



## อธิบายโค้ด

```
require_once('./vendor/autoload.php');
```

ทำการ include ไฟล์ LINEBot SDK เข้ามา

```
$content = file_get_contents('php://input');
$events = json_decode($content, true);
```

get ค่าที่ LINE Server ส่งมาให้แล้วแปลงจาก JSON ไปเป็น Array



















1.

