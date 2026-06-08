# Prompt History — AI Restaurant Finder for Team Dinner

## Project Summary

โปรเจกต์นี้เป็น **HTML decision dashboard ไฟล์เดียว** (`ai_restaurant_finder_team_dinner.html`) สำหรับช่วยทีมเลือก **ร้านอาหารมื้อทีม 8–12 คน** ใน 5 ย่าน: **สยาม, อารีย์, ทองหล่อ, อโศก, พร้อมพงษ์**

- **แหล่งข้อมูล:** Apify / Google Maps (หลัก) + แหล่งข้อมูลที่ 2 (Wongnai / Tripadvisor)
- **ฐานข้อมูล:** Google Sheets หลายชุด — Raw Data, Clean Data, Scoring, Top 10 Ranking, Top 3 Recommendation
- **การวิเคราะห์:** Scoring Model 100 คะแนน (6 มิติ) → Top 10 Ranking → Top 3 Recommendation → Restaurant Comparison
- **n8n automation 2 ส่วน:** (1) Dataset Ask Box (ถาม–ตอบจาก dataset) (2) Live Data API (sync ข้อมูลจาก Google Sheets CSV เข้าหน้าเว็บ)
- **บทบาท AI:** ช่วยวางโครง report, clean ข้อมูล, ทำ scoring, สร้าง/ปรับ HTML, debug, ปรับ wording และตรวจ QA — ไม่ได้ทำแทนทั้งหมด คนเป็นผู้ตัดสินใจและตรวจสอบ

> หมายเหตุความถูกต้อง: **Apify เป็นตัว scrape ข้อมูลจาก Google Maps** ส่วน **n8n ใช้สำหรับ Ask Box และ Live Data API เป็นหลัก** (ไม่ใช่ตัว scrape)

---

## Prompt Log

### 1. Requirement Understanding & Planning

- **Prompt / Instruction:** "อ่านโจทย์ข้อสอบ AI Workflow แล้วสรุปว่า HTML report ต้องมี section อะไรบ้าง และวางโครงหน้าให้เป็น decision dashboard สำหรับทีม 8–12 คน ใน 5 ย่านที่กำหนด ห้ามเลือกย่านอื่น ห้ามเพิ่มร้านเอง ห้ามแก้คะแนนเอง"
- **Purpose:** เข้าใจขอบเขตงานและกำหนดโครง report ก่อนลงมือ
- **What changed / Output:** ได้ลิสต์ section ที่ต้องมี (Header, Objective, Executive Summary, Scoring Criteria, Top 10, Top 3, Comparison, Evidence, Notes ฯลฯ) + ข้อกำหนด: HTML/CSS/JS ล้วน, responsive, ไม่ทำ PDF/Slide
- **Notes:** ตอกย้ำกติกา "ใช้ข้อมูลจริงจากตารางเท่านั้น" ตั้งแต่ต้น

### 2. Data Collection & Source Validation

- **Prompt / Instruction:** "ใช้ข้อมูลร้านที่ scrape จาก Apify / Google Maps และยืนยันด้วยแหล่งข้อมูลที่ 2 (Wongnai / Tripadvisor) — ทุกร้านต้องมี source_url (Google Maps) + second_source_url อย่างน้อย 2 แหล่ง ห้ามแต่งข้อมูลนอก dataset"
- **Purpose:** ให้ทุกร้านตรวจย้อนกลับได้และมีหลักฐาน 2 แหล่ง
- **What changed / Output:** ยืนยัน source_url / second_source_url ครบทุกร้าน; ฟิลด์ที่ยืนยันไม่ได้ทำเครื่องหมายให้ตรวจเพิ่ม
- **Notes / Issue:** บางร้านเวลาเปิด-ปิด/ราคาไม่ชัดจาก Google Maps → ต้องดึงจากแหล่งที่ 2

### 3. Clean Data Preparation

