# K6 lab results


## Part 0

- **Step 0.1, http_req_duration:** ได้ 386.03 ms มี Requestแค่ครั้งเดียว ค่า avg, min, med, max, p(90) และ p(95) เท่ากันทั้งหมด

- **Step 0.2, requests, rate and p(95), and why the rate stays below 5:** ได้ 75 Requests, Request Rate อยู่ที่ 3.70 req/s และ p(95) = 402.49 ms ไม่ถึง 5 req/s เพราะแต่ละ User ต้องรอให้ Request ตอบกลับก่อน แล้วค่อย sleep อีก 1 วินาที ทำให้หนึ่งรอบใช้เวลาประมาณ 1.3 วินาที ดังนั้น 1 User จึงส่งได้ประมาณ 0.77 Requests/sec มี 5 Users ได้ประมาณ 3.8 req/s ซึ่งใกล้เคียงกับที่เทส

- **Step 0.6, exit code of the normal run and of the run with P95=300:** รันครั้งแรกใช้ Threshold p(95)<1500 ms ได้ p(95) = 430.57 ms และ Exit Code = **0** แปลว่าผ่าน ส่วนครั้งที่สองเปลี่ยนเป็น 300 ms ได้ p(95) = 421.15 ms ซึ่งไม่ผ่านและได้ Exit Code = **99**

- **Step 0.7, rules that passed, p(95), p(99), exit code after your change:** ตั้ง 3 VUs เป็น 30 วินาทีและ p(95)<1000 ms, p(99)<1500 ms และ Request Failure Rate<1% ผลการทดสอบผ่านทั้ง 3 อัน ได้ p(95) = 401.10 ms, p(99) = 416.73 ms และ Failed Requests = 0.00% และปรับ p(95) จาก 1000 เป็น 200 ms ผลที่ได้คือ p(95) = 503.94 ms ทำให้ไม่ผ่านและได้ Exit Code = **99**

- **Step 0.8, what was wrong, and how the checks showed it:** ครั้งแรก Checks ผ่านเพียง 88.44% โดย `no pepperoni` ไม่ผ่าน 7 ครั้ง และ `no knife needed` ไม่ผ่าน 19 ครั้ง โดยที่ HTTP Status เป็น 200 ทุกครั้ง

  จากนั้นใช้ `explore.js` ตรวจสอบ ชื่อ Ingredient และ Tool ที่ API ใช้คือ `Pepperoni` และ `Knife` เป็นตัวอักษรขึ้นต้นด้วยตัวใหญ่ แต่ใน Request แรกส่งเป็นตัวพิมพ์เล็ก ทำให้ไม่ตรงกับค่าที่ API รู้จัก

  แก้ชื่อให้ตรงกับ API แล้วทดสอบอีกครั้ง Checks ผ่าน 100% (222/222)

  ทำหลายครั้งเพราะผลลัพธ์ Pizza เป็นการสุ่ม ถ้าทดลองเแค่ครั้งเดียวอาจยังไม่เจอ Pepperoni แต่ถ้าเพิ่มจำนวน Iterations ก็มีโอกาสเจอกรณีที่ API ไม่ทำตามเงื่อนไขที่

## Part A results

| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
|---|---|---:|---:|---:|---:|---:|---:|
| A1 | Smoke | 1 | 0.54 | 375.92 ms | 398.32 ms | 0.00% | 100.00% (24/24) |
| A2 | Average load | 3 | 1.31 | 410.94 ms | 463.72 ms | 0.00% | 100.00% (320/320) |
| A3 | Stress | 5 | 2.13 | 384.54 ms | 470.53 ms | 0.00% | 100.00% (520/520) |
| A4 | Spike | 5 | 1.01 | 401.67 ms | 444.11 ms | 0.00% | 100.00% (196/196) |
| A5 | Soak | 2 | 0.97 | 398.02 ms | 465.20 ms | 0.00% | 100.00% (860/860) |

## Part B

**1. Fill in the results table for Steps A1 to A5.**

Checks ผ่าน 100% ไม่มี Request Failure โดยค่า p(95) อยู่ในช่วงป 375–411 ms, QuickPizza ตอบสนองได้สม่ำเสมอ ถึงแม้จะเปลี่ยน Virtual Users และการทดสอบ

**2. In Step 0.2, why was the request rate below 5 requests per second with 5 virtual users?**

เพราะ Virtual User แต่ละคนต้องรอ Response จาก Server ก่อน ถึงจะ sleep อีก 1 วินาที ทำให้หนึ่งรอบใช้เวลา 1.3 วินาที ดังนั้น 1 User จึงส่ง Request ได้ไม่ถึง 1 ครั้งต่อวินาที มี 5 Users จะได้ประมาณ 3.7 Requests/sec

**3. Did the response time grow from the average-load test to the stress test? Explain your answer with the operational envelope.**

Response Time ไม่ได้เพิ่มขึ้นเมื่อเพิ่มโหลดจาก 3 เป็น 5 Users, A2 มี p(95) = 410.94 ms ส่วน A3 มี p(95) = 384.54 ms ต่ำกว่าการทดสอบ Average Load แม้ Request Rate จะเพิ่มจาก 1.31 เป็น 2.13 req/s

แปลว่าโหลดที่ใช้ในการทดสอบยังไม่มากพอที่จะทำให้ประสิทธิภาพลดลง เลยยังบอกไม่ได้ว่าระบบจะเริ่มมีปัญหาเมื่อรับโหลดสูงสุดเท่าไร

