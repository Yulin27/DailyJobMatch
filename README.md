# DailyJobMatch

![Made with n8n](https://img.shields.io/badge/Made%20with-n8n-00e8a2?style=flat&logo=n8n)
![Powered by Groq](https://img.shields.io/badge/Powered%20by-Groq-F55036?style=flat&logo=groq)
![Docker](https://img.shields.io/badge/Run%20with-Docker-2496ED?style=flat&logo=docker)

**Automated AI-powered job matching workflow built with n8n**

DailyJobMatch automates your job search by collecting fresh job postings daily, comparing each role against your CV using a large language model, and saving a ranked list of the most relevant opportunities to your Notion database.

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Setup Guide](#setup-guide)
4. [Configuration](#configuration)
5. [Output Format](#output-format)
6. [Roadmap](#roadmap)

---

## Overview

### Core Features

- **Daily automation**: Scheduled trigger runs the entire pipeline without manual effort
- **CV-aware matching**: Uses your full CV text (not just keywords) to evaluate fit
- **Multi-dimensional scoring**: Evaluates background match, skills overlap, experience relevance, seniority, language requirements, and company fit
- **Smart filtering**: Removes mismatches (student roles, internships, postdocs) and deduplicates by title/company/link
- **Structured LLM output**: Forces the model to return strict JSON for reliable parsing
- **Top-N selection**: Ranks all jobs by overall score and keeps only the best matches
- **Notion database**: Automatically saves matched jobs to your Notion workspace for easy tracking and status management

### How It Works

1. **CV Retrieval**: Fetches your CV from Google Drive
2. **Job Scraping**: Collects fresh LinkedIn jobs from the last 24 hours via Apify
3. **Filtering**: Removes unwanted roles and duplicates
4. **LLM Scoring**: Evaluates each job against your CV using Groq
5. **Ranking**: Selects top-N jobs based on overall score
6. **Notion Storage**: Saves matched jobs to your Notion database for tracking


---

## Setup Guide

### Prerequisites

- Docker (recommended) or Node.js for n8n installation
- Google account (for Google Drive)
- Apify account
- Groq API key
- Notion account

### 1. Install n8n

The easiest way to run n8n is with Docker:

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

For other installation methods, see the [n8n documentation](https://github.com/n8n-io/n8n).

### 2. Configure Credentials

DailyJobMatch requires credentials for the following services:

#### Google Drive
Used to fetch your CV. Follow the [n8n Google credentials guide](https://docs.n8n.io/integrations/builtin/credentials/google/).

#### Apify
Scrapes fresh job postings from LinkedIn. This workflow uses the [Linkedin Jobs Scraper - PPR](https://apify.com/curious_coder/linkedin-jobs-scraper) actor (pay-per-result pricing).

1. Create an account at [console.apify.com](https://console.apify.com)
2. Find your API token in the Apify console
3. Configure the actor with your search parameters

#### Groq
Provides the LLM for job evaluation. Get your API key from [console.groq.com](https://console.groq.com).

1. Create a Groq account at [groq.com](https://groq.com)
2. Generate an API key from the console
3. Configure the Groq credentials in n8n with your API key

#### Notion
Stores matched jobs in a database for tracking. Follow the [n8n Notion credentials guide](https://docs.n8n.io/integrations/builtin/credentials/notion/).

1. Create a Notion account at [notion.so](https://www.notion.so)
2. Create a new database called "Job Applications" with the following properties:
   - **Job Title** (Title)
   - **Company** (Text)
   - **Location** (Text)
   - **Application Link** (URL)
   - **Keywords** (Multi-select)
   - **Match score** (Number)
   - **Posting date** (Date)
   - **Status** (Select) - optional field for manual tracking
3. Connect Notion to n8n using the API integration

### 3. Import the Workflow

1. Navigate to n8n (default: http://localhost:5678)
2. Import `workflow/Daily_Job_Match_updated.json`
3. Replace all example credentials with your own
4. Configure the workflow nodes (see Configuration section below)

---

## Configuration

### Key Nodes to Customize

**Config Node**
- `apifyDatasetUrl`: Your Apify API key
- `topTotal`: Total number of jobs to save to Notion
- `keywords`: JSON array of search terms (e.g., `["Data Scientist", "ML Engineer"]`)
- `location`: Primary location for searches (e.g., `Paris, Île-de-France, France`)
- `geoId`: LinkedIn geographic ID (e.g., `105015875`)
- `jobFilters`: URL-encoded LinkedIn filters (job type, work type, time posted)
- `preferLanguages`: Preferred job languages (e.g., `English, French`)
- `seniorityPreference`: Preferred seniority level (e.g., `junior`, `mid-level`, `senior`)

**Common LinkedIn Filter Parameters:**
- `f_JT`: Job Type (`F`=Full-time, `C`=Contract, `P`=Part-time, `T`=Temporary, `I`=Internship)
- `f_WT`: Work Type (`1`=On-site, `2`=Remote, `3`=Hybrid)
- `f_TPR`: Time Posted (`r86400`=24 hours, `r604800`=Week, `r2592000`=Month)

**RetrieveCV (Google Drive)**
- Set credentials to your Google Drive OAuth2
- Specify `file` or `fileId` pointing to your CV PDF

**LinkedIn (Apify HTTP Request)**
- Ensure URL points to your Apify dataset/actor endpoint
- Add Apify API token in authentication headers

**Filter & Deduplicate (JavaScript)**
- Update banned keywords list (default: "student", "temporary", "internship", "postdoc")

**Score Job & Extract (LLM Prompt)**
- Customize scoring criteria and domain preferences
- Adjust scoring rubric ranges if needed

**Model Nodes (Groq)**
- Set credentials to your Groq API key
- Select your preferred model (llama3-70b-8192, mixtral-8x7b-32768, gemma-7b-it, etc.)

**Create a database page (Notion)**
- Set credentials to your Notion API
- Ensure `databaseId` points to your "Job Applications" database
- Property mappings are pre-configured to save:
  - Job Title, Company, Location
  - Application Link, Keywords, Match score
  - Posting date

### Testing

Before enabling the schedule:

1. Disable the Schedule Trigger node
2. Click "Execute Workflow"
3. Review each node's execution step-by-step
4. Debug any failures by inspecting node input/output
5. Verify jobs are correctly saved to your Notion database

---

## Output Format

### Scoring Dimensions

The LLM evaluates each job across six dimensions and outputs strict JSON:

```json
{
  "score": {
    "overall": 0,
    "background_match": 0,
    "skills_overlap": 0,
    "experience_relevance": 0,
    "seniority": 0,
    "language_requirement": 0,
    "company_score": 0
  },
  "summary": "",
  "keywords": [],
  "fit_bullets": [],
  "connector": ""
}
```

### Scoring Rubric

| Category | Range | Description |
|----------|-------|-------------|
| `background_match` | 0-10 | Fit with target domain (bioinformatics, pharma, biotech) |
| `skills_overlap` | 0-30 | Match between required skills and technical stack |
| `experience_relevance` | 0-30 | Alignment with responsibilities and past projects |
| `seniority` | 0-10 | Entry-level preference (penalizes senior roles) |
| `language_requirement` | 0-10 | Language fit (penalizes strict requirements) |
| `company_score` | 0-10 | Bonus for biotech/pharma/AI companies |
| `overall` | 0-100 | Sum of all categories |

Note: These scoring criteria are examples and should be customized to match your background and preferences.

### Notion Database Structure

Jobs are automatically saved to your Notion "Job Applications" database with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| **Job Title** | Title | The job position title |
| **Company** | Text | Company name |
| **Location** | Text | Job location |
| **Application Link** | URL | Direct link to job posting |
| **Keywords** | Multi-select | Key technical terms extracted by LLM |
| **Match score** | Number | Overall score (0-100) |
| **Posting date** | Date | When the job was posted |
| **Status** | Select | Manual tracking field (e.g., "Applied", "Interview", "Rejected") |

The Status field allows you to manually track your application progress for each job.

---

## License

See [LICENSE](LICENSE) file for details.