- **Prompt / Instruction:** "clean ข้อมูลร้านให้เหลือเฉพาะ 5 ย่านที่กำหนด, ลบร้านซ้ำ, normalize area / food_type / price_range, ประเมิน suitable_for_group (รองรับโต๊ะ 8–12 คน), เติม notes — ใช้แหล่งอ้างอิงที่แนบเท่านั้น ฟิลด์ที่ไม่ชัดให้ทำเครื่องหมายว่าควรตรวจสอบเพิ่มเติม"
- **Purpose:** ได้ Clean Data ที่พร้อมนำไปให้คะแนน
- **What changed / Output:** Clean Data **25 ร้าน (ย่านละ 5)** ไม่มีร้านซ้ำ พร้อม address, opening_hours, travel_note, group_fit, notes; บันทึก Cleaning Log
- **Notes:** ใช้ Clean Data เป็นฐานหลักของทั้งงาน (ภายหลังใช้เป็น embedded fallback ในหน้าเว็บ)

### 4. Scoring Model

- **Prompt / Instruction:** "ให้คะแนนร้านทั้ง 25 ด้วย Scoring Model 100 คะแนน น้ำหนักคงที่: Rating & Review Quality 25, Group Suitability 20, Price Suitability 15, Travel Convenience 15, Data Completeness 15, Uniqueness/Experience 10 — อิงข้อมูลในตารางเท่านั้น ห้ามแต่งคะแนน ถ้าฟิลด์ไม่ชัดให้หักคะแนนในหมวดที่เกี่ยว"
- **Purpose:** ให้คะแนนเทียบกันได้ด้วยเกณฑ์เดียวทุกร้าน
- **What changed / Output:** ได้คะแนนย่อย 6 มิติ + `total_score` ทุกร้าน (สูงสุด 95.1 — Indulge Restaurant & Cocktail Bar) → sheet Scoring
- **Notes:** ทำ score breakdown ต่อร้านเพื่อให้ตรวจย้อนกลับได้

### 5. Top 10 Ranking

- **Prompt / Instruction:** "สร้าง Top 10 เรียงตาม total_score มากไปน้อย คอลัมน์: rank, restaurant_name, area, food_type, rating, review_count, price_range, group_fit, total_score, reason, evidence — เพิ่ม score breakdown แบบ expand ต่อแถว และ evidence links (Google Maps / Review)"
- **Purpose:** จัดอันดับและให้หลักฐานต่อร้าน
- **What changed / Output:** ตาราง Top 10 + breakdown 6 มิติ + ปุ่ม Maps / Review
- **Notes / Issue:** เจอปัญหา **คอลัมน์เหตุผลขึ้น "Need verification"** กับข้อมูล live → แก้ด้วย fallback ไล่จาก `reason → reasoning → why_recommended → pros → final_note` ก่อนค่อยใช้ข้อความกลาง

### 6. Top 3 Recommendation

- **Prompt / Instruction:** "เลือก Top 3 จากคะแนนสูงสุด เขียน best_for, why_recommended, strengths, trade_off_risk, final_recommendation — ต้องมีทั้งข้อดีและข้อควรระวัง ไม่ชมอย่างเดียว ถ้าการรองรับทีมยังไม่ชัด ให้ระบุว่าควรยืนยันโต๊ะ 8–12 คนก่อน"
- **Purpose:** สรุปตัวเลือกที่แนะนำแบบตัดสินใจได้จริง
- **What changed / Output:** การ์ด Top 3 (gold/silver/bronze) + highlight + จุดเด่น + Trade-off + คำแนะนำท้ายการ์ด
- **Notes / Issue:** **เลือกย่านเดียวแล้วการ์ดขึ้น "Need verification"** เพราะข้อมูลย่านมาจาก Scoring ไม่มี best_for/strengths/trade_off → แก้ด้วย helper fallback (`getCardHighlight / getCardPros / getCardTradeOff / getCardFinalRecommendation`) ที่สร้างข้อความจาก field ที่มีจริง (group_fit, price, area, rating, review, total_score, reasoning, pros, cons, final_note)

### 7. HTML Report Development

