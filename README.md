# Career Agent — Job Monitoring System

An automated agent that tracks company career pages, detects new job openings, and sends personalized email alerts based on each user's selected departments and region.

## How It Works

1. **Users subscribe** with their email, choose departments (Engineering, Product, Marketing, etc.) and a region preference (Israel only / global), and add up to 5 company career pages to follow.
2. **The scanner runs** on a schedule (cron-triggered endpoint) or right after a new subscription.
3. **Each tracked career page is scraped** with a headless browser (Playwright), and candidate job links are extracted from the page and its iframes.
4. **New jobs are detected** by comparing against a job cache in PostgreSQL. Only postings that weren't seen before trigger alerts.
5. **Each user receives a personalized email** with only the new jobs that match their companies, departments and region.
   New users receive the full current list on their first run.

## Job Detection

Career pages differ widely in structure, so the scraper uses a generic, keyword-based approach:

- Collects all links on the page, including links inside embedded iframes (where many hiring widgets live).
- Keeps links whose text contains job-title keywords (engineer, developer, analyst, etc.) or whose URL looks like a job posting (`/job`, `/career`, `/apply`, `lever.co`, `greenhouse.io`, ...).
- Filters out noise such as privacy policies, login pages, blog posts and social links.
- Tags jobs as Israel-based when the title or URL mentions an Israeli location.

## Classification

Each job title is classified into a department using keyword matching:
Engineering, Product, Marketing, Finance, HR / Operations, Support.
Only jobs in the user's selected departments are included in their email.

## Email Delivery

Alerts are sent through the Resend Email API as HTML emails, grouped by department.

## Deployment

- Hosted on Render
- PostgreSQL on Neon
- `/trigger-scan` endpoint for scheduled (cron) runs
- Scans run as background tasks so requests return immediately

## Tech Stack

- **Backend:** FastAPI, Jinja2 templates
- **Scraping:** Async Playwright (Chromium, headless)
- **Database:** PostgreSQL (Neon), psycopg2
- **Email:** Resend API

## Roadmap

- Dedicated integrations for Greenhouse and Lever using their public job-board APIs, instead of scraping
- Scan each unique company once and share results across all users following it
- Unit and regression tests for job filtering and classification, with CI on every pull request
- Whole-word keyword matching to reduce misclassification
- Concurrent scanning of multiple companies
- Authentication for user actions and the scan trigger endpoint
- Monitoring that flags a company whose scan suddenly returns zero jobs (likely a broken scraper)
