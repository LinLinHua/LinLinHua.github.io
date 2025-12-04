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



[Image of serverless architecture diagram]


### Core Features
* **Multi-Format Upload:** Users can upload receipts from a browser, including photos, screenshots, and PDFs.
* **Automatic OCR & Parsing:** Utilizes AWS Textract for automatic text and line-item extraction.
* **AI-Powered Categorization:** Employs ML models to intelligently suggest spending categories.
* **Smart Alerts:** Pushes email notifications (via SES) for detected spending anomalies.
* **Searchable Dashboard:** Provides a full-text search and detailed view of all processed receipts.

---

### My Role: Backend Architecture & Database Design

As the **Backend Lead** for this project, I was responsible for architecting the serverless infrastructure, implementing the secure API layer, and designing the hybrid data model.

#### 1. Serverless API Architecture (FastAPI + Lambda)
I designed and developed the core RESTful API using **FastAPI**, deployed as a serverless function on **AWS Lambda** via **API Gateway**. This architecture allows the system to scale automatically to zero when idle, minimizing costs.

#### 2. Secure Authentication & Authorization
I implemented a custom authentication system from scratch:
* **User Management:** Designed the user schema and deployed a self-managed **PostgreSQL** database on an **Amazon EC2** instance (hardening security via `pg_hba.conf` and Security Groups).
* **Session Security:** Implemented **JWT (JSON Web Tokens)** for stateless authentication, ensuring secure communication between the React frontend and backend.

#### 3. High-Performance Data & Upload Pipeline
To optimize performance for large file uploads and "bursty" traffic:
* **Presigned S3 URLs:** I built a workflow where the API generates temporary, secure URLs, allowing the frontend to upload receipts directly to **Amazon S3**. This bypasses the API server bottleneck entirely.
* **Hybrid Database Model:** I utilized **PostgreSQL** for relational user data and **Amazon DynamoDB** for high-speed, key-value access to receipt metadata and processing status.
* **Async Dispatch:** My API endpoints integrate with **Amazon SQS**, dispatching processing tasks to background workers to decouple the user interface from heavy ML operations.

### Technology Stack
* **Language:** **Python** (FastAPI, SQLAlchemy, Boto3)
* **Compute & API:** AWS Lambda, Amazon API Gateway
* **Data & Storage:** PostgreSQL on EC2, Amazon DynamoDB, Amazon S3
* **Infrastructure:** AWS SAM (Serverless Application Model), AWS IAM, Secrets Manager

---

### Project Links
{% assign github_url1 = 'https://github.com/LinLinHua/ReceiptInbox---Serverless-Receipt-Parser-Smart-Alerts.git' %}
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