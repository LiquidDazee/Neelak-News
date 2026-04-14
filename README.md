# 🛡️ CyberNews Aggregator (news.neelak.dev)

An automated, AI-driven cybersecurity news platform designed to curate, enrich, and publish consumer-focused threat intelligence. 

**Live Site:** [news.neelak.dev](https://news.neelak.dev/)

## 📖 Project Overview
This project is an end-to-end automated publishing pipeline. It aggregates complex B2B cybersecurity alerts and utilizes AI to rewrite them into accessible, long-form content optimized for general consumers. The primary business objective is high-volume traffic generation monetized via Google AdSense.

## 🏗️ System Architecture & Workflows
The core of this platform is built on visual automation and API integrations rather than traditional monolithic code. 

**Core Stack:**
* **Workflow Automation:** Make.com (Integromat)
* **Content Generation:** Gemini
* **Frontend/CMS:** WordPress
* **Image Processing:** Automated resizing/compression pipeline to WebP.

### 🔄 The Dual-Scenario Data Pipeline (Make.com)
To ensure system stability and prevent API timeouts, the automation architecture is split into two asynchronous Make.com scenarios that communicate via an internal Data Store acting as a staging queue.

#### Scenario 1: The Ingestion Engine (`CyberNew Fe`)
<img width="1560" height="906" alt="Screenshot 2026-04-14 123645" src="https://github.com/user-attachments/assets/337cc598-5d90-4ac2-9c87-ed4163b85824" />
This workflow acts as the aggregator, running on a schedule to fetch and normalize raw data.
1.  **Multi-Source Polling:** Simultaneously pulls data from critical cybersecurity feeds, including the CISA Known Exploited Vulnerabilities database, The Hacker News, HelpNet Security, and Google News.
2.  **Data Normalization:** Parses JSON and RSS XML payloads into a standardized format.
3.  **Staging Database:** Pushes the raw news items into a Make.com Data Store (`News_Queue_DB`) and flags them with a "Pending" status, preventing duplicate processing.

#### Scenario 2: The AI Processing & Publishing Engine (`CyberNew Writer`)
<img width="1776" height="699" alt="Screenshot 2026-04-14 123025" src="https://github.com/user-attachments/assets/12e11d30-a7f6-4e56-a578-1ccea05a0e6a" />
This workflow acts as the editorial team, pulling from the staging queue to evaluate, write, and publish the content.
1.  **Queue Consumption:** Searches the `News_Queue_DB` for records flagged as "Pending."
2.  **AI Triage ("The Editor"):** Passes the raw summary to a Gemini Pro LLM with a strict system prompt to rate the relevance for small businesses on a scale of 1-10. 
    * *Routing Logic:* If the score is below 6, the record is immediately updated to "Rejected" and the flow stops, saving token costs and maintaining site quality.
3.  **Content Generation ("The Writer"):** If the score is 6 or higher, a second Gemini Pro module transforms the raw data into an SEO-optimized, 8th-grade reading level article complete with HTML tags, headlines, and an excerpt.
4.  **Asset Procurement:** Dynamically calls the Pexels API to fetch a high-resolution, landscape stock image related to the generated headline.
5.  **CMS Publishing:** Uploads the media item to the self-hosted WordPress instance, drafts the final post with the AI-generated HTML and original source link, and updates the Data Store status to "Published."

## 🖥️ Hosting & Infrastructure (Self-Hosted)
This project is entirely self-hosted, bypassing traditional managed cloud providers to maintain full control over the data and deployment environment.

* **Containerization:** The web application and its dependencies are deployed using **Docker**, ensuring an isolated and reproducible environment.
* **Secure Delivery:** Traffic is securely routed to the internal network using **Cloudflare Tunnels**, exposing the web service to the public internet safely without the need to open incoming firewall ports.
* **Network Redundancy:** The underlying network infrastructure utilizes load balancing and failover configurations to ensure high availability and uptime for the application.

## 📈 SEO & Proposed Monetization Strategy
* **Target Audience:** Consumers and SMBs looking for actionable security advice (e.g., "How to protect my PC from Ransomware").
* **Monetization (WIP):** Google AdSense. Content is structured with high word counts and specific headings (`What Happened?`, `How to Fix It`, `FAQ`) to allow for optimal in-article ad placements without degrading UX.
* **Traffic Strategy:** Optimized for Google Discover feeds using high-contrast imagery and curiosity-gap headlines.

## 🛠️ Operations & Error Handling
To ensure business continuity, the automation pipeline includes built-in redundancies:
* **Image Fallbacks:** If source images are corrupt or return a 404, the workflow routes to a default branded placeholder to prevent publishing failures.
* **Data Sanitization:** All incoming text is stripped of broken HTML and malicious scripts before AI processing.