- **Prompt / Instruction:** "สร้าง HTML report หน้าเดียวจาก restaurantsData (JSON ฝังในไฟล์) ให้มี section: Header, Objective, Executive Summary, Area Filter, Top 3, Top 10, Restaurant Dataset, Restaurant Comparison, Dataset Ask Box, Workflow Overview, Tools Used, Evidence, Notes — responsive, HTML/CSS/JS ล้วน, sort Top 10 ตาม total_score, render Top 3 จากคะแนนสูงสุด"
- **Purpose:** ได้ report เปิดในเบราว์เซอร์ได้จริง
- **What changed / Output:** ไฟล์เดียวพร้อม Area Filter, Restaurant Dataset (กรอง 25 ร้าน + search), Comparison; ภายหลังปรับธีมเป็น food finder (โทนอุ่น + ไอคอน), เพิ่ม sticky nav, Decision Summary, ราคาเป็นบาท (ไม่ใช่ `$$`), เปลี่ยนปุ่ม Source 2 → Review
- **Notes:** ปรับข้อความ "Need verification" ที่ดูเหมือนรายงานไม่เสร็จ → เป็นภาษารายงาน เช่น **"ข้อมูลบางส่วนควรตรวจสอบเพิ่มเติม"**, **"ควรยืนยันกับร้านก่อนจอง"**

### 8. n8n Dataset Ask Box Automation

- **Prompt / Instruction:** "ทำกล่อง Dataset Ask Box ในหน้าเว็บให้ส่งคำถามไป n8n Production Webhook (GET `?question=...`) แล้ว n8n ตอบจาก Google Sheets CSV เท่านั้น ไม่ใช้ความรู้ทั่วไป — แสดง answer + matched_restaurants; เพิ่ม Evidence + proof"
- **Purpose:** ถาม–ตอบจาก dataset จริงผ่าน automation
- **What changed / Output:** Ask Box เชื่อม Production Webhook (`/webhook/restaurant-ask-box`); workflow: **Webhook Ask Box → Fetch Top 10 CSV → Fetch Top 3 CSV → Code in JavaScript → Respond to Webhook**; Evidence proof: `n8n_askbox_workflow_success.png`
- **Notes:** ย้ำในหน้าเว็บว่า "ตอบจาก dataset เท่านั้น ไม่ใช้ความรู้ทั่วไป"

### 9. n8n Live Data API

- **Prompt / Instruction:** "สร้าง n8n Live Data API ให้หน้าเว็บโหลด restaurantsData ล่าสุดจาก Google Sheets — workflow: Webhook → Fetch Scoring CSV → Fetch Top 10 CSV → Fetch Top 3 CSV → Parse CSV to Report JSON → Respond to Webhook; response ต้องมี status:success, restaurantsData, top10, top3 พร้อม CORS headers"
- **Purpose:** sync ข้อมูลสด ๆ จาก Google Sheets เข้า HTML report
- **What changed / Output:** workflow 3 CSV + Code node (parse CSV → JSON, normalize field), Respond to Webhook + CORS (`Access-Control-Allow-Origin: *` ฯลฯ)
- **Notes / Issue (debug จริง):**
  - แรก ๆ หน้าเว็บขึ้น "Embedded fallback" เพราะ **fetch ถูกบล็อก** → แก้ CORS headers + **เปิดเว็บผ่าน `http://localhost` แทน `file://`** → Network ขึ้น **status 200**
  - ได้ 200 แต่ยังไม่ swap เพราะ **`restaurantsData` ว่าง** (node Fetch Scoring CSV ดึงไม่ได้/Scoring CSV มี metadata นำหน้า header) → **แก้ Parse CSV ให้หา header row จริง** และตั้ง CSV published URL ให้ถูก
  - response เคยถูกห่อเป็น array `[ {…} ]` → ฝั่ง HTML เพิ่ม unwrap `Array.isArray(raw) ? raw[0] : raw`
  - สุดท้าย **Production URL Verified**: response มี `status:"success"`, `count:25`, `restaurantsData/top10/top3` ครบ

### 10. Live Data + Embedded Dataset Merge

