---
title: "End-to-End Testing"
date: 2026-08-03
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

After SNS ([5.5.1](../5.5.1-SNS-2-Channels-By-Severity/)), Alarms ([5.5.2](../5.5.2-CloudWatch-Alarms/)), and Dashboard ([5.5.3](../5.5.3-Dashboard-Custom-Metrics/)), the last step is confirming the alert chain actually works — often skipped because infrastructure "looks fine" even when the email subscription is still Pending or Chatbot is not attached to Slack.

Stream 3 is applied (2 SNS topics, 4 alarms, 9-widget dashboard). Diagram to compare with Console:

<div align="center">

![Detailed Diagram for Stream 3 - Monitoring and Alerting](/images/5-Workshop/5.5-Monitorning/image.png)

<p style="font-size: 0.85em; color: #666; font-style: italic; margin-top: 8px;">
Figure 5.5.4a. Monitoring architecture as declared — SNS by severity, CloudWatch Alarms, overview dashboard
</p>

</div>

#### Test scenarios and results

| #   | Scenario                                                                                  | Expected result                                                                                                       | Result |
| --- | ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------ |
| 1   | SNS Console → `alerts-info` subscription                                                  | Status `Confirmed` (not `Pending confirmation`)                                                                       | Pass — confirmation email clicked; subscription no longer Pending |
| 2   | AWS Chatbot → Slack channel configuration                                                 | Correct workspace/channel; if `slack_*` is `NONE`, the Chatbot resource does not exist (by design)                    | Pass per tfvars — Chatbot is created only when Slack is fully set |
| 3   | Simulate Lambda failures (throw exceptions for several minutes past the Errors threshold) | `lambda-errors` → `ALARM`, email via `alerts-info`                                                                    | Pass (manual inject) — alarm went ALARM then back to OK |
| 4   | Cause API/Lambda 5xx (e.g. a transient fault in chat-engine returning 500)                | `apigw-5xx` → `ALARM` when 5xx rate exceeds the % threshold, email received                                           | Pass (manual inject) — server-side 5xx, not 4xx |
| 5   | Trigger `ThrottlingException` (burst Bedrock calls or mock the error in code)             | Log filter matches, `bedrock-throttle` → `ALARM`, Slack message via `alerts-critical`                                 | Pass when Slack is configured; if `slack_*` is `NONE`, alarm is Console-only |
| 6   | Push one failing message into a DLQ (ingestion DLQ or function DLQ)                       | `dlq-depth` → `ALARM` as soon as depth > 0, Slack message                                                             | Pass (manual inject) — depth > 0 is Critical |
| 7   | Open dashboard `${name_prefix}-overview`                                                  | 7 AWS widgets populate after traffic; Cache Hit Rate after a few `/chat` calls; RAGAS after evaluation has run ≥ once | Partial — widgets 1–8 populate after UI traffic; RAGAS widget empty because Stream 4 is not deployed |
| 8   | Stop the fault simulation after `ALARM`                                                   | Alarm returns to `OK` on its own once the threshold is no longer breached — no manual reset                           | Pass — `treat_missing_data = notBreaching`, returns to OK by itself |

{{% notice tip %}}
Scenario 4: a wrong path/method usually yields **4xx**, which does not trip `apigw-5xx`. You need a server-side failure (5xx) or a high enough 5xx rate in the 5-minute window.
{{% /notice %}}

{{% notice note %}}
📌 **Console screenshots** (Alarm OK/ALARM, SNS email or Slack critical, dashboard overview) are **not attached as separate files yet**. Mentors can match Terraform + the diagram above with [5.5.1](../5.5.1-SNS-2-Channels-By-Severity/)–[5.5.3](../5.5.3-Dashboard-Custom-Metrics/). When captured, drop them as `alarm-ok-or-alarm.png`, `sns-email-or-slack.png`, `dashboard-overview.png` under `static/images/5-Workshop/5.5-Monitorning/`.
{{% /notice %}}

#### Outcomes

- Two SNS channels by severity — Warning (email) and Critical (Slack) — reduce alarm fatigue.
- `bedrock-throttle` reuses Lambda logs (Stream 2) instead of a separate throttle pipeline.
- `dlq-depth` covers both DLQ layers (Stream 1) — no silent backlog on one tier.
- Dashboard with 9 widgets: 7 AWS metrics + cache hit rate (EMF) + RAGAS (`put_metric_data`) — the RAGAS widget stays empty until Stream 4 runs.
- `treat_missing_data = "notBreaching"` avoids false alarms in low-traffic environments.
