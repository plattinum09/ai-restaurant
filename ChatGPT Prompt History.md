# ChatGPT Prompt History — AI Restaurant Finder for Team Dinner

หมายเหตุ: ส่วนนี้เป็น Prompt History จากฝั่ง ChatGPT ใช้เสริมจาก prompt history ของ Claude Code เพื่อแสดงขั้นตอนคิด วิเคราะห์ ตรวจ และแก้ไขงานตั้งแต่ Data / Report / n8n / Final QA

---

## 1\. ตรวจโจทย์และกำหนดขอบเขตงาน

**Prompt:**  
ช่วยอ่านโจทย์ AI Food Assistant / AI Restaurant Finder แล้วสรุปว่าต้องทำอะไรบ้าง ต้องส่งไฟล์อะไร และ Final Output ต้องมีองค์ประกอบอะไร

**Purpose:**  
ใช้เพื่อทำความเข้าใจ requirement หลักของงานก่อนเริ่มทำ report

**Output ที่ได้:**  
สรุปว่า Final Output ควรมี HTML report, Google Sheets, Top 10 Ranking, Top 3 Recommendation, Scoring Model, Workflow / Evidence, Prompt History และ Short Reflection

---

## 2\. Clean Data และเตรียมข้อมูลร้านอาหาร

**Prompt:**  
ช่วย clean ข้อมูลร้านอาหารจากตารางนี้ให้พร้อมใช้สำหรับ scoring โดยเก็บเฉพาะร้านในพื้นที่ สยาม, อารีย์, ทองหล่อ, อโศก, พร้อมพงษ์ ลบร้านซ้ำ normalize price\_range จัด food\_type ให้เป็นหมวดเดียวกัน และประเมิน suitable\_for\_group สำหรับทีม 8–12 คน

**Purpose:**  
ใช้เตรียมข้อมูลร้านอาหารให้เป็นโครงสร้างเดียวกันก่อนนำไปให้คะแนน

**Output ที่ได้:**  
ได้แนวทาง clean data และคอลัมน์ที่ควรมี เช่น restaurant\_name, area, food\_type, rating, review\_count, price\_range, opening\_hours, travel\_note, suitable\_for\_group, notes, source\_url

---

## 3\. ปรับ Clean Data ให้ใช้ข้อมูลจากไฟล์ดิบเท่านั้น

**Prompt:**  
ช่วย clean ข้อมูลร้านอาหารจากไฟล์ชุดใหม่ แต่ source\_url ควรเอามาจากไฟล์ดิบเท่านั้น ไม่อนุญาตให้เอามาจากที่อื่น

**Purpose:**  
เพื่อป้องกันการแต่งข้อมูลหรือใช้แหล่งข้อมูลนอก dataset ที่กำหนด

**Output ที่ได้:**  
แนวทางการ clean โดยยึดข้อมูลจากไฟล์ดิบและ source\_url เดิมเป็นหลัก

---

## 4\. ทำ Scoring Model 100 คะแนน

**Prompt:**  
ช่วยทำ Scoring ร้านอาหารจาก Clean Data สำหรับเลือกมื้อทีม 8–12 คน ใช้ Scoring Model 100 คะแนนตามน้ำหนักที่กำหนด ห้ามเปลี่ยนน้ำหนัก และต้องอิงจากข้อมูลในตารางเท่านั้น

**Purpose:**  
ใช้สร้างคะแนนรวมของแต่ละร้านโดยใช้เกณฑ์เดียวกันทุกแถว

**Output ที่ได้:**  
ได้แนวทาง scoring 6 มิติ: 1\. Rating & Review Quality \= 25 2\. Group Suitability \= 20 3\. Price Suitability \= 15 4\. Travel Convenience \= 15 5\. Data Completeness \= 15 6\. Uniqueness / Experience \= 10

---

## 5\. ตรวจเรื่อง Source 2

**Prompt:**  
source 2 มีนะ ในคอลัม note ไงที่เขียนว่า แหล่งที่ 2

**Purpose:**  
ใช้ตรวจว่าข้อมูลแหล่งที่ 2 ถูกนับและใช้ในรายงานหรือยัง

**Output ที่ได้:**  
ยืนยันว่าแหล่งข้อมูลที่ 2 ควรดึงจากคอลัมน์ note / source ที่มีอยู่ในไฟล์ดิบ ไม่ควรสร้างเอง

