# ความปลอดภัย

## ผู้ใช้ที่รัน
แนะนำให้ตั้งผู้ใช้ที่รันให้มีสิทธิ์ต่ำ เช่นเดียวกับผู้ใช้ที่รัน nginx ผู้ใช้ที่รันจะถูกกำหนดใน `config/server.php` ใน`user`และ`group` ต่าง ๆ คล้ายกัน เช่นกันกับการกำหนดผู้ใช้สำหรับกระบวนการที่กำหนดเองผ่าน `config/process.php` จำเป็นต้องระวังว่า กระบวนการดูแลจะไม่ควรกำหนดผู้ใช้ที่รันเนื่องจากต้องการสิทธิ์สูงเพื่อทำงานได้อย่างปกติ

## ข้อกำหนดของคอนโทรลเลอร์
เฉพาะไดเร็กทอรี `controller` หรือไดเร็กทอรีย่อย ๆ เท่านั้นที่จะได้รับการตั้งไฟล์คอนโทรลเลอร์ ห้ามใส่ไฟล์คลาสอื่น ๆ มิฉะนั้นเมื่อไม่เปิด [คำต่อท้ายคอนโทรลเลอร์](https://www.workerman.net/doc/webman/controller.html#%E6%8E%A7%E5%88%B6%E5%99%A8%E5%90%8E%E7%BC%80) ไฟล์คลาสอาจถูกเข้าถึงผ่าน URL ที่ผิดกฎเกณฑ์ แล้วทำให้เกิดผลลัพธ์ที่ไม่สามารถทำนายได้
ตัวอย่างเช่น `app/controller/model/User.php` แท้จริงเป็นคลาสโมเดล แต่ถูกวางไว้อยู่ในไดเร็กทอรี `controller` โดยไม่ได้เปิดใช้งาน [คำต่อท้ายคอนโทรลเลอร์](https://www.workerman.net/doc/webman/controller.html#%E6%8E%A7%E5%88%B6%E5%99%A8%E5%90%8E%E7%BC%80) จะทำให้ผู้ใช้สามารถเข้าถึงเมทอดใด ๆ ใน `User.php` ผ่านทางเช่นเช่น `/model/user/xxx` โดยไม่ใช่คำสั่งใด ๆ ที่ถูกเรียกใช้
เพื่อป้องกันอย่างสมบูรณ์ ขอแนะนำให้ใช้ [คำต่อท้ายคอนโทรลเลอร์](https://www.workerman.net/doc/webman/controller.html#%E6%8E%A7%E5%88%B6%E5%99%A8%E5%90%8E%E7%BC%80) ให้เป็นการชี้ว่าไฟล์ต่าง ๆ จะเป็นคอนโทรลเลอร์ไฟล์

## XSS ฟิลเตอร์
ด้วยความสามารถในการใช้งานทั่วไป webman ไม่ได้ทำการแปลง XSS สำหรับการร้องขอ
webman ขอแนะนำให้ทำการแปลง XSS ขณะที่ทำการแสดงผล และไม่ใช่การแปลงก่อนเข้าสู่ฐานข้อมูล
และ twig、blade、think-tmplate หรือโค้ดตามรูปแบบต่าง ๆ จะทำการแปลง XSS โดยอัตโนมัติ สามารถใช้อย่างสะดวก

> **เคล็ดลับ**
> หากคุณทำการแปลง XSS ก่อนเข้าสู่ฐานข้อมูล อาจจะทำให้เกิดปัญหาที่ไม่สามารถทำงานร่วมกับปลั๊กอินแอปพลิเคชั่นบางอย่าง

## ป้องกันการฉีดโค้ด SQL
เพื่อป้องกันการฉีดโค้ด SQL โปรดใช้ ORM ในการทำงานเท่าที่จะทำได้ เช่น [illuminate/database](https://www.workerman.net/doc/webman/db/tutorial.html)、[think-orm](https://www.workerman.net/doc/webman/db/thinkorm.html) โปรดเลือกใช้ไม่ให้ทำงานในการรวบรวมคำสั่ง SQL ไปเอง

## การแทรกกระจาย nginx
เมื่อแอปพลิเคชันของคุณจำเป็นต้องเปิดเผยแก่ผู้ใช้ออกไปในอินเทอร์เน็ต ขอแนะนำให้เพิ่ม nginx พาร์เซอร์ก่อน webman เช่นนี้จะช่วยกรองคำสั่ง HTTP ที่ผิดกฎหมายบางอย่าง และเพิ่มความปลอดภัย โปรดดูข้อมูลเพิ่มเติมที่ [nginx พาร์เซอร์](nginx-proxy.md)