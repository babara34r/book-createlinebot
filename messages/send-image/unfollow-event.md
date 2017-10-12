# Unfollow Events

อีเว้นท์นี้จะเกิดขึ้นเมื่อมียูสเซอร์ลบบอทออกจากความเป็นเพื่อน หรือ บล้อกบอท โดย LINE Server จะส่งเมสเสจหาระบบของเราดังรูปแบบด้านล่าง

```JSON
{
   "events":[
      {
         "type": "unfollow",
         "timestamp": 1462629479859,
         "source": {
           "type": "user",
           "userId": "U206d25c2ea6bd87c17655609a1c37cb8"
         }
       }
   ]
}
```

type จะเป็น unfollow และบอกว่าใครเป็นคนอันเฟรนโดยส่ง userId มาให้ แต่สังเกตมั้ยครับว่าอะไรอย่างหนึ่งที่คุ้นเคยหายไป replyToken ไง LINE ไม่ส่ง replyToken มาให้เพราะถึงยังไงบอทก็ติดต่อคนที่อันเฟรนไม่ได้อยู่แล้ว

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
        if ($event['type'] == 'unfollow') {

            // ไม่รู้จะทำอะไรต่อ เพราะว่ายูสเซอร์อันเฟรนบอทไปแล้ว
            // บางทีอาจจะแค่นับจำนวนคนอันเฟรนบอท แล้วบอกจำนวนให้การตลาดทราบ
            
        }
    }
}

echo "OK";
```



