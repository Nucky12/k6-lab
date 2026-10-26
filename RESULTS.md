# K6 lab results

## Part 0

- Step 0.1, http_req_duration: avg = med = min = max = p(90) = p(95) = 307.21 ms (only 1 request, so all values are the same).
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 75 requests, 3.72 requests per second, p(95) = 288.74 ms. One iteration of a virtual user is one request (about 0.28 s) followed by `sleep(1)`, so an iteration lasts about 1.3 s (measured average iteration_duration: 1.34 s). One user therefore sends only about 0.75 requests per second (1 / 1.34), and 5 users send about 5 x 0.75 = 3.7 requests per second, which is below 5. The sleep (think time) is part of every iteration, so the users spend most of their time waiting, not requesting.
- Step 0.6, exit code of the normal run and of the run with P95=300: exit code 0 for the normal run (p(95) = 413.74 ms, below 1500 ms, all three thresholds passed) and exit code 99 for the run with P95=300 (p(95) = 431.04 ms, above 300 ms, so the threshold on http_req_duration was crossed).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: With 3 users for 30 s, all three rules passed: p(95)<1000 with p(95) = 445.05 ms, p(99)<1500 with p(99) = 590.48 ms, and http_req_failed rate<0.01 with rate = 0.00%. I then changed the p(95) limit from 1000 ms to 50 ms (`-e P95=50`). The run then failed: p(95)<50 was crossed (p(95) = 427.77 ms), while p(99)<1500 (p(99) = 452.9 ms) and http_req_failed still passed. The exit code was 99.
- Step 0.8, what was wrong, and how the checks showed it: The script sent the names in lower case, `excludedIngredients: ['pepperoni']` and `excludedTools: ['knife']`, but the API writes them as `Pepperoni` and `Knife` (shown by explore.js), so the API did not apply the exclusions. The checks showed it: with 5 users for 20 s, "no pepperoni" passed only 90% (63 passed, 7 failed) and "no knife needed" only 60% (42 passed, 28 failed), so 83.33% of all checks passed (175 of 210). After changing the request to `['Pepperoni']` and `['Knife']`, all 213 of 213 checks passed. A single manual request can pass by luck, because the API picks a random pizza and a random pizza often has no pepperoni or no knife anyway (the checks passed 90% and 60% of the time even with the bug). Only many requests under load make the wrong answers show up as a failure rate, and a failure rate is easy to see and count.

## Part A results

| Step | Test type    | Users | Req/s | p(95)    | Max      | Failed | Checks               |
| ---- | ------------ | ----- | ----- | -------- | -------- | ------ | -------------------- |
| A1   | Smoke        | 1     | 0.44  | 380.49 ms | 407.55 ms | 0.00%  | 100.00% (20 of 20)   |
| A2   | Average load | 3     | 1.33  | 401.55 ms | 1.49 s    | 0.00%  | 100.00% (324 of 324) |
| A3   | Stress       | 5     | 2.20  | 383.14 ms | 754.99 ms | 0.00%  | 100.00% (532 of 532) |
| A4   | Spike        | 5     | 1.02  | 397.71 ms | 446.42 ms | 0.00%  | 100.00% (192 of 192) |
| A5   | Soak         | 2     | 0.97  | 400.55 ms | 1.38 s    | 0.00%  | 100.00% (852 of 852) |

## Part B

1. The results table for Steps A1 to A5 is in the Part A section above.

2. In Step 0.2 the rate was 3.72 requests per second, below 5. Each virtual user does one request (about 0.28 s) and then sleeps for 1 second as think time, so one iteration took about 1.34 s on average. One user sends only about 0.75 requests per second, so 5 users send about 3.7 requests per second, not 5.

3. The response time did not grow from the average-load test to the stress test. The p(95) of http_req_duration was 401.55 ms with 3 users (A2) and 383.14 ms with 5 users (A3). The difference is within the normal noise of a public site (for example the max of A2 was 1.49 s because of one slow request, while the max of A3 was 754.99 ms). The recommendation group shows the same: p(95) 466.78 ms in A2 and 398.27 ms in A3. At the same time the request rate rose from 1.33 to 2.20 requests per second, a factor of 1.66, which is almost the same as the factor of users (5 / 3 = 1.67). So the throughput grew in proportion to the users while the response time stayed flat. In terms of the operational envelope, 3 and 5 users are both far inside the envelope of QuickPizza, so the system answers just as fast. This does not show where the edge of the envelope is, because the lab limits us to 5 users; finding the edge would need a breakpoint test, which we must not run against a public site.

4. The spike test did not slow the system down in a measurable way, so there was no recovery time to measure. During the whole run, including the jump from 1 to 5 users within 5 seconds and the 30 seconds at 5 users, the slowest request took only 446.42 ms (the median of the run was 291.38 ms), the p(95) of the run was 397.71 ms, which is close to the 380.49 ms of the smoke test with 1 user, and no request failed (0.00%). The response time therefore stayed at its normal level, which means it "returned to normal" immediately, within the same few seconds as the drop back to 1 user. A jump from 1 to 5 users is a small spike for the system.

5. The response time and the error rate did not get worse over time in the soak test. In the 11-minute run with 2 users, the error rate stayed at 0.00% (0 of 639 requests failed) and all 852 checks passed. The p(95) of the whole run was 400.55 ms, the same level as in the smoke test (380.49 ms) and the average-load test (401.55 ms); if the response time had kept growing, the p(95) of the long run would have been clearly higher than that of the short runs. There was one slow outlier of 1.38 s, but the p(95) shows it is rare. Ten minutes with 2 users is too short to find slow defects. A soak test of several hours could reveal a memory leak (memory use grows until the service slows down or crashes), but also an exhausted connection pool, leaked file handles, or a disk or log that slowly fills up.

6. The average hides the slowest requests: a few very slow requests barely move it. The p(95) is the time that 95% of the requests beat, so it shows what the slower 1 in 20 requests experience. For example, in A2 the average was 315.37 ms but the max was 1.49 s. Requirements and thresholds therefore use percentiles, because users remember the slow answers.

7. (a) A system in a lab has no internet between the tester and the server, so there is no network latency and jitter, and the response times would be lower and more stable than the roughly 300 ms measured here. (b) In a lab nobody else uses the server, while QuickPizza is shared with people all over the world who also learn K6, so its response time depends on the traffic of others and on the time of day. A lab also has a fixed, known hardware and configuration, so repeated runs give nearly the same numbers.

8. A stress, spike or breakpoint test with hundreds of users can overload or crash a site that we do not own, which harms its real users and costs its owner money. It also looks the same as a denial-of-service attack, so without the written permission of the owner it can break the terms of use and the law. This is why this lab limits QuickPizza to 5 virtual users.

9. Vague requirement: "Sending a message in my chat app should be fast."

   Testable requirement: With 3 users sending messages at the same time for 30 seconds, 95% of the send requests complete within 500 ms and fewer than 1% of the requests fail.

   K6 thresholds:

   ```javascript
   export const options = {
     vus: 3,
     duration: '30s',
     thresholds: {
       http_req_duration: ['p(95)<500'],
       http_req_failed: ['rate<0.01'],
     },
   };
   ```
