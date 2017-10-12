# เขียนโค้ดรับ Location จาก LINE Server

ข้อความที่ LINE Server ส่งมาให้เราจะเป็นรูปแบบนี้

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
            "type":"location",
            "title":"my location",
            "address":"〒150-0002 東京都渋谷区渋谷２丁目２１−１",
            "latitude":35.65910807942215,
            "longitude":139.70372892916203
         }
      }
   ]
}
```

เมื่อมีคนแชร์ Location ให้บอท LINE Server จะส่งข้อความต่อมาให้ระบบของเรา ซึ่งรูปแบบเป็นไปตามด้านบน ข้อมูจะมีครบที่ชื่อสถานที่ ละติจูด และลองจิจูด

## เปิดไฟล์ index.php ขึ้นมาแล้วเขียนโค้ดดังนี้

เราลองเขียนโค้ดเพื่อรับเมสเสจจาก LINE Server 

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
                
                case 'location':
                    $address = $event['message']['address'];

                    // Reply message
                    $respMessage = 'Hello, your address is '. $address;
            
                    break;
                default:
                    // Reply message
                    $respMessage = 'Please send location only';
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

เสร็จแล้ว push โค้ดขึ้น repository รอสักแป้บให้ทาง Heroku ดึงโค้ดส่งขึ้นโฮสต์ก่อน จากนั้นส่ง Location ให้บอทดู

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
case 'location':
```

ตรวจสอบ type ว่าเป็น location

```php
$address = $event['message']['address'];

// Reply message
$respMessage = 'Hello, your address is '. $address;
```

ลองเอาค่าที่เขาส่งมา ส่งกลับไปให้ยูสเซอร์ดู





```php
$httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
$bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

$textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);
```

LINE SDK สำหรับส่งข้อความกลับไปหายูสเซอร์

P

