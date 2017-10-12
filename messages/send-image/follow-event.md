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

เห็นมั้ยครับว่า type มันจะเป็น message และ message:type เป็น file รูปแบบทั้งหมดในกลุ่ม type message เราเห็นมาหมดแล้วในบท Receive Message และบท Send Message

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



