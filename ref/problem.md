# ปัญหาที่พบ

```
failed to create symbolic link '/app/.heroku/php/php': File exists
```

he issue was occuring because i had added buildpack of php multiple times .Once i removed all of them and added only once it worked like a charm.

ref : [https://github.com/heroku/heroku-buildpack-php/issues/181](https://github.com/heroku/heroku-buildpack-php/issues/181)

```
-----> App not compatible with buildpack: https://codon-buildpacks.s3.amazonaws.com/buildpacks/heroku/php.tgz
       More info: https://devcenter.heroku.com/articles/buildpacks#detection-failure
 !     Push failed
```

ปัญหาเกิดจาก heroku มันไม่ทราบว่าจะติดตั้งแพ็กเกจอะไรลงไปในเซิฟเวอร์ที่กำลังจะสร้างขึ้นมาบ้าง

การแก้ปัญหา

สร้างไฟล์

index.php

composer.json

แล้วดีพลอยใหม่



```
อย่าสร้างชื่อคอลัมน์ตัวอักษรตัวใหญ่ หรือ ตัวเล็กปนตัวใหญ่ ใน postgresql เด็ดขาด เพราะอีนี่เวลาส่ง SQL คิวรี่ให้มัน มันจะแปลงเป็นตัวเล็กทั้งหมด 
```





