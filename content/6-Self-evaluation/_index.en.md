---
title: "Self-evaluation"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

During my internship at **Amazon Web Services Vietnam Company Limited** from **June 22, 2026** to **August 15, 2026**, I had the opportunity to learn, practice, and apply academic knowledge gained from university into a real-world working environment.  
I participated in the **RAG on AWS** project, thereby improving my skills in **programming, project deployment, and communication**.

Regarding work ethic and attitude, I always strived to complete assigned tasks well, adhered to rules and regulations, and actively communicated with colleagues to enhance work efficiency.

To objectively reflect on my internship process, I evaluate myself based on the following criteria:

| No. | Criteria                                 | Description                                                                 | Good | Fair | Average | Evidence from this internship |
| --- | ---------------------------------------- | --------------------------------------------------------------------------- | ---- | ---- | ------- | --- |
| 1   | **Professional Knowledge & Skills**      | Domain understanding, practical application, tool proficiency, work quality | ☐    | ✅   | ☐       | Built Flows 1–3 (ingestion, Q&A, monitoring) with Terraform; Flow 4 RAGAS still Partial (skeleton). |
| 2   | **Learning Ability**                     | Absorbing new knowledge, fast learning                                      | ☐    | ✅   | ☐       | Learned Bedrock, Cognito, hybrid search, and the ECR 2-phase gate the hard way via docs + debugging. |
| 3   | **Proactiveness**                        | Self-researching, taking initiative without waiting for instructions        | ✅   | ☐    | ☐       | Self-traced `/documents-decision` **504** to a missing VPC endpoint (documented in [5.10.1](../5-Workshop/5.10-System-Testing/5.10.1-Manual-E2E-Testing/)). |
| 4   | **Sense of Responsibility**              | Completing work on time, ensuring quality                                   | ✅   | ☐    | ☐       | Delivered weekly worklogs and kept IaC modules aligned with what actually ran in AWS. |
| 5   | **Discipline**                           | Punctuality, compliance with rules, adherence to workflows                  | ✅   | ☐    | ☐       | Followed check-in cadence and least-privilege IAM reviews before demo weeks. |
| 6   | **Desire to Learn & Improve**            | Willingness to receive feedback and self-improve                            | ☐    | ✅   | ☐       | Accepted Week 7 feedback (rate-limiting, larger eval set) and logged them as Deferred, not ignored. |
| 7   | **Communication**                        | Presenting ideas, reporting work clearly                                    | ✅   | ☐    | ☐       | Wrote bilingual report sections and presented an honest Partial status for Flow 4 in demo wrap-up. |
| 8   | **Teamwork & Collaboration**             | Working effectively with colleagues, team participation                     | ✅   | ☐    | ☐       | Coordinated Slack Chatbot/OAuth with workspace admin; shared debugging notes with teammates. |
| 9   | **Professional Conduct**                 | Respecting colleagues, partners, and the work environment                   | ✅   | ☐    | ☐       | Kept credentials out of git; used Budget alerts and teardown scripts to avoid cost surprises. |
| 10  | **Problem-Solving Mindset**              | Identifying issues, proposing solutions, creativity                         | ☐    | ✅   | ☐       | Strong on infra/debug (504, DLQ, cache races); weaker on closing quantitative RAGAS evidence. |
| 11  | **Contribution to Project/Organization** | Work effectiveness, improvement initiatives, team recognition               | ☐    | ✅   | ☐       | Contributed a working RAG stack + report; still owe production-hardened Flow 4 evidence. |
| 12  | **Overall**                              | General assessment of the entire internship process                         | ✅   | ☐    | ☐       | Solid operable delivery on Flows 1–3 + docs; growth area is finishing evaluation evidence loops. |

### Areas for Improvement

- Improve knowledge absorption speed on new AWS/GenAI pieces and turn them into production-proven runs sooner (especially Flow 4 image push + invoke).
- Deepen hands-on practice beyond “works in demo” — attach Console evidence whenever a flow is claimed Done.
- Communicate task ownership earlier in team settings so parallel work does not leave gaps.
- Strengthen requirement checking before marking report sections complete (the earlier miss on broken image paths and overclaimed RAGAS wording is a concrete lesson).
