# การคิวรี่ข้อมูล Postgresql ด้วย PHP

คำสั่งที่ใช้สำหรับคิวรี่ข้อมูลนั้นคือคำสั่ง query ดูตัวอย่าง จะอธิบายเพิ่มเติมทีหลัง

```php
$host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
$dbname = 'd74bjtc28mea5m';
$user = 'eozuwfnzmgflmu';
$pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
$connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass); 

$result = $connection->query("SELECT * FROM polls");

if($result !== null) {
    echo $result->rowCount();
}

```

คำสั่ง query เราจะส่งคำสั่ง SQL เข้าไป ถ้าไม่มีผลลัพธ์ มันจะคืนค่ากลับมาเป็น false แต่ถ้ามีผลลัพธ์มันจะคืนค่ากลับมาเป็น object และเราสามารถใช้คำสั่ง $result-&gt;rowCount\(\) เพื่อตรวจสอบจำนวนแถวที่ได้มา 



ตัวอย่างด้านล่างเราให้ค้นหาเฉพาะข้อมูลแถวที่มี id=10 \(คำสั่ง SQL ธรรมดา\)

```php
$host = 'ec2-174-129-223-193.compute-1.amazonaws.com';
$dbname = 'd74bjtc28mea5m';
$user = 'eozuwfnzmgflmu';
$pass = '2340614a293db8e8a8c02753cd5932cdee45ab90bfcc19d0d306754984cbece1';
$connection = new PDO("pgsql:host=$host;dbname=$dbname", $user, $pass); 

$result = $connection->query("SELECT * FROM polls WHERE id=10");

if($result !== null) {
    echo $result->rowCount();
}

```



