---
layout: post
title: "ReceiptInbox: Serverless Receipt Parser & Smart Alerts"
date: 2025-11-13
category: "Cloud Computing · Backend Architecture · AWS"
summary: "Backend lead on a serverless AWS receipt processing pipeline — FastAPI on Lambda, presigned S3 uploads, SQS async fan-out, DynamoDB + PostgreSQL hybrid storage, and ML-powered categorization."
description: "Backend lead on a serverless AWS receipt processing pipeline — FastAPI on Lambda, SQS async fan-out, DynamoDB + PostgreSQL hybrid storage, and ML-powered categorization."
tags: [Python, FastAPI, AWS Lambda, SQS, Textract, PostgreSQL, DynamoDB, S3, JWT, AWS SAM]
links:
  - label: GitHub
    url: https://github.com/kanglinde/ee547-final-project
    external: true
  - label: Project Proposal (PDF)
    url: /assets/pdf/EE547_Proposal.pdf
    external: false
---

This project was completed for EE 547 (Applied and Cloud Computing for Electrical Engineers) at USC, in collaboration with Harry Kang and Vivin Thiyagarajan. I served as Backend Lead, responsible for the serverless infrastructure, the API layer, and the asynchronous processing pipeline. Harry led the frontend; Vivin handled the React UI and dashboard integration.

## Background: Why Serverless for Receipt Processing

Receipt processing is an inherently bursty workload. A user might upload nothing for a week, then photograph fifteen receipts after a business trip. A traditional server idling between bursts wastes money and resources. AWS Lambda's pricing model — pay per invocation, scale to zero at rest — is a natural architectural fit [1].

The serverless pattern also aligns well with the processing characteristics of OCR and ML inference. AWS Textract, Amazon's managed document analysis service, uses deep learning models trained on millions of documents to extract structured text and form data from images and PDFs [2]. Amazon Bedrock provides access to foundation models via API for categorization and reasoning tasks [3]. Both services are inherently asynchronous: a complex PDF might take several seconds to process, far exceeding acceptable synchronous API response times.

For queue-based decoupling of asynchronous workloads, Amazon SQS (Simple Queue Service) provides at-least-once delivery with automatic retry and dead-letter queue support for failed messages [4]. This is the standard AWS pattern for decoupling producers from consumers in event-driven architectures.

Hybrid database design — using both relational and NoSQL stores for different access patterns — is a well-established practice in cloud-native systems. PostgreSQL provides ACID transactions and relational joins for user account data; DynamoDB provides single-digit millisecond key-value access for high-throughput metadata queries [5].

<figure>
  <img src="/assets/img/posts/receiptinbox_architecture.jpg" alt="ReceiptInbox system architecture">
  <figcaption>High-level system architecture: React frontend communicates over HTTP to API Gateway, routing to Lambda 1 (FastAPI) for API handling. Lambda 1 pushes jobs to SQS; Lambda 2 (ML Processor) processes jobs via Rekognition, Bedrock, and SNS. Data flows to DynamoDB and S3.</figcaption>
</figure>

## What We Built

**API layer.** I designed and implemented the REST API using FastAPI deployed as a serverless function on AWS Lambda via API Gateway. FastAPI was chosen over Flask for its native async support and automatic request validation via Pydantic, which reduced boilerplate for input parsing and error handling. The API endpoints cover authentication (signup/login returning JWT), receipt upload initiation, job status polling, and dashboard data retrieval.

**Upload pipeline.** A key design decision was the presigned S3 URL pattern. Rather than routing file uploads through the Lambda function — which would hit the 10MB API Gateway payload limit and waste Lambda execution time on data transfer — the API generates a temporary, cryptographically signed S3 URL granting write permission to a specific key for a short window. The frontend uploads directly to S3, and the completed upload triggers an S3 event notification that kicks off processing.

**Async processing with SQS.** When a receipt lands in S3, Lambda 1 immediately enqueues a processing job on SQS and returns a job ID to the user. Lambda 2 (the ML Processor Worker) polls SQS, runs Textract for OCR text extraction, passes the extracted text to Bedrock for category inference, executes custom anomaly detection logic, updates DynamoDB with the result, and triggers SNS for alert delivery. The SQS Dead Letter Queue catches messages that fail after the configured retry count.

<figure>
  <img src="/assets/img/posts/receiptinbox_backend_detail.jpg" alt="Detailed backend data flow">
  <figcaption>Detailed backend data flow showing Lambda 1 (FastAPI, Python 3.10) generating presigned URLs and pushing to SQS Job Queue. Lambda 2 (ML Processor Worker) runs Rekognition text detection, Bedrock categorization, custom anomaly logic, and updates DynamoDB Users and Receipts tables before publishing alerts via SNS.</figcaption>
</figure>

**Hybrid data model.** I designed the data schema across two storage systems. PostgreSQL on EC2 (hardened via pg_hba.conf and Security Group rules) stores user accounts, job records, and alert rules — data that requires transactional integrity and relational queries. DynamoDB stores receipt metadata and processing status — data accessed by receipt ID at high frequency where predictable latency and automatic scaling matter more than relational joins.

**Authentication.** JWT-based stateless authentication was implemented from scratch rather than delegating to AWS Cognito — an intentional learning choice. Users register with hashed passwords stored in PostgreSQL; login returns a signed JWT with an expiry claim; subsequent requests pass the token in the Authorization header, verified independently by each Lambda invocation without shared session state.

**Infrastructure as code.** The entire stack was defined using AWS SAM (Serverless Application Model) templates, enabling reproducible one-command deployment of all Lambda functions, API Gateway configuration, SQS queues, DynamoDB tables, IAM roles, and environment variable injection from Secrets Manager.

<figure>
  <img src="/assets/img/posts/receiptinbox_userflow.jpg" alt="User flow diagram">
  <figcaption>User flow: registration and login with failure loops, followed by access to the Dashboard/Upload page. All state transitions are backed by JWT validation on the API layer.</figcaption>
</figure>

## References

[1] Amazon Web Services, "AWS Lambda Developer Guide," AWS Documentation, 2024. https://docs.aws.amazon.com/lambda/

[2] Amazon Web Services, "Amazon Textract Developer Guide," AWS Documentation, 2024. https://docs.aws.amazon.com/textract/

[3] Amazon Web Services, "Amazon Bedrock User Guide," AWS Documentation, 2024. https://docs.aws.amazon.com/bedrock/

[4] Amazon Web Services, "Amazon SQS Developer Guide," AWS Documentation, 2024. https://docs.aws.amazon.com/sqs/

[5] Amazon Web Services, "Amazon DynamoDB Developer Guide," AWS Documentation, 2024. https://docs.aws.amazon.com/dynamodb/

[6] A. Kleppmann, *Designing Data-Intensive Applications*. O'Reilly Media, 2017. *(Reference for hybrid database design patterns)*
