# การอัปเดตข้อมูล Postgresql ด้วย PHP

คำสั่งที่ใช้สำหรับอัปเดตข้อมูลนั้นคือคำสั่ง execute ดูตัวอย่าง จะอธิบายเพิ่มเติมทีหลัง

```php
$host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
$dbname = 'd74bjtc28mea5m';
$user = 'eozuwfnzmgflmu';
$pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
$connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass); 

$params = array(
    'name' => $event['message']['text'],
    'slip_date' => date('Y-m-d'),
    'user_id' => $event['source']['userId'],
);
$statement = $connection->prepare('UPDATE slips SET name=:name WHERE slip_date=:slip_date AND user_id=:user_id'); 
$statement->execute($params);

```

คำสั่ง execute จะใช้คู่กับคำสั่ง prepare  

ในคำสั่ง SQL เราจะสังเกตเห็นคำแปลกๆที่มี : นำอยู่ข้างหน้า อย่าง :name, :slip\_date คำที่มี : อยู่ด้านหน้านี้มันจะถูกจับแทนที่ด้วยค่าในตัวแปรอะเรย์ตอนเราเรียกคำสั่ง execute\($params\)

P

