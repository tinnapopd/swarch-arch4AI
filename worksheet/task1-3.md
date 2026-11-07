# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** ติณณภพ ดวงทา

**Student ID / รหัสนิสิต:** 6870107021

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 509 | 613 | 93.5% | 100% |
| t0-small | 46 | 107 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> เพิ่ม accuracy จาก 72.5% เป็น 93.5% แต่แลกมาด้วย latency ที่ช้ากว่าประมาณ 11 เท่า

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 30.3 | 248 | 420 | 21 | 0 | 92.7% |
| t1-after | 10.0 | 414 | 1011 | 0 | 21 | 92.7% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> p50 และ p95 คำนวณจาก request ที่สำเร็จเท่านั้น จึงตัด requests ที่ค้างจน client เลิกรอออกไป column ที่บอกความจริงคือ `errors` ประกอบกับ run time ที่เพิ่มจาก 10.2 เป็น 30.3 วินาที

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> request ที่ค้างจะ hold connection ใน gateway และ review service ไว้ตลอดเวลา พอค้างสะสมหลายตัว request อื่นที่ไม่ได้ค้างก็ต้องรอคิวตามไปด้วย ทั้งระบบเลยช้าลง (run time เพิ่มจาก 10.2 เป็น 30.3 วินาที)

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> เก็บรีวิวไว้ก่อน แล้วให้ batch job มาติด label ทีหลังเมื่อ large model กลับมาปกติ ซึ่งก็คือ topology T4 (Asynchronous / batch)

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> ถ้า 0.1 วินาที จะสั้นกว่าเวลาปกติของ large model (ประมาณ 200 ms) เกือบทุกรีวิวเลยกลายเป็น pending ถ้า 10 วินาที จะผิด QR2 และ request ที่ค้างจะ hold resource ไว้นานจนเกิดปัญหาแบบ `t1-before` อีก

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 11 | 527 | 91.5% | 38% |
| t2-0.7 | 0.7 | 394 | 592 | 93.0% | 53% |
| t2-0.8 | 0.8 | 453 | 585 | 93.5% | 64% |
| t2-0.9 | 0.9 | 485 | 617 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> เลือก 0.7 เพราะ accuracy 93.0% ใกล้กับ large model แต่ใช้ large model แค่ 53% ส่วน 0.6 ใช้ทรัพยากรอื่นๆน้อยกว่าก็จริง แต่ accuracy 91.5% เฉียด 90% เกินไป สิ่งที่ยอมเสียคือ cost และ latency ที่สูงกว่า 0.6

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> รีวิวง่ายๆที่ small model ตอบได้ ช่วยดึง p50 ลงมา แต่ 5% ที่ช้าที่สุดยังเป็นรีวิวที่ต้องส่งไป large model อยู่ดี p95 เลยแทบไม่เปลี่ยน

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> รีวิวที่ประชดประชัน เช่น "Great service, only waited an hour!" small model เห็นคำว่า great แล้วตอบ positive อย่างมั่นใจ เลยไม่ส่งต่อให้ large model ผลคือเจ้าของร้านไม่รู้ว่าลูกค้าไม่พอใจ

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 12 | 549 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> รีวิวสั้นๆ ที่คนพิมพ์ซ้ำกันบ่อย เช่น "Delicious!" หรือ "Too salty" ได้ประโยชน์ ส่วนรีวิวยาวที่ไม่ซ้ำใคร หรือรีวิวที่ความหมายเดียวกันแต่ใช้คำต่างกัน จะไม่มีทาง hit cache

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> ต้องล้าง cache ตอน deploy ไม่งั้นรีวิวซ้ำจะยังได้คำตอบจากโมเดลเก่า วิธีที่ง่ายคือใส่ model version ไว้ใน key ของ cache หรือใส่ TTL

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> ตั้ง timeout 1 วินาทีที่ review service ไม่ได้แปลว่าผู้ใช้จะรอไม่เกิน 1 วินาที ใน `t1-after` p95 ยังเป็น 1011 ms เพราะยังมีเวลาของ gateway บวกเพิ่มเข้ามา
