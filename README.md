# New Life Clinic AI Chatbot

A Persian-language AI chatbot and lead-capture widget for New Life Clinic, built with **Flask**, **Google Gemini**, **Streamlit**, and **SQLite/CSV analytics**.

The project contains two main applications:

1. **Flask chatbot backend and website** — serves the clinic landing page and chat widget, handles AI conversations, captures consultation requests, and records website analytics.
2. **Streamlit analytics dashboard** — provides a password-protected view of visitors, leads, chat activity, clicks, performance, and errors.

> **Important:** This repository is prepared without the original `.env`, `analytics.db`, or `leads.csv`. Those files from the supplied archive contain secrets and/or visitor information and must remain local or in secure server storage.

## Features

### Chatbot and lead capture

- Persian-first virtual assistant for clinic services.
- Google Gemini model integration through the `google-genai` SDK.
- Conversation history maintained per chat session in the running application.
- A registration-first flow for service details, consultation and price requests.
- Lead form with name, Iranian mobile number, service category, and optional note.
- Iranian mobile number normalization and validation.
- Price-list responses after successful lead registration.
- Price and clinic wording maintained in `prices.py`.
- Optional Telegram notification for new leads.
- Basic in-memory rate limiting for chat, lead, and analytics endpoints.
- Health-check endpoint for deployment monitoring.

### Website chat widget

- Served from `static/widget.js`.
- Uses the API host from the script's `data-api` attribute, or the script URL's origin by default.
- Supports configurable title and colors through script attributes.
- Includes the Persian Vazirmatn font files.
- Can run on the Flask-served landing page or be embedded in another website.

### Visitor analytics

The widget sends events to the Flask `/track` endpoint. The backend stores events in `analytics.db`.

Tracked event types include page views, page exits, clicks, scroll depth, widget opens/closes, form activity, chat replies, JavaScript/API errors, and performance measurements.

### Streamlit dashboard

The dashboard is defined in `dashboard.py` and reads `analytics.db` and `leads.csv` from the project directory.

It includes:

- Overview KPIs and daily trends.
- Registered leads and visitor information.
- Visitor journeys.
- Click and scroll-depth analysis.
- Page and chatbot response performance.
- Error and slow-resource reporting.
- Recent visitor chat messages.
- Password-based access using `DASHBOARD_PASSWORD`.

## Project structure

```text
newlife-clinic-ai-chatbot/
├── app.py                    # Flask API, chatbot, lead capture and analytics
├── dashboard.py              # Streamlit analytics dashboard
├── prices.py                 # Clinic contact details, trust wording and price list
├── requirements.txt          # Python dependencies
├── .env.example              # Safe environment-variable template
├── .gitignore                # Excludes secrets and local visitor data
├── README.md
├── .streamlit/
│   └── config.toml           # Streamlit theme
└── static/
    ├── index.html            # Clinic landing page
    ├── widget.js             # Embeddable chat and analytics widget
    ├── fonts/                # Vazirmatn fonts and font CSS
    └── img/                  # Landing-page background images
```

The following files are created or populated at runtime and are intentionally not included in Git:

```text
.env
analytics.db
leads.csv
```

## Requirements

- Python 3.10 or newer.
- A Google Gemini API key.
- Windows, macOS, or Linux.
- Git is required only if you want to push the project using the command line.

## Local setup on Windows

Open Command Prompt in the project directory.

### 1. Create and activate a virtual environment

```cmd
py -3.10 -m venv .venv
.venv\Scripts\activate
```

If you use another installed Python version, replace `-3.10` with that version.

### 2. Install dependencies

```cmd
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 3. Create your local environment file

```cmd
copy .env.example .env
```

Open `.env` in Notepad and enter your own credentials and settings. Do not commit this file.

At minimum, configure:

```dotenv
GEMINI_API_KEY=your_gemini_api_key
DASHBOARD_PASSWORD=replace_with_a_long_unique_password
```

See [Environment variables](#environment-variables) below.

### 4. Start the Flask backend

In the first Command Prompt window:

```cmd
python app.py
```

By default, the backend listens on:

- Landing page: `http://127.0.0.1:8000/`
- Health check: `http://127.0.0.1:8000/health`
- Chat API: `http://127.0.0.1:8000/chat`
- Lead API: `http://127.0.0.1:8000/lead`
- Analytics API: `http://127.0.0.1:8000/track`
- Widget JavaScript: `http://127.0.0.1:8000/widget.js`

The health check should return:

```text
ok
```

### 5. Start the Streamlit dashboard

Open a **second** Command Prompt window, change to the same project directory, and activate the same virtual environment:

```cmd
cd /d E:\path\to\newlife-clinic-ai-chatbot
.venv\Scripts\activate
streamlit run dashboard.py
```

Streamlit normally opens the dashboard at:

```text
http://localhost:8501
```

Enter the password configured as `DASHBOARD_PASSWORD` in `.env`.

Keep the Flask backend running while using the dashboard. The backend writes analytics events and leads; the dashboard reads those files.

## Embedding the chat widget on another website

Add this script tag to the website where you want the chat widget to appear. Replace the example host with the public HTTPS address of your Flask backend:

```html
<script
  src="https://YOUR-CHATBOT-HOST/widget.js"
  data-api="https://YOUR-CHATBOT-HOST"
  data-title="دستیار نیو لایف"
  data-color="#8a5a12"
  data-color2="#c9953a">
</script>
```

If the widget is served from the same host as the API, `data-api` can be omitted because the widget uses the script URL's origin by default.

