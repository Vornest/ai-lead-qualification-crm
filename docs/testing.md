# Testing Documentation

## Testing Overview

The AI Lead Qualification & CRM Automation System was tested using Postman and Supabase/PostgreSQL.

Testing focused on:

- Input validation
- Email validation
- Budget validation
- AI qualification
- Duplicate detection
- API response handling
- Database integrity

## Test Environment

| Component | Tool |
|---|---|
| Workflow Engine | n8n Cloud |
| AI Model | Google Gemini |
| Database | Supabase / PostgreSQL |
| API Testing | Postman |

---

## Test Case 1 — Valid Lead Creation

### Objective

Verify that a valid lead passes validation, is processed by Gemini, stored in the database, and returns a successful response.

### Request

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}
```

### Expected Result

```text
HTTP 201 Created
```

### Actual Result

```text
PASS
```

The lead was successfully processed and stored in Supabase.

The AI generated a lead score and priority, and the API returned a structured success response.

---

## Test Case 2 — Duplicate Lead

### Objective

Verify that an existing lead cannot be inserted again.

### Request

The same email address is submitted again:

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}
```

### Expected Result

```text
HTTP 409 Conflict
```

### Response

```json
{
  "success": false,
  "error": {
    "code": "DUPLICATE_LEAD",
    "message": "Lead already exists"
  }
}
```

### Actual Result

```text
PASS
```

The workflow detected the existing lead and prevented another record from being created.

---

## Test Case 3 — Invalid Email

### Objective

Verify email validation.

### Request

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel123",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": 3500
}
```

### Expected Result

```text
HTTP 400 Bad Request
```

### Response

```json
{
  "success": false,
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Invalid email format"
  }
}
```

### Actual Result

```text
PASS
```

The workflow rejected the request before continuing to AI processing.

---

## Test Case 4 — Missing customer_name

### Objective

Verify required field validation.

### Request

```json
{
  "customer_name": "",
  "customer_email": "daniel2@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": 3500
}
```

### Expected Result

```text
HTTP 400 Bad Request
```

### Actual Result

```text
PASS
```

---

## Test Case 5 — Missing company

### Request

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel3@example.com",
  "company": "",
  "requirement": "Need AI automation",
  "budget": 3500
}
```

### Expected Result

```text
HTTP 400 Bad Request
```

### Actual Result

```text
PASS
```

---

## Test Case 6 — Missing requirement

### Request

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel4@example.com",
  "company": "Nova Systems",
  "requirement": "",
  "budget": 3500
}
```

### Expected Result

```text
HTTP 400 Bad Request
```

### Actual Result

```text
PASS
```

---

## Test Case 7 — Budget as String

### Objective

Verify that the budget must be a Number.

### Request

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel30@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": "3500"
}
```

### Expected Result

```text
HTTP 400 Bad Request
```

### Response

```json
{
  "success": false,
  "error": {
    "code": "INVALID_BUDGET",
    "message": "budget must be a number greater than 0"
  }
}
```

### Actual Result

```text
PASS
```

The request was rejected because `"3500"` is a String rather than a Number.

---

## Test Case 8 — Budget = 0

### Objective

Verify that a zero budget is rejected.

### Request

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel40@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": 0
}
```

### Expected Result

```text
HTTP 400 Bad Request
```

### Actual Result

```text
PASS
```

---

## Test Case 9 — Negative Budget

### Objective

Verify that negative budget values are rejected.

### Request

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel50@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": -500
}
```

### Expected Result

```text
HTTP 400 Bad Request
```

### Actual Result

```text
PASS
```

---

## Test Case 10 — Database UNIQUE Constraint

### Objective

Verify that the database independently rejects duplicate email addresses.

A direct duplicate INSERT was executed against Supabase/PostgreSQL using an email address that already existed.

### Expected Result

The database should reject the record because of the UNIQUE constraint on `customer_email`.

### Actual Result

```text
PASS
```

PostgreSQL returned:

```text
duplicate key value violates unique constraint
"leads_customer_email_unique"
```

This confirms that the database provides an independent layer of duplicate protection.

---

## Test Summary

| Test Case | Expected Result | Result |
|---|---|---|
| Valid lead creation | 201 Created | PASS |
| Duplicate lead | 409 Conflict | PASS |
| Invalid email | 400 Bad Request | PASS |
| Missing customer_name | 400 Bad Request | PASS |
| Missing company | 400 Bad Request | PASS |
| Missing requirement | 400 Bad Request | PASS |
| Budget as String | 400 Bad Request | PASS |
| Budget = 0 | 400 Bad Request | PASS |
| Negative budget | 400 Bad Request | PASS |
| Database duplicate INSERT | Rejected | PASS |

---

## Validation Coverage

The final workflow validates:

```text
Required Fields
        |
        v
Email Format
        |
        v
Budget Type
        |
        v
Budget Value
        |
        v
Duplicate Lead
```

---

## API Response Coverage

The tested API response codes are:

| Status Code | Scenario |
|---:|---|
| `201` | Valid lead successfully created |
| `400` | Invalid input |
| `409` | Duplicate lead |

---

## Database Integrity

The application performs an existence check before creating a new record.

The database additionally enforces:

```text
UNIQUE(customer_email)
```

This creates a defense-in-depth approach:

```text
n8n duplicate detection
        +
PostgreSQL UNIQUE constraint
```

---

## Test Conclusion

All defined validation, duplicate detection, AI qualification, API response, and database integrity tests passed successfully.

The final workflow is suitable for portfolio demonstration as an AI-powered lead qualification and CRM automation system.
