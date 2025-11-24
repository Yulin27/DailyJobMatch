# ✨ DailyJobMatch


> **Automated AI-powered job-matching workflow built with n8n**


## Overview
### 🧠 What DailyJobMatch does


DailyJobMatch automates your job search by running every morning, collecting fresh job postings, matching each role against your CV using a large language model, and then sending you a ranked shortlist of the most relevant positions directly to your inbox.


### ⭐ Core features


- 🔄 **Daily automation**: Scheduled trigger (e.g. 07:30) runs the entire pipeline without manual effort.
- 📝 **CV-aware matching**: Uses the full text of your CV, not just keywords, to evaluate fit.
- 💯 **Multi-dimensional scoring**: Breaks fit into background match, skills overlap, experience relevance, seniority, language requirements, and company score.
- 🧹 **Job cleaning & deduplication**: Normalises fields, removes obvious mismatches (student roles, internships, postdocs), and deduplicates by title/company/link.
- 📊 **Structured LLM output**: Forces the model to return strict JSON, then parses and validates it before ranking.
- 🥇 **Top-N selection**: Ranks all jobs by overall score and keeps only the top matches for you to review.
- 💌 **Nice email report**: Sends a dark-theme HTML digest with job cards, scores, keywords, fit bullets, and “View & Apply” buttons.


### 🏗️ Architecture Diagram


<img width="2268" height="386" alt="image" src="https://github.com/user-attachments/assets/7ceafeef-1611-4131-9f8b-03a719967199" />
<img width="2565" height="406" alt="image" src="https://github.com/user-attachments/assets/5b406f69-ec06-4b07-aae6-ad858304abda" />


## 📘 How to use?


> As it is still an early stage of the project, I'm still looking for methods to make it easy to use. Now it still needs lots of personalised writing and configuration.


### 🏗️ Installing & Running n8n with Docker
DailyJobMatch is built on n8n, a powerful automation tool. Docker is the recommended and easiest deployment method. For more information, please see:
https://github.com/n8n-io/n8n


### 🔐 Setting Up Required Credentials
DailyJobMatch relies on several external services to retrieve your CV, scrape job listings, score them using an LLM, and send your daily email report.


Google Drive (retrieve CV) and Gmail (send job digest email) nodes required the **Google credentials**, and here are the official docs from n8n:


> https://docs.n8n.io/integrations/builtin/credentials/google/




In our workflow, [Apify](https://console.apify.com) is used to collect fresh LinkedIn jobs within the last 24 hours. Apify provides tons of actors to scrape up-to-date web data from any website for AI apps and agents, and I chose [**_Linkedin Jobs Scraper - PPR_**](https://apify.com/curious_coder/linkedin-jobs-scraper) as it is paid by result rather than subscription. But feel free to choose the scrapper which fits your needs, and the detailed configuration steps will be on the actor's page. 


DailyJobMatch uses an LLM to read your job description, read your CV, evaluate the matches and score the fit across 6 dimensions and finally generate structured JSON outputs. n8n can be integrated with almost all the conversational chatbots. Here's how you can set up with the **OpenAI model**:


> https://docs.n8n.io/integrations/builtin/credentials/openai/


### 📘 Importing & Configuring Workflow
Once you prepare the Docker environment and required credentials, it's time to import and configure the workflow based on your own needs and credentials. My workflow is stored as workflow/Job_Hunting_Agent.json and you can download the file and import this file into n8n. 


> PS: Please replace my credentials with yours. Don't use my credentials to test or run anything.


I've written a few lines of descriptions for each node, and feel free to click and check the official docs to learn more