---

## 6\. สรุป Top 3 Recommendation

**Prompt:**  
จากตาราง Top 10 Ranking นี้ ช่วยสรุป Top 3 Recommendation สำหรับทีม 8–12 คน ต้องตอบร้านอันดับ 1–3 เหมาะกับทีมเพราะอะไร จุดเด่น ข้อควรระวัง / trade-off และเหตุผลแบบใช้ตัดสินใจได้จริง

**Purpose:**  
ใช้สร้างเนื้อหา Top 3 Recommendation สำหรับใส่ใน HTML report

**Output ที่ได้:**  
ได้ตาราง Top 3 Recommendation พร้อม best\_for, why\_recommended, strengths, trade\_off\_risk และ final\_recommendation

---

## 7\. ตรวจความครบของ HTML Report

**Prompt:**  
ตรวจหน่อยว่า HTML report ครบตามข้อกำหนดที่ต้องมีหรือยัง

**Purpose:**  
ใช้ QA หน้าเว็บว่ามี section สำคัญครบตาม requirement หรือไม่

**Output ที่ได้:**  
เช็กว่า HTML report มี Objective, Executive Summary, Top 3, Top 10, Restaurant Comparison, Scoring Criteria, Workflow Overview, Tools Used, Data Sources, Evidence, Notes / Limitations และ Dataset Ask Box

---

## 8\. วางโครง n8n Ask Box

**Prompt:**  
หน้าเว็บ HTML ต้องมีจุดบอกเล็กไหมว่าเราต่อข้อมูลแบบนี้ และ Dataset Ask Box ต่อ n8n ควรอธิบายยังไง

**Purpose:**  
ใช้กำหนดข้อความ Evidence และ Workflow ใน HTML ให้บอกว่า Ask Box ทำงานผ่าน n8n Production Webhook

**Output ที่ได้:**  
ได้แนวทางเขียนว่า HTML Ask Dataset → n8n Webhook → Google Sheets CSV → Dataset Answer

---

## 9\. แก้ CORS และตรวจ Live Data API

**Prompt:**  
ทำไมหน้าเว็บขึ้น Embedded fallback และทำยังไงให้เป็นสีเขียว Live from n8n

**Purpose:**  
ใช้ debug การเรียก n8n Production Webhook จากหน้า HTML

**Output ที่ได้:**  
ตรวจพบปัญหา CORS / การเปิดไฟล์ผ่าน file:// และแก้ด้วยการเปิดผ่าน localhost รวมถึงเพิ่ม Access-Control-Allow-Origin ใน n8n response headers

---

## 10\. เปิด localhost เพื่อทดสอบ HTML

**Prompt:**  
เปิด Terminal แล้วเข้าโฟลเดอร์ที่มีไฟล์ ทำยังไง และเปิดหน้าเว็บ localhost ยังไง

**Purpose:**  
ใช้ทดสอบเว็บแบบ local server แทนการเปิดไฟล์ตรงจาก Desktop

**Output ที่ได้:**  
คำสั่ง: cd /Users/farennn/Desktop/Restaurant  
python3 \-m http.server 8000  
แล้วเปิด http://localhost:8000/ai\_restaurant\_finder\_team\_dinner.html

---

## 11\. ตรวจ n8n Production URL

**Prompt:**  
Production URL เปิดแล้วขึ้น JSON แบบนี้ถูกไหม

**Purpose:**  
ใช้เช็กว่า n8n Production Webhook ทำงานจริงและตอบ JSON ได้

**Output ที่ได้:**  
ยืนยันว่า Production URL ส่ง JSON สำเร็จ และ Network ใน browser ขึ้น status 200

---

## 12\. แก้ n8n Live Data API ให้ดึง 3 CSV

**Prompt:**  
เพิ่ม Top 3 Recommendation / Top 10 เดิมใน n8n ได้ไหม และ Code node ให้ return JSON รวม 3 ชุด

**Purpose:**  
ใช้แก้ n8n Live Data API ให้ส่งข้อมูลครบกว่าเดิม ไม่ใช่มีแค่ Scoring CSV

**Output ที่ได้:**  
Workflow ล่าสุด: Webhook → Fetch Scoring CSV → Fetch Top 10 CSV → Fetch Top 3 CSV → Parse CSV to Report JSON → Respond to Webhook

