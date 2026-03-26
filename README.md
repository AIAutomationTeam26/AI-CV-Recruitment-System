# 🤖 AI CV Recruitment System

A fully automated, end-to-end recruitment pipeline built on **n8n**. It eliminates manual HR screening by automatically collecting CVs from email, extracting text from multiple file formats, analyzing candidates using advanced AI models, scoring them on a 0–100 scale, and dispatching personalized email responses — all without any human intervention.

---

## ✨ Features

- **📧 Email Monitoring** — Continuously polls Gmail for unread emails with CV attachments
- **📄 Multi-Format Support** — Handles PDF (text & scanned), DOCX, Image (JPG/PNG), and TXT files
- **🔍 OCR Processing** — Uses Groq Vision AI and OCR.space API for scanned/image CVs
- **🧠 AI Analysis** — Google Gemini 2.0 Flash and Groq LLM evaluate candidates objectively
- **📊 Smart Scoring** — Candidates receive a 0–100 score with Interview / Waitlist / Rejected decision
- **📬 Auto Notifications** — Sends branded HTML emails to candidates based on their result
- **🗃️ Data Logging** — All candidate data is saved to Google Sheets automatically
- **📋 Daily Reports** — Sends a daily recruitment summary email to HR at 9:00 AM

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Workflow Engine | n8n (self-hosted) |
| Primary AI | Google Gemini 2.0 Flash |
| Secondary AI | Groq — llama3-70b |
| OCR (Scanned PDF) | Groq — llama-4-scout-17b |
| OCR (Images) | OCR.space API |
| Email | Gmail API (OAuth2) |
| Storage | Google Sheets API |
| Output Parser | LangChain Auto-fixing Parser |

---

## 🔄 How It Works

```
Gmail Trigger (CV Email Arrives)
        │
        ▼
Detect File Format
  ├── PDF ──► Extract Text ──► Check if Scanned?
  │                               ├── No  ──► Normalize
  │                               └── Yes ──► Groq Vision OCR ──► Normalize
  ├── Image ─────────────────────► OCR.space API ──────────────► Normalize
  ├── DOCX ──────────────────────► Extract DOCX Text ──────────► Normalize
  └── TXT ───────────────────────► Extract from File ──────────► Normalize
                                          │
                                          ▼
                                Prepare Candidate Data
                                          │
                                          ▼
                                AI CV Analyzer Agent
                              (Gemini 2.0 Flash + Groq)
                                          │
                                          ▼
                                  Route by Score
                         ┌────────────┼────────────┐
                      ≥ 70          40–69          < 40
                    Interview      Waitlist       Rejected
                    Email          Email          Email
                         └────────────┼────────────┘
                                      │
                                      ▼
                             Append to Google Sheets
```

**Daily at 9:00 AM (separate pipeline):**
```
Scheduler ──► Fetch All Candidates ──► Calculate Stats ──► Email Report to HR
```

---

## 📊 Scoring System

| Score Range | Decision | Action |
|---|---|---|
| 70 – 100 | ✅ Interview | Congratulatory email sent, HR contacts within 2–3 days |
| 40 – 69 | 🟡 Waitlist | Application under review email sent |
| 0 – 39 | ❌ Rejected | Polite rejection email sent (no score disclosed) |

---

## 📁 Repository Structure

```
├── AI_CV_Recruitment_System_Final.json   # n8n workflow — import this
└── README.md
```

---

## 🚀 Setup Guide

### Prerequisites

