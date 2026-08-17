---
title: "End-to-End Testing"
date: 2026-08-07
weight: 4
chapter: false
pre: " <b> 5.7.4. </b> "
---

After [5.7.1](../5.7.1-Frontend-Architecture-Authentication/)–[5.7.3](../5.7.3-Deployment-Hosting/), I opened `ui/index.html`, pasted connection fields from `terraform output`, and signed in with a Cognito user to run the UI scenarios below against the live stack.

Idle / ready UI (before running scenarios):

<div align="center">

![Two-pane UI during end-to-end testing](/images/5-Workshop/5.7-Frontend/image.png)

<p style="font-size: 0.85em; color: #666; font-style: italic; margin-top: 8px;">
Figure 5.7.4a. Idle UI — Cognito form, ingest / chat not run yet
</p>

</div>

**Pass evidence — ingest** (`.txt` uploaded, “Luồng nạp tài liệu” all green / hoàn tất; log shows failed then successful login):

<div align="center">

![Ingest pipeline pass](/images/5-Workshop/5.7-Frontend/04-ingest-flow-pass.png)

<p style="font-size: 0.85em; color: #666; font-style: italic; margin-top: 8px;">
Figure 5.7.4b. Ingest Pass + Cognito login (IdToken in the log)
</p>

</div>

**Pass evidence — Q&A** (question answered, “Luồng hỏi đáp” completed with step timings):

<div align="center">

![Q&A pipeline pass](/images/5-Workshop/5.7-Frontend/05-qa-flow-pass.png)

<p style="font-size: 0.85em; color: #666; font-style: italic; margin-top: 8px;">
Figure 5.7.4c. Q&amp;A Pass — grounded, cache miss, hybrid search
</p>

</div>

#### UI test scenarios

| #   | Scenario                                                         | Expected                                                            | Result | Figure |
| --- | ---------------------------------------------------------------- | ------------------------------------------------------------------- | ------ | ------ |
| 1   | Login with a valid Cognito user                                  | Receive `IdToken`; auth indicator turns green; Upload / Ask enabled | Pass — token received, controls enabled | [5.7.1b–c](../5.7.1-Frontend-Architecture-Authentication/) |
| 2   | Login with wrong password                                        | Clear error; UI does not crash; buttons stay disabled               | Pass — error shown, buttons remained disabled | [5.7.1c](../5.7.1-Frontend-Architecture-Authentication/) (red log) |
| 3   | Upload `.txt` / `.md` (text)                                     | Content appears in the editor and can be edited before send         | Pass — text editable in the editor | 5.7.4b |
| 4   | Upload a PDF that already has a text layer                       | Ingest completes without OCR confirm; extract via `pypdf`           | Pass — no OCR dialog; ingest completed | — |
| 5   | Upload a scanned PDF (no text) — choose **Yes**                  | OCR confirm dialog → Textract → pipeline continues                  | Pass — dialog shown; Textract path continued | N/A (dialog not captured) |
| 6   | Upload a scanned PDF — choose **No**                             | Stops as cancelled; no Textract cost                                | Pass — cancelled status; no Textract call | N/A |
| 7   | Ask about the document just uploaded                             | Grounded answer; source file tags when present                      | Pass — answer grounded with source tags | 5.7.4c |
| 8   | Ask the **same** question again (**same** session)               | `cache hit` tag; mostly one lit step; response &lt; 1s              | Pass — cache hit; response under 1s | — |
| 9   | Vague follow-up in the **same** session (“còn cái kia thì sao?”) | Rewrite step lights; log shows rewritten query                      | Pass — rewrite step lit; rewritten query in log | — |
| 10  | Ask something not in the corpus                                  | Model refuses / stays grounded instead of inventing freely          | Pass — refused / stayed grounded | — |
| 11  | Burst many chat requests quickly                                 | Possible 429 with `⏳` (retryable) — wait a few seconds and retry   | Pass — 429 with retry affordance; retry succeeded | — |

{{% notice tip %}}
If ingest hangs past ~90s, check CloudWatch Logs for `document-processor` and SQS/DLQ depth (Stream 3 alarms). The UI timeout is intentional so a stuck pipeline surfaces instead of spinning forever.
{{% /notice %}}

#### Outcomes

- Single-file UI exercises all four Stream 2 routes with a real Cognito JWT.
- Upload path distinguishes text vs binary (base64) for Textract.
- Pipeline animation reflects server `trace` / status timings, including skipped steps.
- Cache hit and multi-turn rewrite are visible without opening the AWS Console.
- Rate-limit (429) behavior is distinguishable from hard errors via the `⏳` affordance.