**4. In the spike test, how long did the response time take to return to normal after the spike?**

ก่อนเกิด Spike ค่า p(95) อยู่ประมาณ 293–403 ms ส่วนช่วงที่เพิ่มเป็น 5 Users อยู่ประมาณ 383–406 ms หลังจากลดจำนวน Users กลับมาเหลือ 1 คน ค่า Response Time กลับมาอยู่ในระดับเดิม ไม่มี Request Failure แสดงว่า Spike ที่ 5 Users ไม่ได้ส่งผลต่อระบบมาก

**5. In the soak test, did the response time or the error rate get worse over time? Name one kind of defect that a soak test of several hours could reveal.**

จาก Soak Test ใช้ 2 Users 10 นาที Response Time ไม่ได้เพิ่มขึ้นอย่างต่อเนื่อง เมื่อเทียบกับ 2 นาทีแรก กับ 2 นาทีสุดท้ายก่อน Ramp Down ค่า Average Response Time เพิ่มจาก 307.7 ms เป็น 317.5 ms ต่างกันแค่นิดเดียว

ส่วน p(95) ในแต่ละช่วงใกล้เคียงกัน และ Error Rate เป็น 0% ตลอด เลยไม่สรุปว่าประสิทธิภาพลดลง

ปัญหาที่ Soak Test ระยะเวลาหลายชั่วโมงอาจจะเจอได้ก็คือ Memory Leak ซึ่งถ้าใช้หน่วยความจำเพิ่มขึ้นเรื่อยๆอาจจะทำให้ระบบอาจทำงานช้าลงหรือหยุดทำงานได้

**6. Why do the thresholds in this lab use p(95) rather than the average?**

เพราะ Average ทำให้มองไม่เห็น Request ที่ใช้เวลาตอบกลับนานมาก ๆ แต่ p(95) แสดงเวลาที่ 95% ของ Requests สามารถตอบกลับได้ภายในเวลาที่กำหนด ช่วยให้เห็นได้ดีกว่าการดูค่าเฉลี่ยอย่างเดียว

**7. Your results come from a shared public site reached over the internet. Give two reasons why the same tests against a system in a lab could give different numbers.**

มี 2 ปัจจัยหลักที่อาจทำให้ผลการทดสอบแตกต่างกัน

1. **Network Latency:** การทดสอบนี้ยิง Request จากประเทศไทยไปยัง Server ผ่านอินเทอร์เน็ต ทำให้มีเวลาในการส่งข้อมูลไปและกลับเพิ่มเติม หากทดสอบในเครื่องที่อยู่ใกล้กับ Server มากกว่า Response Time ก็อาจต่ำลงได้
2. **Shared Server Resources:** QuickPizza เป็นเว็บสาธารณะ มีคนอื่นใช้ทดสอบพร้อมกัน จะทำให้โหลดบน Server เปลี่ยนแปลงตลอดเวลา

**8. Why must you never run a stress, spike or breakpoint test with hundreds of users against a site that you do not own?**

เพราะการส่ง Request จำนวนมากไปยังเว็บไซต์ของผู้อื่นอาจทำให้ Server ทำงานหนักจนตอบสนองช้าลงหรือไม่สามารถให้บริการผู้ใช้จริงได้ คล้ายการโจมตีแบบ Denial of Service (DoS) นอกจากนี้อาจจะสร้างค่าใช้จ่ายให้เจ้าของระบบ ควรทำ Performance Testing กับระบบที่เราเป็นเจ้าของหรือได้รับอนุญาต

**9. Write one vague performance requirement for an app you use every day. Rewrite it as a testable requirement, and show the K6 thresholds that check it.**

ตัวอย่างแอปสั่งอาหาร

**Vague Requirement:**

การค้นหาร้านอาหารต้องทำได้รวดเร็ว แม้ในช่วงเวลาเที่ยงที่มีคนใช้งานจำนวนมาก

**Testable Requirement:**

ในช่วงเวลาเที่ยง เมื่อมีผู้ใช้งาน 200 คนค้นหาร้านอาหารพร้อมกันเป็นเวลา 10 นาที API ค้นหาร้านอาหารต้องตอบกลับภายใน 800 ms สำหรับ 95% ของ Requests และภายใน 1,500 ms สำหรับ 99% ของ Requests โดยมี Request Failure Rate น้อยกว่า 0.5% และต้องแสดงผลการค้นหาที่มีร้านอาหารอย่างน้อย 1 ร้านได้มากกว่า 99% ของ Requests

**K6 Thresholds:**

```javascript
export const options = {
  stages: [
    { duration: '2m', target: 200 },
    { duration: '10m', target: 200 },
    { duration: '1m', target: 0 },
  ],
  thresholds: {
    'http_req_duration{name:search}': [
      'p(95)<800',
      'p(99)<1500',
    ],
    'http_req_failed{name:search}': [
      'rate<0.005',
    ],
    checks: ['rate>0.99'],
  },
};
```

`name: 'search'` ให้กับ Request ที่ใช้ค้นหาร้านอาหาร และเพิ่ม Check เพื่อตรวจว่าผลลัพธ์มีร้านอาหารอย่างน้อย 1 ร้านจริง การทดสอบด้วย 200 Users นี้ควรทำกับระบบของเราเองหรือระบบที่ได้รับอนุญาตเท่านั้น