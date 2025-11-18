---
layout: post # Do not change
category: [cloudcomputing] # Project categories
title: "ReceiptInbox - Serverless Receipt Parser & Smart Alerts" # Project title
date: 2025-11-13 20:30:00 -0800 # Date
author: LilyHua # The author nick we created
prevPart: _posts/2025-11-13-intent-classification-lstm.md
nextPart: _posts/2025-11-14-personal-portfolio-ci-cd.md
og_description: "A cloud-native application using AWS Lambda, SQS, Textract, and ML for automated receipt processing and spend analysis." # Project description
---

Manually transcribing receipts from photos, PDFs, and emails into a spreadsheet is tedious, error-prone, and often skipped. ReceiptInbox is a cloud-hosted web application built to automate this entire process.

A user uploads a receipt, and the system automatically extracts the merchant, date, and total. It then leverages machine learning to infer the spending category (e.g., "Groceries," "Travel") and stores everything in a searchable dashboard. The system also runs anomaly detection rules (like "duplicate charge" or "unusually high total") and notifies the user.

### Core Features
* [cite_start]**Multi-Format Upload:** Users can upload receipts from a browser, including photos, screenshots, and PDFs[cite: 5, 11].
* **Automatic OCR & Parsing:** Utilizes AWS Textract for automatic text and line-item extraction.
* [cite_start]**AI-Powered Categorization:** Employs ML models (Comprehend/Bedrock) to intelligently suggest spending categories.
* [cite_start]**Smart Alerts:** Pushes email notifications (via SES) for detected spending anomalies.
* **Searchable Dashboard:** Provides a full-text search and detailed view of all processed receipts.

---

### My Role: Backend Architecture & ML Integration

As a backend and machine learning engineer on this team, my primary responsibility was **designing the asynchronous processing pipeline and integrating the AWS machine learning services**.

#### Why Asynchronous?
Receipt processing, especially OCR (Textract) and ML inference (Comprehend/Bedrock), is a slow and "bursty" operation. Running it synchronously would lead to API timeouts. We therefore adopted a queue-based, serverless architecture, which is fundamental to this cloud computing project.

#### System Flow (Queue Fan-out):
The pipeline I designed follows this event-driven flow:

1.  **Upload & Trigger:** A user uploads a file to **Amazon S3** via a pre-signed URL.
2.  **S3 Event -> SQS (Queue 1):** The S3 "put" event triggers a message containing the file information onto our first **SQS Queue (`ocr-queue`)**.
3.  **OCR Worker (Lambda):** An **AWS Lambda** worker polls this queue. Upon receiving a message, it invokes **AWS Textract** to perform OCR. It then writes the raw JSON text output back to S3.
4.  **-> SQS (Queue 2):** After the OCR job succeeds, the worker places a new "parsing" job onto the second **SQS Queue (`parse-queue`)**.
5.  **Parse & Categorize Worker (Lambda):** A second Lambda worker polls this queue. It retrieves the OCR text, calls **Amazon Bedrock/Comprehend** for ML-based categorization, and writes the final, structured data (merchant, total, category, etc.) into our **PostgreSQL on RDS** database.

This "cascading queues" architecture decouples the OCR and ML inference steps, dramatically improving system reliability, fault tolerance (via retries and DLQs), and maintainability.

### Technology Stack
* [cite_start]**Language:** **Python** (for all backend AWS Lambda functions), JavaScript (React)
* **AWS Services:** Lambda, SQS, S3, API Gateway, Textract, Comprehend/Bedrock, RDS (PostgreSQL), SES, IAM
* [cite_start]**Frameworks:** FastAPI, SQLAlchemy, Pydantic (for backend API)

---

### Project Links
{% assign github_url1 = 'https://github.com/kanglinde/ee547-final-project.git' %}
<div class='sx-button'>
  <a href='{{ github_url1 }}' class='sx-button__content red' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/icons8-github-500.svg'/> View on GitHub
  </a>
</div>

<div class='sx-button'>
  <a href='/assets/pdf/EE547_Proposal.pdf' class='sx-button__content blue' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/resume.svg'/> Read Project Proposal (PDF)
  </a>
</div>

{% assign github_url = 'https://github.com/Update-For-Integrated-Business-AI/CORU' %}
<div class='sx-button'>
  <a href='{{ github_url }}' class='sx-button__content green' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/resume.svg'/> View CORU Database
  </a>
</div>