JSON ที่ส่งกลับ: restaurantsData, top10, top3

---

## 13\. แก้ Area Filter / Top 3 / Restaurant Comparison

**Prompt:**  
พอกดเลือกย่านเดียว Area Filter เป็น 0 และ Restaurant Comparison / Top 3 ขึ้น Need verification แก้ยังไง

**Purpose:**  
ใช้ debug หน้า HTML ตอนเลือกย่านเดียว

**Output ที่ได้:**  
สรุปว่าปัญหาเกิดจาก live data มี field ไม่ครบเท่า embedded Clean Data จึงแนะนำให้ merge live data กับ embedded Clean Data โดยใช้ restaurant\_name เป็น key

---

## 14\. ปรับ wording ไม่ให้ดูเหมือนรายงานยังไม่เสร็จ

**Prompt:**  
คำว่า Need verification, verify, ไม่มีข้อเสียเด่นจากข้อมูลในตาราง ควรเอาออกไหม

**Purpose:**  
ใช้ปรับภาษาในรายงานให้ดูเป็นข้อจำกัดของข้อมูล ไม่ใช่เหมือนงานยังไม่สมบูรณ์

**Output ที่ได้:**  
เปลี่ยนคำให้เป็นภาษารายงาน เช่น:

* ข้อมูลบางส่วนควรตรวจสอบเพิ่มเติม

* ควรยืนยันโต๊ะ 8–12 คนก่อนจอง

* ไม่พบข้อจำกัดชัดเจนจากข้อมูลที่มี

* ควรตรวจสอบเวลาเปิด–ปิด ราคา และการจองโต๊ะก่อนตัดสินใจ

---

## 15\. ปรับ Evidence section ให้ตรงกับ workflow ล่าสุด

**Prompt:**  
n8n Live Data API ใน Evidence ต้องเปลี่ยนไหม เพราะ workflow ล่าสุดเป็น 3 CSV แล้ว

**Purpose:**  
ใช้แก้ข้อความ Evidence ให้ตรงกับระบบจริงล่าสุด

**Output ที่ได้:**  
ข้อความ workflow ที่ถูกต้อง: Webhook → Fetch Scoring CSV → Fetch Top 10 CSV → Fetch Top 3 CSV → Parse CSV to Report JSON → Respond to Webhook

---

## 16\. ปรับ Workflow Overview และ Tools Used

**Prompt:**  
Workflow Overview กับ Tools Used สัมพันธ์กันหรือยัง

**Purpose:**  
ใช้ตรวจความสอดคล้องของ section Workflow Overview และ Tools Used

**Output ที่ได้:**  
ปรับ n8n ให้ระบุว่าใช้ 2 ส่วน: 1\. Dataset Ask Box รับคำถามจากหน้าเว็บผ่าน Production Webhook แล้วตอบจาก Google Sheets CSV 2\. Live Data API ดึง Scoring, Top 10 Ranking และ Top 3 Recommendation จาก Google Sheets CSV เพื่อ sync ข้อมูลเข้า HTML report

---

## 17\. ตรวจ Final HTML Report ก่อนส่ง

**Prompt:**  
ตรวจหน่อยผ่านยังกับข้อกำหนดที่ต้องมี

**Purpose:**  
ใช้ตรวจไฟล์ HTML ล่าสุดก่อนส่งงาน

**Output ที่ได้:**  
ยืนยันว่า HTML report ผ่านในส่วนหลัก ได้แก่ Header, Objective, Workflow Overview, Tools Used, Data Sources, Scoring Criteria, Top 10 Ranking, Top 3 Recommendation, Restaurant Comparison, Evidence, Notes / Limitations และ Dataset Ask Box

---

## 18\. สรุปไฟล์ที่ต้องส่ง

**Prompt:**  
ตอนนี้ต้องมีไฟล์อะไรบ้างสำหรับส่งงาน

**Purpose:**  
ใช้เช็ก checklist ส่งงานสุดท้าย

**Output ที่ได้:**  
ไฟล์ที่ควรมี:

* ai\_restaurant\_finder\_team\_dinner.html

* prompt\_history\_restaurant.md

* short\_reflection\_restaurant.md

* n8n\_askbox\_workflow\_success.png

* n8n\_live\_data\_api\_3csv\_workflow\_success.png

* Apify proof screenshot / dataset export

* Google Sheets link