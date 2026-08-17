---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# From Theory to Execution: The Data Engineering Challenge in a Real-World GenAI/RAG Project

Hello everyone,

In my previous post, I shared my perspective on what a Data Engineer should prioritize learning on AWS to optimize time and be job-ready.

Today, I would like to share in more detail how I applied those AWS mindsets and services directly into my practical project. Although the main problem of the project falls under GenAI / RAG (Retrieval-Augmented Generation), when I started implementing the system architecture, I realized: up to 80% of the system's power and stability lies in Data Engineering.

Below is the full picture of how I applied cloud data processing techniques in this project.

## 1. Data Ingestion & Event-Driven Pipeline

When processing input documents, relying on traditional direct function calls easily creates bottlenecks or overloads the system. Therefore, I built an Event-Driven Architecture:

- **Automatic Trigger (S3 Event Trigger):** Whenever a user or Admin uploads a new document (PDF/TXT/Scan) to Amazon S3 Raw Documents, an S3 Event immediately triggers the processing flow without requiring manual polling.
- **Buffer Queue & System Decoupling (Amazon SQS):** To prevent the system from being overwhelmed when a large volume of files is uploaded simultaneously, data flows through Amazon S3 Event → Amazon SQS (Buffer Queue). Using SQS acts as a load-bearing buffer, combining automatic retries and a Dead Letter Queue (DLQ) to catch failed messages, guaranteeing zero data loss.

## 2. ETL & Unstructured Data Processing

Unstructured text data needs to undergo a strict ETL process before it can serve AI models:

- **Data Extraction (OCR):** AWS Lambda receives messages from SQS and automatically invokes Amazon Textract to perform OCR, accurately extracting text from complex scanned files or PDFs.
- **Structuring & Vector Storage (Chunking & Vectorization):** Processed data is chunked following a Parent-Child model, then sent to the Embedding API for transformation into vectors. All of this structured information is stored in Amazon DynamoDB—optimized for Hybrid Search queries (combining Cosine Similarity and BM25 using the RRF algorithm) to achieve maximum search accuracy.

## 3. Low-Latency Data Retrieval & Caching

A core Data Engineering challenge in Web/App applications is latency and cost:

- **Exact-match cache (not a true semantic cache):** I use Amazon ElastiCache Serverless as a question → answer cache with TTL. The cache key is a hash of the question **after normalisation** (lowercase, strip punctuation, collapse whitespace). An **identical** question (after that step) hits cache and skips the LLM. ElastiCache Serverless has **no** RediSearch/vector module, so it does **not** match “similar meaning” queries. The UI may still label the step “Semantic cache”; the implementation is exact-match — I document that honestly instead of overselling.
- **State Management & Feedback (Transaction Store):** The entire chat history and user feedback store are persisted in DynamoDB—a NoSQL database delivering ultra-fast write speeds with millisecond latency.

## 4. Automated Batch Processing & Continuous Evaluation (Partial)

Beyond real-time flows, a proper data system needs a batch path for quality evaluation. **This part is explicitly Partial** — not claimed as production-live:

- **Terraform design is in place:** EventBridge Scheduler would invoke an evaluation Lambda (packaged as a **container image** on ECR, unlike the two zip Lambdas for ingestion/chat). Results are meant to land in S3 Evaluation Results, with RAGAS metrics (Faithfulness, Relevancy, Precision, Recall) on CloudWatch. A **two-phase gate** applies: phase 1 creates an empty ECR repo + IAM only; Lambda/schedule/alarm exist only after the image is pushed and the flag is turned on.
- **Not fully running on the account yet:** `evaluation_runner.py` is still a **skeleton**; with `evaluation_image_pushed = false` the Lambda / EventBridge schedule / `ragas-faithfulness-low` alarm are **not live**. The Stream 3 dashboard RAGAS widget is **intentionally empty** until that job has run at least once. I keep this Partial and do not mix it with Streams 1–3, which are E2E-tested.

## 5. Data Observability & Monitoring

Finally, data running within a Cloud system must be observable to catch issues promptly:

- **Centralized Logs & Metrics:** Amazon CloudWatch collects logs and live pipeline metrics: DLQ depth, API Gateway 5xx rate, cache hit/miss (EMF from chat-engine). RAGAS custom metrics (Faithfulness, Relevancy, Precision) are **wired on the dashboard** but **have no values** until the evaluation stream in section 4 actually runs — consistent with Partial.
- **Severity-based alerting:** Warning (email via SNS `alerts-info`) and Critical (SNS `alerts-critical`, Slack via AWS Chatbot when configured). PagerDuty is sketched as optional and **off by default**.

## Three Data Engineering Highlights I Learned From the Project

If you are preparing for a thesis or personal project, here are 3 Data Engineering mindsets I found most valuable:

- **Decoupled Architecture:** The Data Ingestion and Query Serving phases operate independently via SQS queues and DynamoDB. Even if an Admin uploads thousands of PDF files at once, the end-user chat experience remains completely smooth and unaffected.
- **Asynchronous Serverless Processing:** Combining S3 Events + SQS + Lambda makes the system fully asynchronous. Compute resources only spin up when data is flowing through, achieving 100% cloud operation cost optimization.
- **Query Optimization:** Instead of calling the LLM every time, I combine an **exact-match cache** (ElastiCache, hash of the normalised question) with Hybrid Search (BM25 + vectors, RRF). Knowing Serverless Redis has no RediSearch and picking the right cache type is part of a Data Engineer’s job — not just drawing a prettier “semantic” box.

## Conclusion

This project proved one thing to me: being a Data Engineer is not just about writing SQL queries or running Spark scripts, but about the ability to design reliable Cloud infrastructure, automate data flows, and optimize operational costs for the system.

I hope this practical project perspective gives you a clearer picture of how AWS services like S3, SQS, Lambda, DynamoDB, and ElastiCache work together in real-world scenarios.

## Link

<https://www.facebook.com/groups/awsstudygroupfcj/permalink/2240430060055287/#>