- **Prompt / Instruction:** "ห้ามแทนที่ embedded Clean Data ด้วย live data ทั้งหมด — ให้ merge ตาม restaurant_name: live เป็นหลักสำหรับ score/ranking/breakdown/top10/top3/URLs; embedded เป็น fallback สำหรับ address, opening_hours, travel_note, notes, reason, pros, cons, final_note, group_fit (และ price/area/food_type ถ้า live ว่าง)"
- **Purpose:** ได้คะแนน/อันดับสดจาก n8n แต่รายละเอียดจาก Clean Data ไม่หาย
- **What changed / Output:** เพิ่ม `EMBEDDED_DATA` (Clean Data เดิม) + `mergeLiveWithEmbedded()` — `nameKey` (trim/lowercase/ยุบช่องว่าง), `{ ...embedded, ...live }` แล้วถ้า live field ว่าง/`Need verification`/`ควรตรวจสอบเพิ่มเติม` → ใช้ embedded; `restaurantsData = mergedRestaurantsData`
- **Notes:** ทุก section (Area Filter, Restaurant Dataset, Top 3 รายย่าน, Comparison รายย่าน) ใช้ `mergedRestaurantsData` เพราะอ่านจาก `restaurantsData` ตัวเดียว; ทดสอบแล้ว live total_score/urls ชนะ ส่วน address/opening_hours/reason จาก embedded ไม่หาย

### 11. Area Filter & Restaurant Comparison Fixes

- **Prompt / Instruction:** "แก้ Area Filter ที่จำนวนย่านเป็น 0 หลังโหลด live (เพราะ area เป็นอังกฤษ เช่น Phrom Phong / Sukhumvit) ด้วย `normalizeArea()` แม็พเป็น key ไทยกลาง; แก้ Restaurant Comparison ตอนเลือกย่านเดียวไม่ให้ขึ้น Need verification เต็มตาราง ด้วย helper fallback"
- **Purpose:** ให้ filter/comparison ทำงานทั้ง embedded (ไทย) และ live (อังกฤษ)
- **What changed / Output:**
  - `normalizeArea()` แม็พ siam/asoke/thonglor/ari/phrom phong/sukhumvit (+ไทย) → สยาม/อโศก/ทองหล่อ/อารีย์/พร้อมพงษ์ ใช้ทุกจุด filter/count/display
  - helper `getBestFor / getStrengths / getTradeOff / getFinalRecommendation` (ใช้ค่าจริงก่อน ไม่มีค่อยสร้างจาก field ที่มี) + แก้นับจำนวนหัวข้อ "(อารีย์ · 5 restaurants)" ให้ใช้ normalizeArea
- **Notes / Issue:** ปุ่มย่านกลับมาเป็น 5/5; Comparison รายย่าน Need verification = 0

### 12. Evidence / Workflow Overview / Tools Used Updates

- **Prompt / Instruction:** "ปรับ Workflow Overview, Tools Used, Evidence ให้ตรงระบบล่าสุด — ระบุ n8n 2 ส่วน (Ask Box + Live Data API ดึง Scoring/Top10/Top3), ปรับ Evidence n8n Live Data API เป็น workflow 3 CSV, ใส่ proof filenames"
- **Purpose:** ให้หลักฐานและคำอธิบายตรงกับสิ่งที่ทำจริง ไม่กล่าวเกินจริง
- **What changed / Output:**
  - Workflow Overview step 7 (n8n Live Data API): "โหลด restaurantsData, top10, top3 จาก Google Sheets CSV"
  - Tools Used (n8n): "live automation 2 ส่วน — Ask Box + Live Data API"
  - Evidence n8n Live Data API: workflow **Webhook → Fetch Scoring CSV → Fetch Top 10 CSV → Fetch Top 3 CSV → Parse CSV to Report JSON → Respond to Webhook**
  - proof files: `n8n_askbox_workflow_success.png`, `n8n_live_data_api_3csv_workflow_success.png`, `n8n_live_data_api_json_success.png`
- **Notes:** ไม่ลบ Evidence section; ไม่อ้างว่า n8n เป็นตัว scrape Apify

