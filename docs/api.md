# API Documentation

## Base Endpoint

```text
POST /leads

The API receives lead data, validates the request, performs AI-based qualification, checks for duplicates, stores the lead in Supabase/PostgreSQL, and returns a structured response.

Create Lead
Request

Method

POST

Endpoint

/leads

Content-Type

application/json
Request Body
{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}
Successful Response
HTTP 201 Created
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
    "recommended_action": "Schedule discovery call."
  }
}
Validation Errors
Missing Required Fields

HTTP 400 Bad Request

{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "customer_name, company, and requirement are required"
  }
}

Required fields:

customer_name
customer_email
company
requirement
budget
Invalid Email

HTTP 400 Bad Request

{
  "success": false,
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Invalid email format"
  }
}
Invalid Budget

HTTP 400 Bad Request

{
  "success": false,
  "error": {
    "code": "INVALID_BUDGET",
    "message": "budget must be a number greater than 0"
  }
}

Valid budget requirements:

Must exist
Must be Number
Must be greater than 0

Examples:

2500    → Valid
"2500"  → Invalid
0       → Invalid
-500    → Invalid
Duplicate Lead

The API prevents duplicate leads based on customer_email.

HTTP 409 Conflict

{
  "success": false,
  "error": {
    "code": "DUPLICATE_LEAD",
    "message": "Lead already exists"
  }
}

Duplicate protection is implemented at both the workflow and database levels.

AI Qualification Response

The AI qualification stage generates:

{
  "lead_score": 90,
  "priority": "HOT",
  "reason": "The lead has a high budget and a clear automation requirement.",
  "recommended_action": "Schedule a discovery call."
}
Scoring
Budget	Score
>= 3000	90
1500 - 2999	70
500 - 1499	50
< 500	30
Priority
Score	Priority
80 - 100	HOT
50 - 79	WARM
0 - 49	COLD
HTTP Status Codes
Status	Meaning
201	Lead created successfully
400	Invalid request
409	Duplicate lead
Example Flow
POST /leads
      ↓
Validate request
      ↓
AI qualification
      ↓
Check duplicate
      ↓
Store in Supabase
      ↓
Return API response
