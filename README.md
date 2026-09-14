# AI Lead Qualification & CRM Automation System

AI-powered lead qualification and CRM automation system built with n8n, Google Gemini, and Supabase/PostgreSQL.

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

```text
Incoming Lead
      ↓
Input Validation
      ↓
AI Lead Qualification
      ↓
Lead Scoring
      ↓
Priority Classification
      ↓
Duplicate Detection
      ↓
Supabase / PostgreSQL
      ↓
Structured API Response


Architecture

                    ┌───────────────────┐
                    │      Client       │
                    │   REST / Postman  │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │    n8n Webhook    │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │  Input Validation │
                    │                   │
                    │ Required Fields   │
                    │ Email Validation  │
                    │ Budget Validation │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │   Google Gemini   │
                    │ Lead Qualification│
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Structured Output │
                    │      Parser       │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Duplicate Check   │
                    └─────────┬─────────┘
                              │
                       ┌──────┴──────┐
                       │             │
                     EXISTS       NOT EXISTS
                       │             │
                       ▼             ▼
                    409 Error     Create Row
                                     │
                                     ▼
                                201 Created

Input

The API accepts the following payload:

{
  "customer_name": "Daniel",
  "customer_email": "daniel@example.com",
  "company": "Nova Systems",
  "requirement": "Need AI automation for our sales process",
  "budget": 3500
}
AI Qualification

Google Gemini analyzes the lead and generates:

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
Priority
Score	Priority
80 - 100	HOT
50 - 79	WARM
0 - 49	COLD
Validation

The API validates incoming data before processing.

Required fields

The following fields are required:

customer_name
customer_email
company
requirement
budget
Email validation

Invalid email formats are rejected with:

400 Bad Request
Budget validation

Budget must:

Exist
Be a Number
Be greater than 0

Examples:

2500     → Valid
"2500"   → Invalid
0        → Invalid
-500     → Invalid
Duplicate Protection

Duplicate protection is implemented at two levels.

Application level

n8n checks whether the lead already exists before creating a new record.

Database level

Supabase/PostgreSQL uses a UNIQUE constraint on:

customer_email

This prevents duplicate records even if application-level validation fails.

API Response
Successful lead creation
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
    "recommended_action": "Schedule discovery call."
  }
}
Validation error
400 Bad Request
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "customer_name, company, and requirement are required"
  }
}
Invalid email
400 Bad Request
{
  "success": false,
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Invalid email format"
  }
}
Invalid budget
400 Bad Request
{
  "success": false,
  "error": {
    "code": "INVALID_BUDGET",
    "message": "budget must be a number greater than 0"
  }
}
Duplicate lead
409 Conflict
{
  "success": false,
  "error": {
    "code": "DUPLICATE_LEAD",
    "message": "Lead already exists"
  }
}
Database

The project uses Supabase/PostgreSQL for persistent lead storage.

Main table:

leads
├── id
├── customer_name
├── customer_email
├── company
├── requirement
├── budget
├── lead_score
├── priority
├── reason
├── recommended_action
└── created_at

Database protection:

UNIQUE(customer_email)
Testing

The workflow was tested using Postman and Supabase.

Test cases include:

Test	Expected Result
Valid lead	201 Created
Duplicate email	409 Conflict
Invalid email	400 Bad Request
Missing customer_name	400 Bad Request
Missing company	400 Bad Request
Missing requirement	400 Bad Request
Budget as String	400 Bad Request
Budget = 0	400 Bad Request
Negative budget	400 Bad Request
Duplicate database insert	Rejected by UNIQUE constraint
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

This project demonstrates how AI and workflow automation can be combined to automate a real business process from API input to database storage.

Author

Built as part of an Automation Engineer portfolio project.


---

# STEP 2 — Commit README

Set commit message:

```text
Create project README
