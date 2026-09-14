# System Architecture

## Overview

The AI Lead Qualification & CRM Automation System is an n8n-based workflow that automates the initial processing of incoming sales leads.

The system validates incoming API requests, performs AI-based lead qualification using Google Gemini, checks for existing leads, and stores valid leads in Supabase/PostgreSQL.

---

## High-Level Architecture

```text
Client / API Request
        |
        v
+----------------------+
|      n8n Webhook     |
+----------+-----------+
           |
           v
+----------------------+
|  Required Validation |
+----------+-----------+
           |
           v
+----------------------+
|   Budget Validation  |
+----------+-----------+
           |
           v
+----------------------+
|    Email Validation  |
+----------+-----------+
           |
           v
+----------------------+
|   Prepare Lead Data  |
+----------+-----------+
           |
           v
+----------------------+
|    Google Gemini     |
| Lead Qualification   |
+----------+-----------+
           |
           v
+----------------------+
| Structured Output    |
|      Parser          |
+----------+-----------+
           |
           v
+----------------------+
|    Merge AI Result   |
+----------+-----------+
           |
           v
+----------------------+
|     Search Lead      |
+----------+-----------+
           |
           v
+----------------------+
|     IF Exists?       |
+------+----------+---+
       |          |
     TRUE       FALSE
       |          |
       v          v
  409 Conflict  Create Row
                    |
                    v
              201 Created
Workflow Components
1. Webhook

Receives incoming lead data through an HTTP POST request.

Example:

{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}
2. Required Field Validation

Validates that the following fields are present and not empty:

customer_name
company
requirement

Invalid requests return:

HTTP 400 Bad Request
3. Budget Validation

Validates that:

budget exists
budget is a Number
budget is greater than 0

Examples:

2500     → valid
"2500"   → invalid
0        → invalid
-500     → invalid

Invalid requests return:

HTTP 400 Bad Request
4. Email Validation

Validates the email format before any AI or database processing occurs.

Invalid email requests return:

HTTP 400 Bad Request
5. Prepare Lead Data

Normalizes validated webhook input into a consistent structure for downstream processing.

6. Google Gemini

Google Gemini analyzes the lead and generates:

Lead score
Priority
Reason
Recommended action

Example:

{
  "lead_score": 90,
  "priority": "HOT",
  "reason": "The lead has a high budget and a clear automation requirement.",
  "recommended_action": "Schedule a discovery call."
}
7. Structured Output Parser

The AI response is validated against a predefined JSON schema to ensure that downstream nodes receive predictable structured data.

Expected structure:

{
  "lead_score": 90,
  "priority": "HOT",
  "reason": "string",
  "recommended_action": "string"
}
8. Merge AI Result

Combines the original validated lead data with the AI-generated qualification result.

Example:

{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500,
  "lead_score": 90,
  "priority": "HOT",
  "reason": "The lead has a high budget and a clear automation requirement.",
  "recommended_action": "Schedule a discovery call."
}
9. Search Lead

Searches the Supabase leads table using the customer's email address.

This step is used to identify whether the lead already exists.

10. Duplicate Detection

The workflow checks whether an existing record was found.

If the lead exists:

HTTP 409 Conflict

Response:

{
  "success": false,
  "error": {
    "code": "DUPLICATE_LEAD",
    "message": "Lead already exists"
  }
}
11. Supabase / PostgreSQL

Valid leads are stored in the leads table.

Main fields:

id
customer_name
customer_email
company
requirement
budget
lead_score
priority
reason
recommended_action
created_at

The database also contains a unique constraint on:

customer_email

This provides a second layer of duplicate protection at the database level.

Error Handling

The workflow uses HTTP status codes to communicate the result of each request.

Status Code	Meaning
201	Lead created successfully
400	Invalid input
409	Duplicate lead
404	Resource not found
Data Flow

The complete data flow is:

HTTP Request
    ↓
Validation
    ↓
Normalization
    ↓
AI Qualification
    ↓
Structured Data
    ↓
Duplicate Detection
    ↓
Database
    ↓
API Response
Design Principles

The workflow follows several engineering principles:

Validation Before Processing

Invalid requests are rejected before expensive AI processing or database operations.

Defense in Depth

Duplicate protection exists both in the workflow and at the database level.

Structured AI Output

AI results are forced into a predictable schema before being used by downstream automation.

Consistent API Responses

Success and error responses follow a predictable structure.

Separation of Responsibilities

Each workflow stage has a specific responsibility:

Validation
    ↓
AI Processing
    ↓
Data Transformation
    ↓
Database
    ↓
API Response
