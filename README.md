# Clara Answers - AI Agent Configuration Pipeline

**Loom Video Walkthrough:** [(https://www.loom.com/share/6ce8accd30124466bd0b7843a2fd9d73)]

##  Overview
This repository contains a zero-cost, end-to-end automation architecture built to scale the deployment of Retell AI voice agents. It transforms unstructured Demo and Onboarding calls into strict, version-controlled JSON configurations (`v1` and `v2`) without manual intervention.

This pipeline is designed as a **small internal product**, prioritizing systems thinking, prompt hygiene, robust failure handling, and strict data schema enforcement over fragile one-off scripts. 



---

##  Architecture & Engineering Choices

Built on **Make.com**, leveraging **Groq (Whisper-large-v3)** for audio processing and **Gemini 2.5 Flash** for high-fidelity extraction. 

### 1. Robustness & Idempotency
* **Universal Payload Scrubber:** Transcripts often contain rogue control characters (newlines, unescaped quotes). A regex-based Text Parser sanitizes all inputs (`[\r\n"\\]+`) before they hit the LLM, eliminating JSON parsing crashes.
* **Idempotent Database Writes:** Pipeline B performs a lookup and updates the existing `ACC-001` row rather than appending duplicates, ensuring repeatable, clean runs.

### 2. Intelligent Handling of Missing Data
* **No Hallucination:** The LLM is strictly instructed *not* to invent missing rules.
* **Non-Blocking Alert System:** If the LLM detects missing operational logic during onboarding, it logs them in a `questions_or_unknowns` array. The router intelligently saves the `v2` config to the database (preserving the extraction) but flags the database status as `⚠️ Needs Human Review` and fires an asynchronous email alert to the operations team.

### 3. Prompt Discipline & Security
* **Mandatory Call Hygiene:** The generated `system_prompt` strictly enforces both a **Business Hours Flow** (Greet -> Qualify -> Route -> Fallback -> Close) and an **After-Hours Flow** (including explicit emergency confirmation).
* **Security & Boundary Enforcement:** Pipeline B automatically injects a Red Team defense guardrail, instructing the AI to refuse prompt injection or unauthorized discount authorizations.

---

##  Ethics & Confidentiality
In strict adherence to the assignment guidelines:
* **No raw audio files** or raw transcripts are published in this repository.
* All outputs contain only the extracted operational logic required for the voice agent. 
* Personal identifiable information (PII) beyond what was explicitly necessary for the agent's routing rules has been excluded.

---

##  Reproducibility & Setup Guide

This pipeline is designed to be easily reproducible in your own environment.

### Prerequisites
1. A free **Make.com** account.
2. API Keys for **Groq** (Whisper) and **Google AI Studio** (Gemini 2.5 Flash).
3. A **Google Sheet** to act as your operational database.
4. A **Dropbox** (or Google Drive) folder for file ingestion.

### Step 1: Database Setup
Create a new Google Sheet with the following exact column headers:
`Account ID` | `Company Name` | `Version` | `Memo JSON` | `Agent Spec JSON` | `Change Log` | `Status`

### Step 2: Import Workflows
1. Log into Make.com and create a new scenario.
2. Click the `...` (More) menu at the bottom and select **Import Blueprint**.
3. Upload `pipeline_a_demo_to_v1.json` from the `/workflows` folder of this repo.
4. Repeat this process in a second scenario for `pipeline_b_onboarding_to_v2.json`.

### Step 3: Connect Your Accounts
To run this locally, you must remap the modules to your own credentials:
* **Dropbox Modules:** Authenticate your account and point them to your target folders.
* **Google Sheets Modules:** Authenticate your account, select the Google Sheet you created in Step 1, and map the outputs.
* **HTTP (Gemini) Modules:** Ensure your Gemini API key is active.
* **Groq Module (Pipeline B):** Input your Groq API key.
* **Gmail Module (Pipeline B):** Connect your email to receive the "Missing Data" fallback alerts.

### Step 4: Execution
1. Drop a Demo Transcript into your Pipeline A ingestion folder. Click **Run Once**. Watch the `v1` configuration populate your Google Sheet.
2. Drop an Onboarding Audio file into your Pipeline B folder. Click **Run Once**. Watch the system fetch `v1`, analyze the audio, and overwrite the row with `v2` and generate the changelog.

---

##  Repository Structure

* `/workflows/` - Make.com blueprint JSON files for immediate import.
* `/outputs/accounts/ACC-001/v1/` - The initial preliminary agent configuration and memo derived from the demo call.
* `/outputs/accounts/ACC-001/v2/` - The updated, deployment-ready configuration derived from the onboarding call.
* `/changelog/` - The version-control diff logging the specific operational changes made between v1 and v2.
