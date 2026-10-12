# Research Humanizer — secure hosted web version

The project is **deploy-ready, not already hosted**. Deploy it once to a web-hosting account, then access the HTTPS website on mobile and desktop without installing software on the devices you use.

## Publish from your browser

1. Create a **private** GitHub repository. Unzip the project and upload the extracted files into the repository root, preserving `render.yaml`. Do not upload a `.env` file or credentials.
2. Sign into [Render](https://dashboard.render.com/). Select **New → Blueprint**, connect the GitHub repository and create the web service described by `render.yaml`.
3. In Render's secret environment-variable prompts enter **only there**:
   - `OPENAI_API_KEY`: the key obtained from https://platform.openai.com/api-keys
   - `APP_ACCESS_PASSWORD`: a unique, strong private passphrase of **at least 16 characters**; **not** the API key.
4. Wait until deployment shows **Live**. Open your Render HTTPS URL (usually `https://<your-service>.onrender.com`). Use the *website access password* on the login screen.
5. Bookmark the site on Android, iPhone, or desktop. After publishing, you need no local Python server, package installation, or API key in the browser.

The app uses Render's Python runtime (`buildCommand: pip install -r requirements.txt`, `startCommand: python server.py`). Free-tier availability, cold-start delays, usage, and pricing depend on Render's current terms. OpenAI API usage is billed separately from a ChatGPT subscription.

## Architecture and security

- `index.html`: responsive browser client; **contains no API key**. It never asks the visitor to enter an API key.
- `server.py`: hosts the UI, validates origin, provides authenticated API routes, extracts PDF/DOCX/TXT/MD in memory, calls OpenAI server-side, and creates DOCX exports.
- `render.yaml`: single Python web service with required private secrets supplied via Render dashboard, never committed to Git.
- Auth uses a separate high-entropy 16+ character password and ephemeral server-side sessions (8-hour lifetime), with `HttpOnly`, `SameSite=Strict` and `Secure` cookies on HTTPS. The session is revoked on logout. Failed sign-in attempts have a best-effort 15-minute lockout. For multi-instance or high-security deployment, use external access controls and a persistent rate limiter.
- The AI editing call sets `store=False`. The application does not intentionally persist documents or generated text. Uploaded files are transferred to the host for in-memory extraction and edited excerpts to the API provider. Hosting/API providers may separately retain operational metadata or data per their policies. Do not upload confidential material without approval.
- Browser-origin checks, JSON-only APIs, no CORS, request-size limits, output caps and generic upstream-error messages reduce exposure. A strong access password **does not** replace provider-side spend limits or WAF rules; set usage limits/alerts for the API key.

## File constraints

PDF/DOCX/TXT/MD, upload up to 12 MB. DOCX tables are turned into linearized text. PDF text and reading order may be imperfect. Scanned/image-only PDFs require OCR. Each AI request is limited to 8,500 characters; longer content is automatically processed in chunks. A reference list is skipped and reattached *only* when preceded by a recognized standalone heading. Check all citations, statistics, quotations, tables and mathematical notation in the final result.

Output is a fresh, simple DOCX, not pixel-perfect preservation of the imported document. Text comparison flags possible missing numerical and citation tokens; it does not verify reference authenticity or scientific validity. No tool can guarantee any result on AI-authorship or plagiarism checks, nor should editing be used to misrepresent authorship.

## Troubleshooting

- "Unlock" requires `APP_ACCESS_PASSWORD`, not the OpenAI API key.
- "API key missing on server" means `OPENAI_API_KEY` has not been configured on Render.
- If the page does not load, check Render's service status and deployment logs.
- For PDF scans, perform OCR separately first.

## Optional developer tests

Once Python dependencies in `requirements.txt` plus `requests` are installed, run:

```sh
python -m unittest discover -s tests -v
```

Local development (never set `ALLOW_HTTP_COOKIES=1` on a public host):

```sh
# export OPENAI_API_KEY=...           # use your shell's secure setup
# export APP_ACCESS_PASSWORD=...
# export ALLOW_HTTP_COOKIES=1         # localhost ONLY
python server.py
```

The frontend cannot safely protect an API key by itself; hosted server-side proxying and access control are integral to this version.