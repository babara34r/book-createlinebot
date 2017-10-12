# Follow Events

อีเว้นท์นี้จะเกิดขึ้นเมื่อมียูสเซอร์เพิ่มบอทเป็นเพื่อน หรือ ปลดบล้อกบอท โดย LINE Server จะส่งเมสเสจหาระบบของเราดังรูปแบบด้านล่าง

```JSON
{
   "events":[
      {
       "replyToken": "nHuyWiB7yP5Zw52FIkcQobQuGDXCTA",
       "type": "follow",
       "timestamp": 1462629479859,
       "source": {
         "type": "user",
         "userId": "U206d25c2ea6bd87c17655609a1c37cb8"
       }
     }
   ]
}
```

type จะเป็น follow และ LINE ยังใจดีแนบ userId มาบอกด้วยว่าใครเป็นคนเพิ่มบอทเป็นเพื่อน

## ตัวอย่างการเขียนโค้ด

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
        if ($event['type'] == 'follow') {

            // Get replyToken
            $replyToken = $event['replyToken'];
            
            // Greeting
            $respMessage = 'Thanks you. I try to be your best friend.';

            $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
            $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

            $textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
            $response = $bot->replyMessage($replyToken, $textMessageBuilder);
        }
    }
}

echo "OK";
```