- n8n instance (self-hosted or cloud)
- Gmail account
- Google Cloud project with Gmail API + Sheets API enabled
- Groq API key — [console.groq.com](https://console.groq.com)
- Google Gemini API key — [aistudio.google.com](https://aistudio.google.com)
- OCR.space API key — [ocr.space](https://ocr.space/ocrapi)
- Google Sheet named **Recruitment Tracker**

---

### Step 1 — Install n8n

**Via npm:**
```bash
npm install -g n8n
n8n start
```

**Via Docker:**
```bash
docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n
```

Then open: `http://localhost:5678`

---

### Step 2 — Import the Workflow

1. In n8n, go to **Workflows → Import from File**
2. Upload **`AI_CV_Recruitment_System_Final.json`**
3. The workflow will load with all 26 nodes intact

---

### Step 3 — Configure Gmail OAuth2

1. Go to **Credentials → Add Credential → Gmail OAuth2**
2. In [Google Cloud Console](https://console.cloud.google.com):
   - Create a new project
   - Enable the **Gmail API**
   - Create OAuth2 credentials (Desktop or Web app)
   - Copy the **Client ID** and **Client Secret**
3. Paste them into the n8n Gmail OAuth2 credential
4. Complete the authentication flow with the Gmail account that will receive CVs

---

### Step 4 — Set Up Google Sheets

1. Create a new Google Sheet named **`Recruitment Tracker`**
2. Add these column headers in **Row 1**:

```
ID | Name | Email | Phone | Experience | Score | Education | Skill1 | Skill2 | Skill3 | Skill4 | Skill5 | Recommendation | Date | Role | Category
```

3. Copy the **Spreadsheet ID** from the URL:
```
https://docs.google.com/spreadsheets/d/YOUR_SPREADSHEET_ID_HERE/edit
```
4. Open the **`Append row in sheet`** node in n8n and paste your Spreadsheet ID

---

### Step 5 — Add API Keys

**Groq API (for OCR + fallback LLM):**
1. Get your key from [console.groq.com](https://console.groq.com)
2. Open the **`Groq OCR (Scanned PDF)`** node
3. Replace the Bearer token value with your key
4. Also update the **`Groq Chat Model1`** credential node

**OCR.space (for image CVs):**
1. Get your free key from [ocr.space/ocrapi](https://ocr.space/ocrapi)
2. Open the **`OCR.space (Image CV)`** node
3. Add your API key to the request parameters

**Google Gemini:**
1. Get your key from [aistudio.google.com](https://aistudio.google.com)
2. Go to **Credentials → Add Credential → Google Gemini API**
3. Select it inside the **`Google Gemini Chat Model`** node

---

### Step 6 — Update the HR Report Email

1. Open the **`Email Daily Report to HR`** node
2. Change the **`sendTo`** field to your HR manager's email address

---

### Step 7 — Activate

1. Click the toggle in the top-right corner to set the workflow to **Active**
2. The Gmail trigger will begin polling immediately
3. The daily report will fire automatically at 9:00 AM

---

## 🧩 AI Output Schema

The AI analyzer returns this JSON structure for every CV:

```json
{
  "score": 85,
  "name": "Jane Smith",
  "email": "jane@email.com",
  "phone": "+1234567890",
  "experience_years": 5,
  "education": "BSc Computer Science — MIT",
  "skills": ["Python", "Machine Learning", "AWS", "Docker", "FastAPI"],
  "recommendation": "interview",
  "summary": "Experienced ML engineer with strong cloud background",
  "strengths": "Strong technical depth, proven project delivery",
  "weaknesses": "Limited team leadership experience"
}
```

---

## ❓ Troubleshooting

**CV email not triggering the workflow**
> Ensure Gmail OAuth2 is authenticated. The email must be unread and contain an attachment. Wait 1–2 minutes after sending — the trigger polls at intervals.

**AI returns malformed JSON**
> The Auto-fixing Output Parser handles this automatically. If it persists, check your Groq and Gemini API keys and verify you have not exceeded rate limits.

**OCR returns blank text for scanned PDF**
> Groq Vision requires a clear, high-resolution scan. Low-quality or blurry scans may fail. Increase the source document DPI before sending.

**Google Sheets not receiving data**
> Verify the Spreadsheet ID in the Append node matches your actual sheet. Confirm the Google Sheets credential has write permission.

**All candidates getting rejected**
> Review the scoring thresholds in the **Route by Score** node. You can lower the Interview threshold from 70 or the Waitlist threshold from 40 to match your requirements.

**Daily report email not arriving**
> Check the schedule trigger is Active. Verify the recipient email in the Email Daily Report node. Review n8n execution logs for errors in the Fetch All Candidates node.

---

## 📄 License

This project is open for personal and commercial use. If you build something with it, a ⭐ on the repo is always appreciated!

---

## 🙌 Built With

[n8n](https://n8n.io) &nbsp;•&nbsp; [Google Gemini](https://deepmind.google/technologies/gemini/) &nbsp;•&nbsp; [Groq](https://groq.com) &nbsp;•&nbsp; [OCR.space](https://ocr.space) &nbsp;•&nbsp; [Google Workspace](https://workspace.google.com)
