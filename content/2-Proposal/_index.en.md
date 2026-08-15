---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# RAG Knowledge Assistant

## An AWS Serverless Solution for Internal Document Q&A

---

### 1. Executive Summary

This project started from a real problem many organizations face: internal knowledge is abundant, but retrieving it is slow and frustrating.

**RAG Knowledge Assistant** is an internal document Q&A chatbot built on **Retrieval-Augmented Generation (RAG)** architecture. Instead of relying solely on an LLM's general pre-trained knowledge, the system lets employees upload actual documents (PDFs, scanned images, plain text) and receive answers grounded directly in that content.

The entire stack runs on **AWS Serverless** — Lambda, SQS, Amazon Bedrock, DynamoDB — provisioned through Terraform for consistent, reviewable, and reproducible deployments. Beyond the core Q&A experience, the platform includes: semantic caching to control costs, content moderation via Bedrock Guardrails, real-time operational monitoring, and an automated daily quality evaluation loop using the RAGAS framework.

---

### 2. Problem & Solution

#### The Problem

When I looked at the internal knowledge management challenge, three specific pain points stood out as genuinely worth solving:

**First**, enterprise knowledge is typically scattered across hundreds of PDFs and scanned files. Every lookup means opening files manually, searching page by page — slow and repetitive work that adds up quickly.

**Second**, off-the-shelf LLMs respond fluently but are not grounded in an organization's actual internal content. This leads to *hallucination* — confidently wrong answers — which is especially dangerous in a business context where accuracy matters.

**Third**, there is usually no quantitative way to measure whether a Q&A system is actually performing well. Most teams rely on gut feel: "it seems fine" — which is clearly not enough.

#### The Solution

Rather than building a complex multi-layer pipeline, I chose to design around **four clearly defined processing flows**, each with a specific responsibility:

- Documents are ingested through **Amazon S3**, buffered through **Amazon SQS** (with Dead Letter Queues for retry safety), then processed by **AWS Lambda** together with **Amazon Textract** to digitize scanned files.

- Extracted content is split into parent/child chunks, embedded via **Amazon Bedrock**, and stored directly in **Amazon DynamoDB** as packed vectors alongside BM25 term-frequency data. A custom Python hybrid search layer (cosine similarity + BM25, fused via Reciprocal Rank Fusion) runs inside Lambda itself — no dedicated search engine needed.

- User queries flow through **Amazon API Gateway** (secured with **Amazon Cognito**), check an **ElastiCache Serverless** cache layer first to avoid redundant Bedrock calls, retrieve relevant context via hybrid search, and generate answers through **Amazon Bedrock (Claude 3)** filtered through **Bedrock Guardrails**.

- The monitoring setup uses **CloudWatch + SNS + AWS Chatbot** to classify and route alerts to Slack by severity. **EventBridge Scheduler** triggers a daily Lambda to run RAGAS metrics (Faithfulness, Answer Relevancy, Context Precision) against recent conversations.

#### Why Design It This Way?

One key decision was **not using a dedicated search engine** (like OpenSearch Serverless). Storing vectors and BM25 data directly in DynamoDB eliminates an always-on baseline cost and keeps the retrieval layer effectively pay-per-use — consistent with the serverless philosophy of the entire stack.

---

### 3. Solution Architecture

All infrastructure is managed through Terraform so every change is reviewable via Pull Request and the entire stack can be rebuilt from scratch at any time.

![RAG Knowledge Assistant System Architecture Overview](/images/5-Workshop/5.1-Workshop-overview/aws-new.drawio.png)

#### AWS Services Used

| Service | Role |
|---|---|
| **AWS Lambda** | Runs document processing, chat engine, and RAGAS evaluation logic (Python 3.12) |
| **Amazon S3** | Stores raw uploaded documents and RAGAS evaluation results |
| **Amazon SQS** | Buffers document processing events with a Dead Letter Queue for retry handling |
| **Amazon Textract** | Performs OCR on scanned files and images |
| **Amazon Bedrock** | Generates embeddings (Titan/Cohere) and answers (Claude 3), enforced through Guardrails |
| **Amazon DynamoDB** | Stores document chunks, packed vectors, BM25 data, chat history, and user feedback |
| **Amazon API Gateway** | Exposes the chat, upload, and status endpoints |
| **Amazon Cognito** | Authenticates end users before granting API access |
| **Amazon ElastiCache Serverless** | Caches recent Q&A pairs to reduce latency and Bedrock cost |
| **Amazon CloudWatch** | Collects logs/metrics, custom dashboards, and alarms |
| **Amazon SNS + AWS Chatbot** | Routes severity-classified alerts to Slack |
| **Amazon EventBridge Scheduler** | Triggers the daily RAGAS evaluation job |
| **Terraform (HCP Terraform)** | Manages all infrastructure as code with remote state |

#### Four Core Processing Flows

**Flow 1 — Data Ingestion**
S3 receives uploads → S3 Event triggers SQS → Lambda (Document Processor) extracts text (Textract for scanned files) → splits into parent/child chunks → generates embeddings + BM25 data → stores in DynamoDB.

**Flow 2 — Realtime Q&A**
API Gateway (behind Cognito) → Chat Engine Lambda checks cache → hybrid search against DynamoDB → Bedrock generates answer through Guardrails → writes conversation to DynamoDB.

**Flow 3 — Monitoring & Alert**
CloudWatch Alarms watch Lambda errors, API 5xx rates, DLQ depth, and Bedrock throttling → publish to severity-based SNS topics → route to Slack via AWS Chatbot.

**Flow 4 — RAG Evaluation**
EventBridge Scheduler → Lambda samples recent Q&A pairs → scores with RAGAS metrics → stores results in S3 → publishes summary scores to CloudWatch.

