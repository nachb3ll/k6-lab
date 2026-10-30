# K6 lab results

## Part 0
- Step 0.1, http_req_duration: 
Ans 282.24 ms
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 
Ans ยิง 5 VUs นาน 20s ทำได้ 80 requests, อัตราเฉลี่ย 3.79 req/s, ค่า p(95) = 273.94 สาเหตุที่ Request rate ต่ำกว่า 5 req/s เพราะแต่ละ Iteration ประกอบด้วย HTTP Round-trip (265 ms) + Think time sleep(1) (1000 ms) รวมเป็น 1.26–1.31 วินาทีต่อรอบ ส่งผลให้ 1 VU ทำได้สูงสุดเพียง 0.76–0.79 req/s เมื่อรันพร้อมกัน 5 VUs จึงได้ Throughput รวมที่ประมาณ 3.8 req/s ซึ่งไม่มีทางเกิน 5 req/s
- Step 0.6, exit code of the normal run and of the run with P95=300: 
Ans Exit Code ในรอบปกติคือ 0 (ผ่านเกณฑ์ทั้งหมด) และเมื่อกำหนด -e P95=300 ได้ Exit Code เป็น 99 (เนื่องจาก p(95) วัดได้ 389.52 ms ซึ่งเกินขีดจำกัด 300 ms)
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: 
Ans ผ่านกฎทั้งหมด โดย p(95)<1000 วัดได้ 389.18 ms, p(99)<1500 วัดได้ 393.09 ms เมื่อเปลี่ยนตัวเลขเพื่อทดสอบ Fail case มี exit code คือ 99
- Step 0.8, what was wrong, and how the checks showed it: 
Ans  สาเหตุที่ Check Fail เพราะ ในสคริปต์เริ่มต้นส่งคำว่า 'pepperoni' และ 'knife' เป็นตัวพิมพ์เล็ก แต่จากทดสอบ API ด้วย explore.js พบว่าระบบ QuickPizza เป็น Case-sensitive มี expected output เป็น 'Pepperoni' และ 'Knife' เป็นตัวพิมพ์ใหญ่ การทดสอบด้วย Load ตรวจพบได้ดีกว่า Manual เพราะ QuickPizza สุ่มส่วนผสมและอุปกรณ์แบบ Random หากกด Manual Test ครั้งเดียวอาจสุ่มได้พิซซ่าที่ไม่มี Pepperoni และใช้อุปกรณ์อื่น ทำให้เข้าใจผิดว่าระบบทำงานถูกต้อง แต่เมื่อยิงทดสอบแบบ Load 75 รอบ การสุ่มจึงเจอ Pepperoni 7 ครั้งและ Knife 21 ครั้ง ทำให้ Check ล้มเหลวทันที แต่เมื่อแก้เป็นพิมพ์ใหญ่ Check ผ่าน 100%

## Part A results
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.53 | 429.85 ms | 799.18 ms | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.27 | 393.95 ms | 474.04 ms | 0.00% | 100.00% |
| A3 | Stress | 5 | 2.10 | 394.79 ms | 612.52 ms | 0.00% | 100.00% |
| A4 | Spike | 5 | 1.00 | 389.58 ms | 430.21 ms | 0.00% | 100.00% |
| A5 | Soak | 2 | 0.98 | 385.80 ms | 1200.00 ms | 0.00% | 100.00% |

## Part B
1. Fill in the results table for Steps A1 to A5.
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.53 | 429.85 ms | 799.18 ms | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.27 | 393.95 ms | 474.04 ms | 0.00% | 100.00% |
| A3 | Stress | 5 | 2.10 | 394.79 ms | 612.52 ms | 0.00% | 100.00% |
| A4 | Spike | 5 | 1.00 | 389.58 ms | 430.21 ms | 0.00% | 100.00% |
| A5 | Soak | 2 | 0.98 | 385.80 ms | 1200.00 ms | 0.00% | 100.00% |
2. In Step 0.2, why was the request rate below 5 requests per second with 5 virtual users? 
Ans อัตรา Request Rate ต่ำกว่า 5 req/s เกิดจาก Think Time 1 วินาทีบวกเวลา Latency ของ network
3. Did the response time grow from the average-load test to the stress test? Explain your answer with the operational envelope.
Ans Response time กับ Operational Envelope ค่า p(95) ระหว่าง Average Load (393.95 ms) กับ Stress (394.79 ms) แทบไม่เพิ่มขึ้น เนื่องจากโหลดระดับ 5 VUs ยังอยู่ภายใน Operational Envelope ของระบบ QuickPizza ทำให้ Server ไม่เกิด Request Queueing
4. In the spike test, how long did the response time take to return to normal after the spike?
Ans เมื่อ spike ขึ้นเป็น 5 VUs ระบบยังคงทำงานได้ทันที และกลับสู่ระดับ normal ต่ำกว่า 5–10 วินาที
5. In the soak test, did the response time or the error rate get worse over time? Name one kind of defect that a soak test of several hours could reveal.
Ans ในเวลา 10 นาที Error rate ยังคงเป็น 0.00% และ Response time มีความเสถียรดี โดยการทำ Soak Test เป็นเวลาหลายชั่วโมงจะช่วยตรวจจับปัญหา Memory Leaks, Database Connection Pool Exhaustion หรือ Disk Space เต็มจาก Unpruned Logs
6. Why do the thresholds in this lab use p(95) rather than the average?
Ans ค่าเฉลี่ยจะถูกเบี่ยงเบน Outliers เช่น ผู้ใช้ 95% เร็ว 200 ms แต่ 5% ช้าถึง 5 วินาที ค่าเฉลี่ยจะยังดูดี การใช้ Percentile p(95) แสดงให้เห็นประสบการณ์ของผู้ใช้ 95% ได้ถูกต้องกว่า
7. Your results come from a shared public site reached over the internet. Give two reasons why the same tests against a system in a lab could give different numbers.
Ans 1) Public Internet มี Latency, Hops, Jitter และ Wi-Fi ที่ผันผวน 
   2) QuickPizza มี Noisy Neighbors ที่ผู้ใช้คนอื่นทั่วโลกยิงทดสอบพร้อมกัน ไม่เหมือน Lab ที่ทรัพยากรถูกกำหนดแยกไว้
8. You must never run stress, spike, or breakpoint tests with hundreds of users against an external site you do not own because high load can overwhelm server hardware, exhaust network bandwidth, crash production databases, and deny service to real users (effectively constituting an accidental or deliberate Denial-of-Service / DoS attack). It can also inflict substantial cloud hosting bills on the site owners and violates acceptable use policies and computer fraud and abuse laws.
9. Why must you never run a stress, spike or breakpoint test with hundreds of users against a site that you do not own?
Ans เสี่ยงทำให้ระบบล่ม เกิด DoS เจ้าของระบบต้องจ่ายค่า Cloud จำนวนมาก และผิดกฎหมายพ.ร.บ.คอมพิวเตอร์
10. Write one vague performance requirement for an app you use every day. Rewrite it as a
testable requirement, and show the K6 thresholds that check it.
Ans Spotify Track Search
   - Requirement: เมื่อมีผู้ใช้ 20 VUs ค้นหาเพลงต่อเนื่อง 3 นาที ค่า p(95) ต้องต่ำกว่า 500 ms, p(99) ต่ำกว่า 1,000 ms และ Error rate ต้องน้อยกว่า 0.5%
   - K6 thresholds:
     ```javascript
     export const options = {
       thresholds: {
         'http_req_duration': ['p(95)<500', 'p(99)<1000'],
         'http_req_failed': ['rate<0.005'],
       },
     };
     ```
