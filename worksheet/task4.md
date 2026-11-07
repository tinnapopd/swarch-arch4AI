# Task 4: Deployment topologies and trade-offs

**Name / ชื่อ:** ติณณภพ ดวงทา

**Student ID / รหัสนิสิต:** 6870107021

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง ใน fork ของตัวเอง*

**Scenario (A, B or C) / Scenario ที่เลือก:** B. Canteen kiosk

Choose **one** scenario below. If the last digit of your student ID is odd, take A or B; if it is even, take B or C. Use your measurements from Tutorial 6 as evidence: cite run names from `results/runs.csv`. Answer in Thai or English.

เลือก scenario **หนึ่ง** ข้อด้านล่าง ถ้าเลขตัวสุดท้ายของรหัสนิสิตเป็นเลขคี่ ให้เลือก A หรือ B ถ้าเป็นเลขคู่ ให้เลือก B หรือ C ใช้ผลการวัดจาก tutorial ที่ทำไปเป็นข้อมูลอ้างอิง โดยอ้างชื่อ run จาก `results/runs.csv` จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

Facts that hold in every scenario / ข้อเท็จจริงที่ใช้กับทุก scenario:

- The large model serves at most **20 calls per second** (4 calls at once × about 200 ms each), and every call costs money.

  *large model รับได้สูงสุด **20 calls ต่อวินาที** (พร้อมกัน 4 calls × ประมาณ 200 ms ต่อ call) และทุก call มีค่าใช้จ่าย*
- The small model is fast and cheap enough to run anywhere, including inside the review service or on a kiosk, but it is much less accurate on its own.

  *small model เร็วและถูกพอที่จะรันที่ไหนก็ได้ รวมถึงภายใน review service หรือบน kiosk แต่ถ้าใช้ตัวเดียวจะแม่นน้อยกว่ามาก*

## Scenarios

**A. Owner alerts.** When a customer posts a negative review, the restaurant owner gets a push notification within **2 seconds**. At the lunch peak, CampusEats receives **40 reviews per second**. A missed alert is acceptable, but a wrong alert (for example, "your customer is unhappy" when they are happy) makes owners stop trusting alerts and, in the end, ignore them.

*เมื่อลูกค้าโพสต์รีวิวเชิงลบ เจ้าของร้านจะต้องได้รับ push notification ภายใน **2 วินาที** เวลาช่วงพีคตอนเที่ยง CampusEats ได้รับรีวิวจำนวน **40 รายการต่อวินาที** การพลาด alert ไปบ้างนั้นยอมรับได้ แต่ alert ที่ผิด (เช่น "ลูกค้าไม่พอใจ" ทั้งที่จริงๆ แล้วพอใจ) จะทำให้เจ้าของร้านไม่เชื่อถือและเลิกสนใจ alert ในที่สุด*

**B. Canteen kiosk.** A tablet in each canteen lets students leave a review on the spot. However, the canteen Wi-Fi is poor: the tablets lose their connection several times a day, for minutes at a time. The system must accept every review, even while a tablet is offline. Restaurant owners need the labels **by the end of the day**.

*มี tablet ในแต่ละโรงอาหารให้นิสิตรีวิวได้ทันที อย่างไรก็ดี การเชื่อมต่อ Wi-Fi ในโรงอาหารไม่ดี ทำให้ tablet หลุดการเชื่อมต่อวันละหลายครั้ง ครั้งละหลายนาที ระบบจะต้องรับรีวิวได้ทุกรายการแม้ขณะที่กำลัง offline อยู่ และเจ้าของร้านต้องการ label **ภายในสิ้นวัน***

**C. Weekly report.** Restaurant owners want a weekly sentiment report. Nothing in producing this report has to be real-time. **Accuracy matters most** and **cost must be minimal**.

*เจ้าของร้านต้องการรายงาน sentiment รายสัปดาห์ ไม่มีอะไรที่จะต้องทำแบบ real-time ในการออกรายงานนี้ ความแม่นยำ **(accuracy) สำคัญที่สุด** และ **ค่าใช้จ่ายต้องน้อยที่สุด***

