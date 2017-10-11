# Composer ฉบับเร่งด่วน

Composer เป็นเครื่องมือตัวหนึ่งที่ช่วยเราในการดาวน์โหลด คลาสที่เขาแจกฟรี ลงมายังเครื่องเรารวมทั้งจะช่วยดาวน์โหลดไฟล์ที่มัน depend กับคลาสที่เราต้องการให้ด้วย 

## การติดตั้ง Composer บน Mac

1. เปิด Terminal ขึ้นมาแล้วพิมพ์คำสั่ง แล้วกด Enter  
   `brew install composer`  
   ถ้าหากมีเออเร่อแจ้งกลับมาว่า

   ```
   composer: Missing PHP53, PHP54 or PHP55 from homebrew-php. Please install one of them before continuing
   Error: An unsatisfied requirement failed this build.
   ```

   ให้พิมพ์คำสั่ง  
   `brew install php54`

2. หลังจากรอการติดตั้งเสร็จเรียบร้อยแล้ว ให้ลองพิมพ์คำสั่ง  
   `composer --version`ใน Teminal ควรแสดงหมายเลขเวอร์ชั่นของ Composer ออกมา

## การติดตั้ง Composer บน Windows

1. ดาวน์โหลด Composer มาก่อน โดยเข้าไปที่ URL นี้ [https://getcomposer.org/download](https://getcomposer.org/download)

2. คลิกลิ้งก์ Composer-Setup.exe

   ![](http://www.select2web.com/wp-content/uploads/2016-05-23_16-01-31-1024x646.png "2016-05-23\_16-01-31")

3. หน้าต่างดาวน์โหลดไฟล์จะแสดงขึ้นมา บันทึกไฟล์ไว้ที่ตรงไหนสักแห่งที่หามันได้ง่ายๆ

4. ดับเบิลคลิกไฟล์ที่ดาวน์โหลดมา แล้วคลิก Next 
   ![](http://www.select2web.com/wp-content/uploads/2016-05-13_10-26-45.png "2016-05-13\_10-26-45")
5. คลิก Next

   ![](http://www.select2web.com/wp-content/uploads/2016-05-13_10-27-09.png "2016-05-13\_10-27-09")

6. ในกรณีอินเตอร์เน็ตที่ใช้อยู่ต้องวิ่งผ่าน proxy จึงจะกำหนดค่า proxy แต่ถ้าอินเตอร์เน็ตไม่ได้วิ่งผ่าน proxy ก็คลิก Next   
   ![](http://www.select2web.com/wp-content/uploads/2016-05-13_10-27-48.png "2016-05-13\_10-27-48")

7. คลิก Install 
   ![](http://www.select2web.com/wp-content/uploads/2016-05-13_10-28-27.png "2016-05-13\_10-28-27")
8. รอระหว่างที่ Coposer กำลังติดตั้ง 
   ![](http://www.select2web.com/wp-content/uploads/2016-05-13_10-28-56.png "2016-05-13\_10-28-56")
9. ไม่ต้องสนใจอะไร คลิก Next ไป 
   ![](http://www.select2web.com/wp-content/uploads/2016-05-13_10-29-10.png "2016-05-13\_10-29-10")
10. คลิก Next 
    ![](http://www.select2web.com/wp-content/uploads/2016-05-13_10-29-36.png "2016-05-13\_10-29-36")
11. คลิก Finish เสร็จสิ้นกระบวนการติดตั้ง 
    ![](http://www.select2web.com/wp-content/uploads/2016-05-13_10-30-04.png "2016-05-13\_10-30-04")

## การใช้งาน Composer

1. ถ้าบน Mac ก็ใช้ Terminal ถ้าบน Windows ก็ใช้ Command Line เปิดขึ้นมา
2. จากนั้นพิมพ์คำสั่งเข้าไปยังโฟลเดอร์โปรเจ็กของเรา
3. พิมพ์คำสั่งติดตั้งคลาสที่ต้องการ ในตัวอย่างจะเป็นการติดตั้งโค้ด LINEBot SDK
   ```
   composer require linecorp/line-bot-sdk
   ```
4. เมื่อเราลองเปิดเข้าไปดูในโฟลเดอร์โปรเจ็กเรา เราจะพบกับโฟลเดอร์ vendor ภายในจะมีไฟล์สำคัญไฟล์หนึ่งชื่อ autoload.php ไฟล์นี้ที่เราจะใช้สำหรับ include ไปใช้งาน  
   ![](/assets/2017-10-11_1102.png)



