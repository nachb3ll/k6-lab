# K6 lab results

## Part 0
- Step 0.1, http_req_duration: 282.24 ms
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 80 requests, 3.79 req/s, p(95) = 273.94 ms. The rate stays below 5 req/s because each iteration consists of an HTTP request (~265 ms) plus sleep(1) (1000 ms), totalling ~1.26-1.31 s per iteration. Therefore, each VU produces at most ~0.76-0.79 req/s, resulting in an aggregate rate of ~3.8 req/s for 5 VUs.
- Step 0.6, exit code of the normal run and of the run with P95=300: Normal run exit code is 0 (all thresholds met); run with P95=300 exit code is 99 (p(95) of 389.52 ms breached the 300 ms threshold).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: All rules passed (http_req_failed rate<0.01, http_req_duration p(95)<1000, p(99)<1500), measured p(95) = 389.18 ms, p(99) = 393.09 ms. After changing p(95)<1000 to p(95)<300 to induce failure, the exit code was 99.
- Step 0.8, what was wrong, and how the checks showed it: The request used lowercase 'pepperoni' and 'knife', but the QuickPizza API requires exact case-sensitive names 'Pepperoni' and 'Knife' (as revealed by explore.js). Checks under load caught this because QuickPizza randomly selects ingredients and tools; a single manual request might coincidentally omit Pepperoni and Knife, whereas 75 requests under load hit Pepperoni 7 times and Knife 21 times, causing check failures. Once fixed to 'Pepperoni' and 'Knife', 100% of checks passed.

## Part A results
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.53 | 429.85 ms | 799.18 ms | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.27 | 393.95 ms | 474.04 ms | 0.00% | 100.00% |
| A3 | Stress | 5 | 2.10 | 394.79 ms | 612.52 ms | 0.00% | 100.00% |
| A4 | Spike | 5 | 1.00 | 389.58 ms | 430.21 ms | 0.00% | 100.00% |
| A5 | Soak | 2 | 0.98 | 385.80 ms | 1200.00 ms | 0.00% | 100.00% |

## Part B
1. See the summary table in "Part A results" above, comparing Smoke (1 VU), Average load (3 VUs), Stress (5 VUs), Spike (5 VUs peak), and Soak (2 VUs).
2. In Step 0.2, each virtual user makes an HTTP request (~265 ms round-trip) followed by a mandatory `sleep(1)` (1000 ms think time). Therefore, one complete iteration takes ~1.26–1.31 seconds. This caps each virtual user at roughly 0.76–0.79 requests per second. With 5 concurrent virtual users, the aggregate request rate is at most $5 \times 0.76 \approx 3.8$ req/s, which mathematically cannot reach 5 requests per second.
3. The response time grew only very slightly between the average-load test (p(95) = 393.95 ms) and the stress test (p(95) = 394.79 ms), with max duration moving from 474.04 ms to 612.52 ms. In terms of the system's operational envelope, 5 concurrent users is still well within the normal capacity that QuickPizza is architected to handle. Because the traffic did not approach or exceed the edge of the operational envelope, server queues did not back up, CPU/memory remained uncongested, and response times remained stable near the baseline network latency.
4. During the spike test, when load surged from 1 user to 5 users within 5 seconds, the p(95) remained stable around ~389.58 ms and max response time peaked at 430.21 ms. Because the spike volume was modest relative to server capacity, the server absorbed the sudden surge immediately without persistent queuing, and response times stabilized back to normal within seconds (under 5–10 seconds) once traffic stabilized.
5. In the soak test (2 users for 10 minutes), the response time and error rate did not degrade over time; error rate remained 0.00% and response times remained consistent between the first and last minutes. A soak test running over several hours could reveal defects such as memory leaks (where memory is allocated but never released, gradually exhausting heap space and triggering severe garbage collection pauses or OutOfMemory crashes), resource/socket leaks (unclosed database connection pools or file descriptors), or database degradation (table fragmentation or unindexed growth causing queries to slow down over time).
6. Thresholds use p(95) rather than arithmetic average because averages mask extreme outliers. In a web service, 95% of requests might take 200 ms while the remaining 5% suffer severe 5-second delays; the average would appear deceptively low (~440 ms), hiding poor user experience. The p(95) percentile guarantees that 95% of users receive responses within the stated SLA/SLO boundary, representing true user experience much more reliably.
7. Two reasons why testing against a lab system would give different numbers:
   - Network latency and jitter: Testing over the public internet introduces variable routing hops, CDN caches, ISP bandwidth limits, and Wi-Fi latency, whereas an isolated LAN in a lab offers sub-millisecond, highly predictable latency with zero internet jitter.
   - Multi-tenancy and noisy neighbors: QuickPizza is a shared public demo instance accessed simultaneously by students worldwide, introducing uncontrollable background load. A private lab environment provides dedicated, isolated CPU and memory, ensuring 100% reproducible test conditions.
8. You must never run stress, spike, or breakpoint tests with hundreds of users against an external site you do not own because high load can overwhelm server hardware, exhaust network bandwidth, crash production databases, and deny service to real users (effectively constituting an accidental or deliberate Denial-of-Service / DoS attack). It can also inflict substantial cloud hosting bills on the site owners and violates acceptable use policies and computer fraud and abuse laws.
9. Example application: Spotify Search
   - Vague requirement: "The search feature in Spotify must be fast and reliable for all users."
   - Testable requirement: "Under a normal load of 20 concurrent users over 3 minutes, 95% of track search requests (`GET /v1/search`) must return within 500 ms, 99% within 1000 ms, and less than 0.5% of requests may fail."
   - K6 thresholds:
     ```javascript
     export const options = {
       thresholds: {
         'http_req_duration': ['p(95)<500', 'p(99)<1000'],
         'http_req_failed': ['rate<0.005'],
       },
     };
     ```