### 13. Notes / Limitations

- **Prompt / Instruction:** "ปรับ Notes / Limitations ให้ตรงระบบล่าสุด"
- **Purpose:** สื่อข้อจำกัด/วิธีใช้ข้อมูลอย่างตรงไปตรงมา
- **What changed / Output:** bullet สุดท้าย:
  - ข้อมูลบางส่วน เช่น ราคา เวลาเปิด–ปิด การเดินทาง และการรองรับโต๊ะ 8–12 คน ควรยืนยันกับร้านก่อนจอง
  - Final decision ควรตรวจสอบเวลาเปิด–ปิด ราคา และการจองโต๊ะกับร้านอีกครั้ง
  - คะแนนคำนวณด้วย Scoring Model 100 คะแนน จาก Clean Data ด้วยเกณฑ์เดียวกันทุกร้าน
  - n8n Live Data API ดึงข้อมูลจาก Google Sheets CSV 3 ชุด (Scoring, Top 10 Ranking, Top 3 Recommendation)
  - HTML ใช้ live data จาก n8n เป็นหลัก + embedded Clean Data เป็น fallback
  - Dataset Ask Box ตอบจาก dataset เท่านั้น ไม่ใช้ความรู้ทั่วไป

### 14. Final QA

- **Prompt / Instruction:** "ตรวจว่า HTML report ครบตาม requirement และระบบทำงานจริง"
- **Purpose:** ยืนยันก่อนส่งงาน
- **What changed / Output (checklist):**
  - [x] section ครบ: Header, Objective, Executive Summary, Area Filter, Top 3, Top 10, Restaurant Dataset, Comparison, Ask Dataset, Scoring Criteria, Workflow, Tools, Data Sources, Evidence, Notes
  - [x] Top 3 / Top 10 เรียงตาม total_score ถูก (ไม่เปลี่ยนคะแนน/ลำดับ)
  - [x] Area Filter ไม่เป็น 0 (normalizeArea)
  - [x] Restaurant Comparison / Top 3 รายย่าน ไม่ขึ้น Need verification เต็มตาราง (helper fallback)
  - [x] Network `restaurant-report-data` = **status 200**
  - [x] Production URL JSON มี `status:"success"`, `restaurantsData`, `top10`, `top3`, `count:25`
  - [x] live + embedded merge: address/opening_hours/travel_note/reason ไม่หาย
  - [x] ไฟล์หลักฐาน + `prompt_history_restaurant.md` พร้อมส่ง
- **Notes:** เปิดผ่าน `http://localhost` เพื่อทดสอบ live; ถ้าโหลด n8n ไม่ได้ ยัง fallback เป็น embedded 25 ร้านเต็ม

---

## สรุป n8n (2 ส่วน)

| # | ส่วน | Workflow | Proof |
|---|---|---|---|
| 1 | **Dataset Ask Box** | Webhook Ask Box → Fetch Top 10 CSV → Fetch Top 3 CSV → Code in JavaScript → Respond to Webhook | `n8n_askbox_workflow_success.png` |
| 2 | **Live Data API** | Webhook → Fetch Scoring CSV → Fetch Top 10 CSV → Fetch Top 3 CSV → Parse CSV to Report JSON → Respond to Webhook | `n8n_live_data_api_3csv_workflow_success.png`, `n8n_live_data_api_json_success.png` |

**Data sources:** Google Maps (via Apify) + Wongnai / Tripadvisor (แหล่งที่ 2)
**Database:** Google Sheets — Raw Data · Clean Data · Scoring · Top 10 Ranking · Top 3 Recommendation
**Output:** `ai_restaurant_finder_team_dinner.html` (HTML/CSS/JS หน้าเดียว, responsive, live + embedded fallback)
**บทบาท AI:** ช่วยวางโครง, clean, scoring, สร้าง/ปรับ HTML, debug n8n/CORS/merge, ปรับ wording และตรวจ QA — การตัดสินใจและการตรวจสอบขั้นสุดท้ายเป็นของผู้ทำงาน
