# Week 6 worksheet: serving patterns

**Name:** Gianmario Carta

**Student ID:** 01224055

This is **individual work**: each student completes and submits their own copy, in their own fork.

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in English.

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large |513|661|93.5%|100%|
| t0-small |84|205|72.5%|0%|

**Q0.** What does the large model buy, and what does it cost?

> Gives better accuracy at the cost of slower inference speed

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before |28.6|234|368|20|0|92.8%|
| t1-after |9.8|408|1027|0|23|93.8%|

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

> The requests that errored out were never recorded, so we never see the long latency from them. Should notice the long run time along with the number of errors.

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

> The review service will keep holding on to the connection for 20 seconds while the client already gives up at 10 seconds. This could lead to lots of piled up connections that bottleneck the whole system.

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

> An asynchronous batch job can label them later. (Asynchronous / batch)

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

> if the timeout is too quick (0.1s), results might not be sent in time. If too late (10), it might hang the system for other processes

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 |23|499|91.5%|38%|
| t2-0.7 | 0.7 |373|572|93%|53%|
| t2-0.8 | 0.8 |431|568|93.5%|64%|
| t2-0.9 | 0.9 |477|599|93.5%|77%|

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

> 0.7 would probably be a good pick. It balances model accuracy with relatively low latency. Giving up 0.5% higher accuracy seems worth the lower latency.

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

> Many easy requests use the fast small model so p50 is improved, while p95 remains closer to the large model latency.

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

> One bad case I can think of is the small model being totally wrong on the intent of the review. A positive review could be flagged as a negative one even if the model is "confident".

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache |21|521|93%|38%|35%|

**Q3.1** Which reviews benefit from the cache, and which never will?

> Short phrases or one-liners would probably benefit from caching. Really long reviews probably wont be able to get a cache hit.

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

> Could maybe feed the cache through the new large model to verify whether their stored user intents are correct or not. Maybe have an automated pipeline that feeds the cache directly when a new tagged version of model is deployed.

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

> interpreting measurements and the real story behind them