---

### 4. Technical Implementation

#### Implementation Phases

The project follows a **5-week build cycle** after topic finalization, each phase building directly on the previous one:

1. **Research & Architecture Design** — Finalize the topic, evaluate Serverless vs. managed alternatives (e.g., Bedrock Knowledge Bases), and produce a proposal with architecture and data-flow diagrams.

2. **Environment Setup** — Prepare the Terraform/IaC project structure, request Amazon Bedrock model access (Claude 3, Titan Embeddings), and configure the development environment.

3. **Core Flow Development** — Implement Flow 1 (Data Ingestion) first, then Flow 2 (Realtime Q&A with Semantic Cache), validating each with hands-on testing before moving to the next.

4. **Observability & Quality** — Implement Flow 3 (Monitoring & Alerting) and Flow 4 (RAGAS Evaluation) so the system can detect its own issues and measure its own answer quality.

5. **Hardening & Delivery** — Tune retrieval parameters based on RAGAS findings, run load tests, audit IAM permissions, refactor Terraform into modules, finalize documentation, and deliver the live demo.

#### Technical Requirements

- **Account & Region**: AWS account in **us-east-1 (N. Virginia)** — the region with full support for the required Bedrock models — plus an HCP Terraform account for remote state management.
- **Tooling**: Terraform 1.5+, AWS CLI v2, Python 3.12 (Lambda runtime), Git, and a code editor.
- **Permissions**: A deployment IAM policy scoped to exactly the service groups used — least privilege throughout.
- **CI/CD**: GitHub Actions for checks/plan on every PR, with `terraform apply` gated behind a manual, reviewed trigger — not auto-applied on merge.

---

### 5. Timeline & Milestones

_5-week timeline, following 2 weeks of AWS foundations training_

| Week | Milestone |
|---|---|
| **Week 1** | Topic ideation, finalize proposal, design architecture diagrams, prepare development environment |
| **Week 2** | Deliver Flow 1 — Data Ingestion end-to-end (S3 → SQS → Lambda → OCR → embeddings) |
| **Week 3** | Deliver Flow 2 — Realtime Q&A with authenticated API, semantic cache, and Guardrails |
| **Week 4** | Deliver Flow 3 — Monitoring & Alerting, and Flow 4 — automated RAGAS evaluation |
| **Week 5** | Tune retrieval quality, load test, harden IAM, refactor IaC, finalize documentation, live demo |

---

### 6. Budget Estimation

By storing vectors and BM25 data in DynamoDB (pay-per-request) instead of provisioning a dedicated search engine, the main always-on baseline cost driver is **Amazon ElastiCache Serverless** minimum provisioned capacity, with Bedrock invocations metered on top.

**Estimated running cost while the stack is deployed**: ~**$2.5/day**

ElastiCache Serverless is the primary cost driver; Lambda, S3, SQS, DynamoDB, and API Gateway usage-based costs add comparatively little at this scale.

**Cost controls in place:**

- Using DynamoDB instead of a dedicated search engine for vector/BM25 storage avoids an additional always-on cost for the retrieval layer.
- Exact-match caching reduces repeated Bedrock invocation costs for identical questions.
- Dedicated teardown/rebuild scripts destroy the full stack (with a documents backup) when not in use — no fixed monthly baseline.
- An AWS Budget alert monitors monthly spend independently of the main stack's lifecycle.

---

### 7. Risk Assessment

Every technical project carries risk. What matters is identifying them early and having a clear response plan rather than scrambling when things go wrong.

#### Risk Matrix

| Risk | Impact | Probability |
|---|---|---|
| Delayed Amazon Bedrock model access approval | Medium | Medium |
| Low retrieval/answer quality (hallucination, poor context match) | High | Medium |
| Cost overrun from ElastiCache Serverless always-on baseline | Medium | Low |
| Misconfigured IAM permissions between services | High | Low |
| Lambda concurrency limits under higher load | Medium | Low |

#### Mitigation Strategies

- **Bedrock access**: Submit model access requests as early as possible in the setup phase — before development work depends on them.
- **Retrieval quality**: Establish the RAGAS evaluation loop early so quality regressions are caught with actual metrics, not guesswork, and drive concrete tuning (chunk size, hybrid search weighting).
- **Cost**: AWS Budget alerts plus scripted teardown when the stack is idle.
- **IAM**: Apply least-privilege policies from day one and run a dedicated permissions audit before final delivery.
- **Concurrency**: Load-test before the final demo to surface limits early and document scaling options.

#### Contingency Plans

- If Bedrock access is delayed → continue infrastructure and pipeline development using mocked embedding/generation calls, then integrate real model calls once access is granted.
- If costs approach budget thresholds → immediately run the teardown script to destroy non-critical resources.
- If retrieval quality cannot be sufficiently improved within the timeline → document the gap explicitly and scope it as a follow-up item rather than silently shipping a system that underperforms.

---

### 8. Expected Outcomes

#### Technical Improvements

- Automated document ingestion and OCR replace manual document handling entirely.
- Cached responses return in under a second for repeated questions, versus several seconds for a fresh Bedrock call.
- Automated daily RAGAS evaluation replaces subjective quality checks with quantitative, trackable scores.
- Real-time, severity-classified alerting shortens the time to detect and respond to operational issues.

#### Long-term Value

This project is more than a technical exercise. By the end, I expect to have:

- A **reusable, documented reference architecture** for Serverless GenAI systems on AWS.
- Hands-on experience with **Infrastructure as Code (Terraform)** and event-driven design.
- A foundation that can be extended toward broader enterprise knowledge management use cases in the future.
