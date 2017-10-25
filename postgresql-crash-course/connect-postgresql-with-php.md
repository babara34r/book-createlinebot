# การเชื่อมต่อฐานข้อมูล Postgresql ด้วย PHP

เราจะใช้คลาส PDO เป็นตัวเชื่อมต่อฐานข้อมูล คำสั่งเพียงคำสั่งเดียวก็อยู่หมัด new PDO

```php
$host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
$dbname = 'd74bjtc28mea5m';
$user = 'eozuwfnzmgflmu';
$pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
$connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass); 

$params = array(
    'log' => $event['message']['text'],
);
$statement = $connection->prepare("INSERT INTO logs (log) VALUES (:log)");
$result = $statement->execute($params);
```



