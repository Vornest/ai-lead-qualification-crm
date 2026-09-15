# System Architecture

## Overview

The AI Lead Qualification & CRM Automation System is an n8n-based workflow that automates lead intake, validation, AI qualification, duplicate detection, and CRM storage.

The workflow combines:

- n8n for workflow orchestration
- Google Gemini for AI lead qualification
- Supabase/PostgreSQL for persistent CRM storage
- REST webhook for API access

## High-Level Architecture

```text
                        Client / Postman
                               |
                               v
                        +--------------+
                        |   Webhook    |
                        +------+-------+
                               |
                               v
                     +-------------------+
                     | Required          |
                     | Validation        |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | Budget            |
                     | Validation       |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | Email Validation |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | Prepare Lead Data|
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | Google Gemini     |
                     | Lead Qualification|
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | Structured Output |
                     | Parser            |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | Merge AI Result   |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | Search Lead      |
                     +---------+---------+
                               |
                               v
                         +-----------+
                         | IF Exists?|
                         +-----+-----+
                              / \
                            YES  NO
                             |    |
                             v    v
                       +------+  +-------------+
                       | 409  |  | Create Row  |
                       |      |  +------+------+
                       +------+         |
                                        v
                                  +-----------+
                                  | 201       |
                                  | Created   |
                                  +-----------+
```

## Workflow Components

### 1. Webhook

The Webhook node receives an HTTP `POST` request containing the incoming lead data.

Expected input structure:

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}
```

The Webhook acts as the entry point of the automation.

---

### 2. Required Validation

The `Required Validation` node checks that the following fields exist and contain values:

```text
customer_name
customer_email
company
requirement
```

Invalid input is routed to the `Required False` response node.

Response:

```text
HTTP 400 Bad Request
```

---

### 3. Budget Validation

The `Budget Validation` node checks three conditions:

```text
budget exists
budget is a Number
budget > 0
```

The validation expression evaluates the original webhook payload before data normalization.

Example:

```text
2500     → Valid
"2500"   → Invalid
0        → Invalid
-500     → Invalid
```

Invalid input is routed to the `Budget False` response node.

Response:

```text
HTTP 400 Bad Request
```

---

### 4. Email Validation

The `Email Validate` node checks the incoming `customer_email` against the workflow email validation pattern.

Invalid email input is routed to the `Email False` response node.

Response:

```text
HTTP 400 Bad Request
```

---

### 5. Prepare Lead Data

The `Prepare Lead Data` node normalizes the validated webhook data into a consistent structure used by downstream nodes.

The normalized fields are:

```text
customer_name
customer_email
company
project_requirements
budget
```

This creates a predictable input structure for the AI qualification stage.

---

### 6. Google Gemini

The `Basic LLM Chain` uses the Google Gemini chat model.

Gemini receives the normalized lead information and is instructed to generate:

```text
lead_score
priority
reason
recommended_action
```

The workflow currently uses these budget-based scoring rules:

```text
>= 3000
→ 90

1500 - 2999
→ 70

500 - 1499
→ 50

< 500
→ 30
```

Priority is then assigned according to the score:

```text
80 - 100
→ HOT

50 - 79
→ WARM

0 - 49
→ COLD
```

---

### 7. Structured Output Parser

The `Structured Output Parser` ensures that Gemini returns data in a predictable JSON structure.

Expected schema:

```json
{
  "lead_score": 90,
  "priority": "HOT",
  "reason": "string",
  "recommended_action": "string"
}
```

Required fields:

```text
lead_score
priority
reason
recommended_action
```

This provides structured data for the following workflow stages.

---

### 8. Merge AI Result

The `Merge AI Result` node combines the original lead data with the AI-generated qualification result.

The combined structure contains:

```text
customer_name
customer_email
company
project_requirements
budget
lead_score
priority
reason
recommended_action
```

Example:

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "project_requirements": "Need AI automation for our sales process",
  "budget": 3500,
  "lead_score": 90,
  "priority": "HOT",
  "reason": "The lead has a high budget and a clear automation requirement.",
  "recommended_action": "Schedule a discovery call."
}
```

---

### 9. Search Lead

The `Search Lead` node queries the Supabase `leads` table using:

```text
customer_email
```

The objective is to determine whether the lead already exists in the CRM database.

The node is configured to return data when a matching lead is found.

---

### 10. IF Exists?

The `IF Exists?` node determines the next action based on whether a lead record was found.

#### TRUE branch

If an existing lead is found:

```text
Duplicate
    ↓
409 Conflict
```

#### FALSE branch

If no existing lead is found:

```text
Create a row
    ↓
Success
    ↓
201 Created
```

---

### 11. Create Row

The `Create a row` node inserts the qualified lead into the Supabase `leads` table.

Stored fields include:

```text
customer_name
customer_email
company
budget
lead_score
priority
reason
recommended_action
requirement
```

---

### 12. Success Response

The `Success` node returns a structured response after successful database insertion.

Response:

```text
HTTP 201 Created
```

Example:

```json
{
  "success": true,
  "data": {
    "id": 28,
    "customer_name": "Daniel",
    "customer_email": "daniel@example.com",
    "company": "Nova Systems",
    "budget": 3500,
    "requirement": "Need AI automation for our sales process",
    "lead_score": 90,
    "priority": "HOT",
    "reason": "The lead has a high budget and a clear automation requirement.",
    "recommended_action": "Schedule a discovery call."
  }
}
```

---

## Error Handling

The workflow contains dedicated response nodes for invalid requests.

### Required Validation Failure

```text
Required Validation
        |
        v
Required False
        |
        v
HTTP 400
```

### Budget Validation Failure

```text
Budget Validation
        |
        v
Budget False
        |
        v
HTTP 400
```

### Email Validation Failure

```text
Email Validate
        |
        v
Email False
        |
        v
HTTP 400
```

### Duplicate Lead

```text
IF Exists?
    |
   TRUE
    |
    v
Duplicate
    |
    v
HTTP 409
```

---

## Database Architecture

The system uses Supabase/PostgreSQL as the CRM storage layer.

Main table:

```text
leads
```

Relevant fields:

```text
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
```

### Database-Level Duplicate Protection

The `customer_email` field is protected by a UNIQUE constraint.

Constraint:

```text
leads_customer_email_unique
```

This prevents duplicate records at the database layer.

---

## Defense in Depth

Duplicate protection is intentionally implemented in two layers:

```text
Application Layer
        |
        v
n8n Search Lead
        |
        v
Duplicate Detection

        +

Database Layer
        |
        v
UNIQUE(customer_email)
```

This means the workflow checks for duplicates before insertion while the database independently enforces uniqueness.

---

## End-to-End Data Flow

```text
HTTP POST Request
        |
        v
Webhook
        |
        v
Input Validation
        |
        v
Data Preparation
        |
        v
Google Gemini
        |
        v
Structured AI Output
        |
        v
Merge Lead + AI Data
        |
        v
Duplicate Check
        |
        +----------------+
        |                |
     Exists          Not Exists
        |                |
        v                v
   409 Conflict      Create Row
                         |
                         v
                    201 Created
```

## Design Principles

### Validate Before Processing

Invalid input is rejected before AI qualification and database creation.

### Structured AI Output

Gemini output is validated through a structured output parser before downstream processing.

### Defense in Depth

Duplicate protection exists in both the workflow and database.

### Separation of Responsibilities

Each node performs a specific role:

```text
Validation
    ↓
Normalization
    ↓
AI Processing
    ↓
Structured Output
    ↓
Duplicate Detection
    ↓
Database
    ↓
API Response
```
