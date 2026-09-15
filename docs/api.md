# API Documentation

## Overview

This API provides an endpoint for automated lead qualification and CRM storage.

The workflow receives a lead through an HTTP POST request, validates the input, uses Google Gemini to generate a lead score and priority, checks for duplicate leads, stores valid leads in Supabase/PostgreSQL, and returns a structured JSON response.

## Endpoint

```text
POST /leads
```

### Content Type

```text
application/json
```

## Request Body

The API expects the following fields:

| Field | Type | Required | Description |
|---|---|---|---|
| `customer_name` | string | Yes | Customer or contact name |
| `customer_email` | string | Yes | Customer email address |
| `company` | string | Yes | Customer company |
| `requirement` | string | Yes | Customer project requirement |
| `budget` | number | Yes | Customer budget |

### Example Request

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}
```

## Validation Rules

Validation is performed before AI processing.

### Required Fields

The following fields must exist and must not be empty:

```text
customer_name
customer_email
company
requirement
```

`budget` is validated separately.

### Email Validation

The email address must match the workflow email validation pattern.

Example of an invalid email:

```text
daniel123
```

Expected response:

```text
HTTP 400 Bad Request
```

### Budget Validation

The budget must satisfy all of the following requirements:

```text
budget must exist
budget must be a Number
budget must be greater than 0
```

Examples:

| Input | Result |
|---|---|
| `3500` | Valid |
| `"3500"` | Invalid |
| `0` | Invalid |
| `-500` | Invalid |

## AI Qualification

After successful validation, the lead is passed to Google Gemini.

Gemini generates:

```text
lead_score
priority
reason
recommended_action
```

### Example AI Result

```json
{
  "lead_score": 90,
  "priority": "HOT",
  "reason": "The lead has a high budget and a clear automation requirement.",
  "recommended_action": "Schedule a discovery call."
}
```

## Scoring Rules

The current workflow uses the following budget-based scoring rules:

| Budget | Score |
|---|---:|
| `>= 3000` | 90 |
| `1500 - 2999` | 70 |
| `500 - 1499` | 50 |
| `< 500` | 30 |

### Priority Rules

| Score | Priority |
|---|---|
| `80 - 100` | HOT |
| `50 - 79` | WARM |
| `0 - 49` | COLD |

## Successful Response

When the lead passes validation, does not already exist, and is successfully stored in the database, the API returns:

```text
HTTP 201 Created
```

### Example

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

## Validation Error

When required input validation fails:

```text
HTTP 400 Bad Request
```

### Example

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "customer_name, company, and requirement are required"
  }
}
```

## Invalid Email Response

When the email format is invalid:

```text
HTTP 400 Bad Request
```

```json
{
  "success": false,
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Invalid email format"
  }
}
```

## Invalid Budget Response

When the budget is missing, is not a Number, is zero, or is negative:

```text
HTTP 400 Bad Request
```

```json
{
  "success": false,
  "error": {
    "code": "INVALID_BUDGET",
    "message": "budget must be a number greater than 0"
  }
}
```

## Duplicate Lead Response

The workflow searches the database using `customer_email` before creating a new record.

When a matching lead already exists:

```text
HTTP 409 Conflict
```

```json
{
  "success": false,
  "error": {
    "code": "DUPLICATE_LEAD",
    "message": "Lead already exists"
  }
}
```

## Duplicate Protection

Duplicate protection is implemented at two levels.

### Application Level

n8n searches the `leads` table using the customer's email address.

If an existing record is found, the workflow returns `409 Conflict` and stops the creation process.

### Database Level

Supabase/PostgreSQL also enforces a UNIQUE constraint on:

```text
customer_email
```

The database therefore provides an independent protection against duplicate records.

## Response Status Codes

| HTTP Status | Meaning |
|---:|---|
| `201` | Lead created successfully |
| `400` | Invalid request |
| `409` | Duplicate lead |

## Processing Flow

```text
POST /leads
    |
    v
Required Field Validation
    |
    v
Budget Validation
    |
    v
Email Validation
    |
    v
Prepare Lead Data
    |
    v
Google Gemini
    |
    v
Structured Output Parser
    |
    v
Merge AI Result
    |
    v
Search Existing Lead
    |
    v
IF Exists?
   / \
 YES  NO
  |    |
  v    v
409   Create Row
       |
       v
      201
```

## Example Scenarios

### Scenario 1 — New Lead

```text
Valid input
    |
    v
AI qualification
    |
    v
No existing record
    |
    v
Create database record
    |
    v
201 Created
```

### Scenario 2 — Duplicate Lead

```text
Valid input
    |
    v
AI qualification
    |
    v
Existing email found
    |
    v
409 Conflict
```

### Scenario 3 — Invalid Request

```text
Invalid input
    |
    v
400 Bad Request
```
