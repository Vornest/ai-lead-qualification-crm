# AI Lead Qualification & CRM Automation System

AI-powered lead qualification and CRM automation built with n8n, Google Gemini, and Supabase/PostgreSQL.

## Overview

This project automates the initial lead qualification process for a sales workflow.

Incoming leads are received through a REST webhook, validated, analyzed by Google Gemini, scored based on business value, classified by priority, checked for duplicates, and stored in Supabase/PostgreSQL.

The goal is to reduce manual lead qualification, improve lead prioritization, and prevent duplicate CRM records.

## Problem

Sales teams can receive a large number of leads without a consistent qualification process.

Manual qualification can result in:

- Slow lead response
- Inconsistent prioritization
- Duplicate CRM records
- Missed sales opportunities

## Solution

The system automates the initial lead processing pipeline:

Webhook → Input Validation → AI Qualification → Lead Scoring → Priority Classification → Duplicate Detection → Supabase → API Response

## Architecture

```text
Client / Postman
      |
      v
n8n Webhook
      |
      v
Required Validation
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
Search Lead
      |
      v
IF Exists?
    /      \
  YES       NO
   |         |
   v         v
409       Create Row
Conflict      |
              v
         201 Created
Input

The API accepts:

{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}
AI Qualification

Google Gemini generates:

lead_score
priority
reason
recommended_action

Example:

{
  "lead_score": 90,
  "priority": "HOT",
  "reason": "The lead has a high budget and a clear automation requirement.",
  "recommended_action": "Schedule a discovery call."
}
Scoring Rules
Budget	Score
>= 3000	90
1500 - 2999	70
500 - 1499	50
< 500	30
Priority Rules
Score	Priority
80 - 100	HOT
50 - 79	WARM
0 - 49	COLD
Validation

The API validates incoming data before AI processing.

Required Fields
customer_name
customer_email
company
requirement
budget
Email Validation

Invalid email formats return:

400 Bad Request

Budget Validation

Budget must:

Exist
Be a Number
Be greater than 0

Examples:

Input	Result
2500	Valid
"2500"	Invalid
0	Invalid
-500	Invalid
Duplicate Protection

Duplicate protection is implemented at two levels.

Application Level

n8n searches for an existing lead using customer_email before creating a new record.

Database Level

Supabase/PostgreSQL enforces a UNIQUE constraint on:

customer_email

This provides a second layer of protection against duplicate records.

API Response
Successful Lead Creation

201 Created

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
Validation Error

400 Bad Request

{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "customer_name, company, and requirement are required"
  }
}
Invalid Email

400 Bad Request

{
  "success": false,
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Invalid email format"
  }
}
Invalid Budget

400 Bad Request

{
  "success": false,
  "error": {
    "code": "INVALID_BUDGET",
    "message": "budget must be a number greater than 0"
  }
}
Duplicate Lead

409 Conflict

{
  "success": false,
  "error": {
    "code": "DUPLICATE_LEAD",
    "message": "Lead already exists"
  }
}
Database

The project uses Supabase/PostgreSQL for persistent CRM storage.

Main table:

leads

Fields:

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

Database protection:

UNIQUE(customer_email)

Documentation
System Architecture
API Documentation
Testing Documentation
n8n Workflow Export
Project Evidence
n8n Workflow

Successful API Request

Validation / Error Handling

Supabase Database

Testing

The system was tested using Postman and Supabase.

Test	Expected Result
Valid lead	201 Created
Duplicate email	409 Conflict
Invalid email	400 Bad Request
Missing required field	400 Bad Request
Budget as String	400 Bad Request
Budget = 0	400 Bad Request
Negative budget	400 Bad Request
Database duplicate insert	Rejected

All defined validation, duplicate detection, AI qualification, API response, and database integrity tests passed successfully.

Technology Stack
n8n
Google Gemini
Supabase
PostgreSQL
REST API
Webhooks
Postman
Skills Demonstrated
Workflow Automation
REST API Integration
Webhook Architecture
AI Integration
Prompt Engineering
Structured JSON Output
Database Design
PostgreSQL
Input Validation
Error Handling
Duplicate Detection
API Testing
Business Logic
Data Transformation
Project Goal

This project demonstrates how AI and workflow automation can automate a real business process from API input to CRM database storage.

Author

Built as part of an Automation Engineer portfolio project.
