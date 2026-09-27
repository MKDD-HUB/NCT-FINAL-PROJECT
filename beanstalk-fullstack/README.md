# Project 3: Student Grade Tracker — Full-Stack App on AWS Elastic Beanstalk

A full-stack student grade tracking application deployed on AWS Elastic Beanstalk, backed by Amazon RDS (PostgreSQL). Built as the final project for the Neo Cloud Tech IT placement (cloud computing track).

*status* Deployed and verified — full CRUD flow tested live (add student → add grade → confirmed on dashboard).


## Overview

The app lets a teacher/admin add students, record grades per subject and semester, and view a dashboard summarizing class performance (averages, top students, grade distribution).

*Stack:* Node.js + Express + EJS (server-rendered views) + PostgreSQL (Amazon RDS)

## Architecture

```
Browser (User)
      |
      v
Elastic Beanstalk Environment
  |------------------------|
  |  EC2 + App (Node.js)   |----->  RDS PostgreSQL (private, no public access)
  |------------------------|
   Security Group Layering:
   - EC2 SG: allows inbound HTTP from the load balancer
   - RDS SG: allows inbound Postgres (5432) only from the EC2 SG

   ```




- *Elastic Beanstalk* manages provisioning, deployment, scaling, and health monitoring of the EC2 instance running the app — no manual server setup.
- *RDS PostgreSQL* is kept private (not publicly accessible); only the Beanstalk EC2 instance's security group is allowed to connect to it, following least-privilege access.
- The app creates its own tables on startup (see initializeDatabase() in app.js), so no manual schema migration is needed on first deploy.

## Features

- Dashboard: total students, total grades, class average, top 5 students, grade distribution (A–F)
- Add / view / delete students
- Add grades per student (subject, score, semester) with automatic letter-grade calculation
- Input validation (score 0–100, unique email) with inline error messages
- /health endpoint for Elastic Beanstalk's health monitoring

## Project structure

```
application/
├── app.js                  # Express app: routes, DB connection, table init
├── package.json
├── views/                  # EJS templates
│   ├── dashboard.ejs
│   ├── students.ejs
│   ├── student-form.ejs
│   ├── student-detail.ejs
│   ├── grade-form.ejs
│   └── error.ejs
└── public/style.css
README.md          # Full AWS deployment walkthrough
```

## Lessons learned during deployment

Two real issues came up wiring this together, worth knowing rather than assuming a clean first deploy:

1. *Wrong security group as the RDS source.* When adding the inbound rule to allow the app to reach RDS, autocomplete initially suggested the RDS security group's own ID (visually similar to the EC2 instance's ID) instead of the actual EC2/Beanstalk security group. The fix was to explicitly cross-check the EC2 instance's security group ID (EC2 console → Instances → Security tab) against the source selected in the RDS security group's inbound rule, rather than trusting the first autocomplete match.
2. *ECONNREFUSED 127.0.0.1:5432.* The security group was correct, but the app still failed to connect — because it was falling back to localhost (process.env.RDS_HOSTNAME || 'localhost' in app.js). The Elastic Beanstalk *Environment properties* had never actually been saved — the form still showed the blank placeholder row. Fixed by going to Configuration → Software → Environment properties and filling in RDS_HOSTNAME, RDS_PORT, RDS_DB_NAME, RDS_USERNAME, RDS_PASSWORD explicitly, then applying — which redeployed and connected successfully.

Takeaway for the presentation: health status alone ("Severe" → "Degraded" → "Ok") doesn't tell you what's wrong — the actual fix each time came from reading *Logs → Request Logs → Last 100 Lines*.

*Live URL:* http://student-grade-tracker-env.eba-xgwjfxbn.us-east-1.elasticbeanstalk.com/

![Dashboard image](image/nct%20final1.png)