For a separate website, set `ALLOWED_ORIGINS` to the exact website origin(s), separated by commas. Example:

```dotenv
ALLOWED_ORIGINS=https://www.example.com,https://example.com
```

Do not leave wildcard CORS (`*`) enabled for a production deployment unless you have a specific reason and understand the exposure.

## Environment variables

Copy `.env.example` to `.env` and configure it locally.

| Variable | Required | Purpose |
|---|---:|---|
| `GEMINI_API_KEY` | Yes | API key used by the Google Gemini client. |
| `GEMINI_BASE_URL` | No | Optional custom Gemini-compatible API base URL. Leave blank for the SDK's default Google endpoint. |
| `MODEL` | No | Gemini model name. The source currently defaults to `gemini-3.5-flash-lite`. |
| `ALLOWED_ORIGINS` | Recommended | Comma-separated website origins allowed by Flask CORS. |
| `DASHBOARD_PASSWORD` | Yes for dashboard access | Password required to open the Streamlit dashboard. Use a long, unique password. |
| `DEBUG_ERRORS` | No | Set to `1` to include extra error details in chat API responses. Keep `0` in production. |
| `LOG_CHAT_TEXT` | No | Set to `0` to disable storing chat message text in analytics. The source defaults to `1` if unset. |
| `TELEGRAM_BOT_TOKEN` | No | Optional Telegram bot token for new-lead notifications. |
| `TELEGRAM_CHAT_ID` | No | Optional Telegram destination chat ID. |
| `PORT` | No | Flask listening port. Defaults to `8000`. |
| `CLINIC_PHONE` | Currently unused | Read by the current backend, but the displayed contact numbers are currently configured in `prices.py`. |
| `REQUEST_PAGE_URL` | Currently unused | Read by the current backend; no current request flow depends on it. |

### Clinic prices and contact details

Edit `prices.py` to update the values currently used in chatbot responses:

- `CLINIC_TEL`
- `CLINIC_WHATSAPP`
- `TRUST_CLAIM`
- `PRICES`
- `CLINIC_NAME`

Check all prices, units, phone numbers, and claims carefully before deploying changes.

## Data storage and privacy

This application processes personal information. Review your legal and privacy obligations before using it with real visitors.

- `leads.csv` stores submitted lead details, including name, phone number, requested service, optional note, session ID, and IP address.
- `analytics.db` stores website events and may include visitor/session identifiers, IP addresses, user-agent strings, page and click details, and performance information.
- Chat message text is logged by default because `LOG_CHAT_TEXT` defaults to `1`. Set `LOG_CHAT_TEXT=0` if you do not want message text written to analytics.
- The dashboard can display lead contact details and recorded chat messages.
- Use HTTPS in production, restrict dashboard access, define a data-retention policy, and provide appropriate visitor notice/consent.
- Do not upload real leads, analytics databases, API keys, bot tokens, or passwords to a public repository.

The `.gitignore` file in this repository excludes `.env`, `analytics.db`, `leads.csv`, virtual environments, Python cache files, and Streamlit secrets.

## Basic verification

After starting the Flask server:

1. Open `http://127.0.0.1:8000/`.
2. Open `http://127.0.0.1:8000/health` and confirm it returns `ok`.
3. Test a general chat message.
4. Test a service/price question and confirm the registration-first form appears.
5. Submit a test lead using a test phone number.
6. Confirm the lead is written to the local `leads.csv`.
7. Open the Streamlit dashboard and confirm the lead and analytics events appear.

Do not test with real customer details unless you are ready to store and protect them.

## GitHub upload

### Recommended: upload with Git from Command Prompt

1. Create a new repository on GitHub. A **Private** repository is recommended for this clinic project.
2. Do not select the options to add a README, `.gitignore`, or license on GitHub because this project already contains its README and `.gitignore`.
3. Open Command Prompt in the extracted project folder.
4. Check that Git is installed:

```cmd
git --version
```

5. Initialize the local repository and inspect the files:

```cmd
git init
git status
```

6. Stage the project files:

```cmd
git add .
git status
```

Before committing, verify that `.env`, `analytics.db`, and `leads.csv` are **not** listed as files to be committed.

7. Commit and connect the GitHub repository. Replace `YOUR_GITHUB_USERNAME` with your GitHub username:

```cmd
git commit -m "Initial commit: New Life Clinic AI chatbot"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/newlife-clinic-ai-chatbot.git
git push -u origin main
```

GitHub may ask you to sign in through your browser or credential manager.

### Alternative: upload through the GitHub website

1. Create a new repository on GitHub.
2. Open the repository and choose **Add file → Upload files**.
3. Upload the project files and folders from the GitHub-ready project folder.
4. Do not upload `.env`, `analytics.db`, `leads.csv`, `.venv`, or `__pycache__`.
5. Add a commit message and select **Commit changes**.

For large projects or repeated updates, using Git from Command Prompt is more convenient.

## Security checklist before publishing

- [ ] `.env` is not committed.
- [ ] Gemini API key is not present in source files or commit history.
- [ ] Telegram bot token is not committed.
- [ ] `analytics.db` is not committed.
- [ ] `leads.csv` is not committed.
- [ ] Dashboard password is strong and private.
- [ ] Production CORS origins are restricted to trusted website domains.
- [ ] `DEBUG_ERRORS=0` in production.
- [ ] HTTPS is enabled on the deployed service.
- [ ] Visitor data retention and access rules are defined.

> If a real API key or Telegram token has ever been pushed to a public GitHub repository, revoke/rotate it. Deleting the file in a later commit does not erase the secret from Git history.
