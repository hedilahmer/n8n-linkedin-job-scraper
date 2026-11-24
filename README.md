# Automated LinkedIn Job Digest (n8n + Apify + Gemini AI)

An end-to-end automated workflow built in **n8n** that scrapes daily LinkedIn job postings, filters out previously seen listings deterministically, evaluates relevance using **Google Gemini**, and delivers a single formatted HTML digest via **Gmail OAuth2**.

![Workflow Architecture](workflow-screenshot.png)

## Why I Built This
Manually checking LinkedIn daily for *Werkstudent* (working student) positions is repetitive and noisy. I built this pipeline to automate discovery while avoiding two common AI automation pitfalls:
1. **Zero LLM waste on deduplication:** Instead of asking an LLM to compare today's jobs against past jobs (which is expensive and prone to hallucinations), the workflow uses n8n's stateful deduplication node to filter out seen job IDs before any AI tokens are spent.
2. **Batched processing:** Newly discovered listings are aggregated into a single payload so the AI evaluates all new openings in one pass and sends a single clean email rather than spamming individual alerts.

## Architecture Overview

1. **Scheduled Ingestion (`Schedule Trigger` + `Apify API`)**
   - Triggers daily at 8:00 AM.
   - Executes the Apify LinkedIn Jobs Scraper Actor for target roles (*Werkstudent* in Frankfurt am Main) posted within the last 24 hours (`r86400`), then fetches the resulting dataset items.
2. **Deterministic Deduplication & Batching (`Remove Duplicates` + `Aggregate` + `If`)**
   - Compares incoming job IDs against persistent execution history to discard duplicates.
   - Aggregates all genuinely new jobs into a single `jobs` array and halts execution cleanly if zero new jobs are found.
3. **AI Screening & HTML Generation (`AI Agent` + `Google Gemini`)**
   - Evaluates new job descriptions against target technical skills and excludes non-student or irrelevant roles.
   - Formats matching roles into clean, mobile-friendly HTML with concise summaries and direct application links.
4. **Automated Delivery (`Gmail OAuth2`)**
   - Sends a single daily HTML digest directly to my inbox.

## How to Import & Run This Workflow

1. Download [`linkedin-job-digest-workflow.json`](./linkedin-job-digest-workflow.json) from this repository.
2. Open your **n8n** instance (Cloud or self-hosted) and create a new workflow.
3. Click the top-right menu (`...`) → **Import from File** and select the downloaded JSON file.
4. Configure your credentials in the respective nodes:
   - **Apify:** Add your Apify API token.
   - **Google Gemini Chat Model:** Add your Google AI Studio API key.
   - **Gmail:** Connect your Google account via OAuth2 and set the recipient email address.
5. Customize the target skills in the **AI Agent** prompt and toggle the workflow to **Active**.
