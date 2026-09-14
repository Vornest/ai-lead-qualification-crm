# Testing Documentation

## Testing Overview

The AI Lead Qualification & CRM Automation System was tested using Postman and Supabase to verify input validation, AI processing, duplicate protection, API responses, and database integrity.

---

## Test Environment

| Component | Tool |
|---|---|
| Workflow Engine | n8n Cloud |
| AI Model | Google Gemini |
| Database | Supabase / PostgreSQL |
| API Client | Postman |

---

## Test Cases

### 1. Valid Lead Creation

**Purpose:** Verify that a valid lead passes all validation steps, is processed by AI, stored in the database, and returns a successful response.

**Input:**

```json
{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}

Expected Result:

HTTP 201 Created

Result: PASS

The lead was successfully processed by Gemini and stored in Supabase with an AI-generated score and priority.

2. Duplicate Lead

Purpose: Verify that an existing email address cannot create another CRM record.

Input:

{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}

Expected Result:

HTTP 409 Conflict
{
  "success": false,
  "error": {
    "code": "DUPLICATE_LEAD",
    "message": "Lead already exists"
  }
}

Result: PASS

The workflow detected the existing lead and prevented a new record from being created.

3. Invalid Email

Purpose: Verify email format validation.

Input:

{
  "customer_name": "Daniel",
  "customer_email": "daniel123",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": 3500
}

Expected Result:

HTTP 400 Bad Request

Result: PASS

The request was rejected before AI processing.

4. Missing customer_name

Purpose: Verify required field validation.

Input:

{
  "customer_name": "",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": 3500
}

Expected Result:

HTTP 400 Bad Request

Result: PASS

5. Missing company

Input:

{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "",
  "requirement": "Need AI automation",
  "budget": 3500
}

Expected Result:

HTTP 400 Bad Request

Result: PASS

6. Missing requirement

Input:

{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "",
  "budget": 3500
}

Expected Result:

HTTP 400 Bad Request

Result: PASS

7. Budget as String

Purpose: Verify that budget must be a Number rather than a String.

Input:

{
  "customer_name": "Daniel",
  "customer_email": "daniel30@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": "3500"
}

Expected Result:

HTTP 400 Bad Request
{
  "success": false,
  "error": {
    "code": "INVALID_BUDGET",
    "message": "budget must be a number greater than 0"
  }
}

Result: PASS

8. Budget = 0

Input:

{
  "customer_name": "Daniel",
  "customer_email": "daniel40@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": 0
}

Expected Result:

HTTP 400 Bad Request

Result: PASS

9. Negative Budget

Input:

{
  "customer_name": "Daniel",
  "customer_email": "daniel50@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation",
  "budget": -500
}

Expected Result:

HTTP 400 Bad Request

Result: PASS

10. Database Unique Constraint

Purpose: Verify that the database independently prevents duplicate email addresses.

A direct duplicate INSERT was executed against the Supabase/PostgreSQL database.

Expected Result:

PostgreSQL error code: 23505
duplicate key value violates unique constraint
"leads_customer_email_unique"

Result: PASS

The database rejected the duplicate record.

Test Summary
Test Case	Expected	Result
Valid lead creation	201	PASS
Duplicate lead	409	PASS
Invalid email	400	PASS
Missing customer_name	400	PASS
Missing company	400	PASS
Missing requirement	400	PASS
Budget as String	400	PASS
Budget = 0	400	PASS
Negative budget	400	PASS
Database duplicate insert	Rejected	PASS
Validation Coverage

The system validates:

Required Fields
       ↓
Email Format
       ↓
Budget Type
       ↓
Budget Value
       ↓
Duplicate Email
Database Integrity

Duplicate protection is implemented using both:

Application Layer
        +
Database Layer

The application checks for an existing lead before insertion.

The database independently enforces:

UNIQUE(customer_email)

This provides defense in depth against duplicate records.

Test Conclusion

All defined validation, duplicate detection, AI qualification, API response, and database integrity tests passed successfully.

The workflow is considered ready for portfolio demonstration.
