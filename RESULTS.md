# K6 lab results

## Part 0

- Step 0.1, http_req_duration: avg=med=min=max=p(90)=p(95)=307.21 ms (1 request)
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: TODO_requests requests, TODO_rate req/s, p(95) TODO_p95 ms. Each virtual user does one request per iteration and then sleeps for 1 second, so one iteration takes about 1 s plus the response time (roughly 1.3 s). One user therefore sends less than 1 request per second, and 5 users send less than 5 requests per second.
- Step 0.6, exit code of the normal run and of the run with P95=300: TODO (normal) and TODO (P95=300)
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: TODO
- Step 0.8, what was wrong, and how the checks showed it: TODO

## Part A results

| Step | Test type    | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1   | Smoke        | 1     |       |       |     |        |        |
| A2   | Average load | 3     |       |       |     |        |        |
| A3   | Stress       | 5     |       |       |     |        |        |
| A4   | Spike        | 5     |       |       |     |        |        |
| A5   | Soak         | 2     |       |       |     |        |        |

## Part B

1. See the results table in Part A above.

2. Each virtual user does one request, then sleeps 1 second (think time), so one iteration lasts about 1 s plus the response time. One user therefore sends under 1 request per second, and 5 users send under 5 requests per second (measured: TODO req/s).

3. TODO: compare p(95) of A2 (TODO ms) with A3 (TODO ms). Explain with the operational envelope: 3 and 5 users are far inside the envelope of the site, so the response time should change little. If it grew clearly, the load is approaching the edge of the envelope.

4. TODO: after the spike the response time returned to its earlier level after about TODO seconds (see the A4 report).

5. TODO: compare the first minutes and the last minutes of A5 (response time and error rate). A soak test of several hours could reveal a memory leak, an exhausted connection pool, leaked file handles, or a full disk or log.

6. The average hides the slowest requests. The p(95) shows the time that 95% of requests beat, so it reflects what the slower users really experience. Requirements are stated in percentiles for this reason.

7. (a) A lab system has no internet latency and no jitter, so response times would be lower and more stable. (b) In a lab, nobody else shares the server, while the public QuickPizza site serves users from all over the world, so its response times vary with other people's traffic.

8. Such a test can overload or crash the site, harming its real users, and it looks like a denial-of-service attack. Without the owner's written permission it may break the terms of use and the law.

9. Vague requirement: "Sending a message in my chat app should be fast."
   Testable requirement: With 3 users sending messages at the same time for 30 seconds, 95% of send requests complete within 500 ms and fewer than 1% fail.
   K6 thresholds:

   ```javascript
   thresholds: {
     http_req_duration: ['p(95)<500'],
     http_req_failed: ['rate<0.01'],
   },
   ```