## Topologies and patterns to choose from / Topology และ pattern ที่เลือกใช้ได้

| | Topology | Where the model runs / โมเดลรันที่ไหน |
|---|---|---|
| T1 | Model as a service, synchronous | Inference service, called for each review (today's baseline)<br>*inference service ถูกเรียกทุกรีวิว (baseline ของวันนี้)* |
| T2 | Model as a library | Inside the review service, or on the device<br>*ภายใน review service หรือบนอุปกรณ์* |
| T3 | Hybrid | Small model locally, large model as a remote service<br>*small model รันในเครื่อง large model เป็น remote service* |
| T4 | Asynchronous / batch | Reviews are queued (broker, W10) or stored, and scored later in bulk<br>*รีวิวถูกเข้าคิว (broker, W10) หรือเก็บไว้ แล้วติด label ทีหลังเป็นชุด* |

Combine one topology with any of the patterns from Tutorial 6: **fallback**, **cascade** and **cache**.

*เลือก topology หนึ่งแบบ แล้วผสมกับ pattern จาก Tutorial 6 ได้ตามต้องการ: **fallback**, **cascade** และ **cache***

## 1. Quality requirements

The three most important quality requirements for your scenario, each with a number where possible.

*quality requirements ที่สำคัญที่สุด 3 ข้อของ scenario ที่เลือก ระบุตัวเลขเมื่อทำได้*

| # | Quality | Requirement |
|---|---|---|
| 1 | Availability | tablet รับรีวิวได้ทุกรายการแม้ offline นานถึง 30 นาที และไม่มีรีวิวหาย |
| 2 | Timeliness | ทุกรีวิวได้ label ภายในสิ้นวัน |
| 3 | Accuracy / Cost | accuracy อย่างน้อย 90% และใช้ large model ไม่เกิน 40% ของรีวิว |

## 2. Design / การออกแบบ

Your topology, the patterns you add, and a container-level diagram (C4 level 2). The C4 diagram must be written in Mermaid.

*topology ที่เลือก pattern ที่เพิ่ม และ diagram ระดับ container (C4 level 2) เขียนด้วย Mermaid*

ใช้ **T4 (Asynchronous / batch)** tablet เก็บรีวิวไว้ในเครื่องก่อน แล้วส่งขึ้น server เมื่อกลับมา online จากนั้น scoring worker ติด label ให้เป็นชุดๆ

- **Cascade 0.7:** ได้ accuracy ใกล้ large model แต่จ่ายเฉพาะรีวิวที่ยาก
- **Cache:** รีวิวที่ kiosk มักสั้นและซ้ำกัน
- **Fallback:** ถ้าติด label ไม่สำเร็จ ให้เป็น pending ไว้แล้วลองใหม่รอบถัดไป

```mermaid
C4Container
    title Canteen kiosk: container diagram
    Person(student, "Student", "Leaves a review on the canteen tablet")
    Person(owner, "Restaurant owner", "Reads label counts by the end of the day")
    System_Boundary(campuseats, "CampusEats") {
        Container(kiosk, "Kiosk app", "Tablet, SQLite outbox", "Saves reviews offline with a UUID")
        Container(gateway, "API gateway", "Python, FastAPI", "Entry point for kiosks and owners")
        Container(review, "Review service", "Python, FastAPI", "Stores reviews as pending")
        ContainerDb(reviewdb, "Review database", "PostgreSQL", "Reviews, labels, status")
        Container(inference, "Inference service", "Python, FastAPI", "Small and large model")
        Container(worker, "Scoring worker", "Python", "Cascade 0.7 + cache, max 4 large calls")
        ContainerQueue(broker, "Message broker", "RabbitMQ", "review-created events")
    }
    Rel(student, kiosk, "Writes review", "Touch screen")
    Rel(owner, gateway, "Views label counts", "HTTPS/JSON")
    Rel(kiosk, gateway, "Syncs when online", "HTTPS/JSON")
    Rel(gateway, review, "Forwards", "HTTP/JSON")
    Rel(review, reviewdb, "Saves as pending", "SQL")
    Rel(review, broker, "Publishes", "AMQP")
    Rel(worker, broker, "Consumes", "AMQP")
    Rel(worker, inference, "POST /predict", "REST")
    Rel(worker, reviewdb, "Writes labels", "SQL")
    UpdateRelStyle(kiosk, gateway, $offsetY="-25")
    UpdateRelStyle(gateway, review, $offsetY="-25")
    UpdateRelStyle(review, reviewdb, $offsetY="-25")
    UpdateRelStyle(worker, broker, $offsetY="-25")
    UpdateRelStyle(worker, inference, $offsetY="-25")
    UpdateRelStyle(review, broker, $offsetX="-45")
    UpdateRelStyle(worker, reviewdb, $offsetX="90", $offsetY="10")
    UpdateLayoutConfig($c4ShapeInRow="4", $c4BoundaryInRow="1")
```

## 3. Evidence / หลักฐาน

Which runs support your choice? Show the numbers and, for scenario A, the capacity arithmetic: how many large-model calls per second does your design need at the peak?

*run ไหนที่สนับสนุนการตัดสินใจของนิสิต ให้นิสิตแสดงตัวเลขประกอบ และสำหรับ scenario A ให้นิสิตแสดงการคำนวณ capacity ด้วย เพื่อแสดงว่าในช่วงพีคนั้น design ของนิสิตต้องใช้ large model กี่ calls ต่อวินาที*

| Run | What it shows / แสดงอะไร |
|---|---|
| `t0-small` | small model อย่างเดียวแม่นแค่ 72.5% ไม่ถึง 90% |
| `t1-after` | รับรีวิวไว้ก่อนแล้วค่อยติด label ทีหลัง ทำให้ errors เป็น 0 |
| `t2-0.7` | cascade ได้ accuracy 93.0% โดยใช้ large model 53% |
| `t3-cache` | เพิ่ม cache แล้วใช้ large model เหลือ 38% โดย accuracy เท่าเดิม |

ถ้ามีรีวิววันละ 5,000 รายการ (เหตุการณ์สมมติ) จะใช้ large model ประมาณ 1,900 calls ที่ 20 calls ต่อวินาทีใช้เวลาไม่ถึง 2 นาที จึงทันสิ้นวันแน่นอน

## 4. ADR

Use the ADR format from W5.

*ใช้รูปแบบ ADR จาก W5*

**Title:** ใช้ batch scoring สำหรับรีวิวจาก kiosk

**Status:** Proposed

**Context:** Wi-Fi ในโรงอาหารหลุดบ่อย แต่ต้องรับรีวิวได้ทุกรายการ และเจ้าของร้านต้องการ label แค่ภายในสิ้นวัน ส่วน small model อย่างเดียวแม่นไม่พอ (72.5%)

**Decision:** tablet เก็บรีวิวไว้ในเครื่องแล้วส่งเมื่อ online review service บันทึกเป็น pending แล้วให้ worker ติด label เป็นชุดด้วย cascade 0.7 และ cache

**Consequences:**
- Positive: รีวิวไม่หายแม้ offline ได้ accuracy 93% และใช้ large model แค่ 38% (`t3-cache`)
- Negative: label ไม่ real-time และมี component เพิ่ม (broker, worker, database) ถ้า tablet เสียก่อน sync รีวิวที่ค้างอยู่จะหาย

**Alternatives considered:** at least one topology you rejected, and why.

*อย่างน้อยหนึ่ง topology ที่ไม่เลือก และเหตุผล*

> - **T1:** tablet ต้อง online ตอนส่ง จึงรับรีวิวตอน offline ไม่ได้
> - **T2:** small model บน tablet ทำงาน offline ได้ แต่แม่นแค่ 72.5%
