# Lab Reflection — Day 23 Track 02 (Observability)

> Fill in each section. Grader reads the "What I'd change" paragraph closest.

**Student:** Nguyễn Đôn Đức
**Submission date:** 2026-05-11
**Lab repo URL:** https://github.com/edward1503/Day23-Track2-Observability-Lab

---

## 1. Track 01 — Instrumentation

### One thing I learned about metrics vs. logs
Logs provide the "why" (contextual details of a single request), while metrics provide the "what" and "how fast" (aggregates across all requests). In this lab, I saw how a single `inference_active_gauge` could tell me more about system concurrency than thousands of lines of logs ever could.

### Why use a Gauge for `inference_active` instead of a Counter?
A Counter only goes up, which is useful for total requests. However, concurrency (active requests) is transient—it goes up when a request starts and down when it ends. A Gauge is the only way to accurately represent this "current state" value.

---

## 2. Track 02 — Dashboards & Alerts

### 6 essential panels (screenshot)
![Dashboard Overview](./screenshots/dashboard-overview.png)

### Burn-rate panel
![SLO Burn-rate](./screenshots/slo-burn-rate.png)

### Alert fire + resolve

| When | What | Evidence |
|---|---|---|
| _T0_ | killed `day23-app`         | ![Alertmanager Firing](./screenshots/alertmanager-firing.png) |
| _T0+90s_ | `ServiceDown` fired   | ![Slack Firing](./screenshots/slack-firing.png) |
| _T1_ | restored app              | — |
| _T1+60s_ | alert resolved        | ![Slack Resolved](./screenshots/slack-resolved.png) |

### One thing surprised me about Prometheus / Grafana
I was surprised by the power of PromQL for calculating SLO Burn Rates. Seeing how a simple mathematical expression can predict if we will violate our SLA in the next few hours (before it actually happens) is a game-changer for proactive monitoring.

---

## 3. Track 03 — Tracing & Logs

### One trace screenshot from Jaeger
![Jaeger Trace](./screenshots/jaeger-trace.png)

### Log line correlated to trace
Paste the log line and the trace_id it links to:

```json
{"model": "llama3-mock", "input_tokens": 4, "output_tokens": 40, "quality": 0.937, "duration_seconds": 0.416, "trace_id": "36a61e0d9fe15cddf0383498a4acbb76", "event": "prediction served", "level": "info", "timestamp": "2026-05-11T14:41:09.123456Z"}
TraceID: 36a61e0d9fe15cddf0383498a4acbb76
```

### Tail-sampling math
If your service produced N traces/sec, what fraction did the policy keep?
The policy keeps 100% of errors and 1% of successful traces.
If 5% of traffic is errors, then: `(1.0 * 0.05) + (0.01 * 0.95) = 0.05 + 0.0095 = 0.0595`.
We keep roughly **5.95%** of the total traffic volume.

---

## 4. Track 04 — Drift Detection

### Did you detect drift? In which features?
Yes, drift was detected in **`prompt_length`** (PSI=3.461) and **`response_quality`** (PSI=8.849). This indicates that the input distribution has changed (longer prompts) and the model performance is degrading.

### Screenshot of Evidently report
![Drift Report](./screenshots/drift-report.png)

### Why use PSI (Population Stability Index) over just comparing means?
Comparing means can hide significant shifts in distribution (e.g., if the data becomes more spread out but the average stays the same). PSI looks at the entire distribution across buckets, making it much more sensitive to subtle but important changes in user behavior.

---

## 5. Summary & Integration

### Cross-Day Dashboard (screenshot)
![Cross-Day Stack](./screenshots/cross-day-stack.png)

### The single change that mattered most
Correcting the **Span Context Propagation** in `main.py` was the most significant change. Without moving the `predict` span into a context manager, the tracing was disconnected, making it impossible to see the "waterfall" of AI internal steps. This taught me that observability is only as good as the instrumentation's context-awareness.

### Final thoughts on the Full Stack
Building this stack from scratch showed me how OTel Collector acts as the "universal translator" between my application and multiple backends (Prometheus, Jaeger, Loki). It's a powerful architecture that ensures no vendor lock-in.
