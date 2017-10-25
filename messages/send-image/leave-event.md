# Leave Events

อีเว้นท์นี้จะเกิดขึ้นเมื่อบอทออกจากกลุ่ม หรือโดนถีบกระเด็นออกมา โดย LINE Server จะส่งเมสเสจหาระบบของเราดังรูปแบบด้านล่าง

```JSON
{
   "events":[
      {
         "type": "leave",
         "timestamp": 1462629479859,
         "source": {
           "type": "group",
           "groupId": "cxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
         }
       }
   ]
}
```

type จะเป็น leave และไม่มี replyToken

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
        if ($event['type'] == 'leave') {

            // ไม่รู้จะทำอะไรต่อ อาจจะแค่บันทึกไว้ในฐานข้อมูลว่า บอทได้ออกจากกลุ่มหมายเลขอะไร

        }
    }
}

echo "OK";
```



