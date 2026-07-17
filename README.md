# Zoho Workspace Connector for ERPNext 16

**Publisher:** Meta Cloud Networks Ltd | https://www.mcnets.co.uk  
**Version:** 1.0.0  
**Compatibility:** ERPNext v16 / Frappe v16

Integrates ERPNext with **Zoho Mail** and **Zoho WorkDrive** via OAuth 2.0 REST APIs.

---

## Features

| Module | Capabilities |
|--------|-------------|
| **Zoho Mail** | Send email, list folders, fetch/search messages, inbox → Communication sync |
| **Zoho WorkDrive** | List files, upload, download, create folders, shareable links |
| **OAuth 2.0** | Full PKCE-safe flow, auto token refresh (hourly scheduler) |
| **ERPNext UI** | Settings doctype with Authorize / Test / Sync buttons |

---

## Installation

### 1. Zoho API Console Setup

1. Go to [api-console.zoho.com](https://api-console.zoho.com)
2. Create a **Server-based Application** (not Self Client for production)
3. Set **Redirect URI** to: `https://YOUR_DOMAIN/zoho/oauth/callback`
4. Note your **Client ID** and **Client Secret**
5. Required scopes (auto-requested during OAuth):
   - `ZohoMail.accounts.READ`
   - `ZohoMail.messages.ALL`
   - `ZohoMail.folders.ALL`
   - `WorkDrive.files.ALL`
   - `WorkDrive.team.READ`
   - `WorkDrive.workspace.READ`

### 2. Install the App

```bash
# From your Frappe bench directory
cd /path/to/frappe-bench

bench get-app /path/to/zoho_workspace_connector
# OR from git:
# bench get-app https://github.com/your-org/zoho_workspace_connector

bench --site YOUR_SITE install-app zoho_workspace_connector
bench --site YOUR_SITE migrate
bench build
bench restart
```

### 3. Configure in ERPNext

1. Go to **Zoho Workspace Settings** (search in Awesome Bar)
2. Enter **Client ID**, **Client Secret**, **Redirect URI**, and **Data Centre**
3. Enable **Zoho Mail** and/or **Zoho WorkDrive** as needed
4. Click **Zoho Actions → Authorize Zoho** — a popup will open
5. Log in with your Zoho account and grant permissions
6. The popup redirects back and tokens are saved automatically
7. Click **Zoho Actions → Test Connection** to verify

---

## Python API Usage

### Send Email
```python
import frappe
frappe.call("zoho_workspace_connector.api.zoho_api.send_mail",
    to="client@example.com",
    subject="Invoice from MCN",
    body="<p>Please find your invoice attached.</p>",
)
```

### Upload to WorkDrive
```python
from zoho_workspace_connector.zoho_drive.drive_client import upload_file

with open("invoice.pdf", "rb") as f:
    result = upload_file(f.read(), "invoice.pdf", mime_type="application/pdf")
    print(result)  # includes resource_id and permalink
```

### Direct Python import
```python
from zoho_workspace_connector.zoho_mail.mail_client import send_email, list_messages
from zoho_workspace_connector.zoho_drive.drive_client import list_files, get_shareable_link
from zoho_workspace_connector.oauth.token_manager import get_valid_access_token
```

---

## Scheduled Tasks

| Frequency | Task |
|-----------|------|
| Hourly | `refresh_token_if_expiring` — proactively refreshes OAuth token |
| Daily | `sync_inbox` — syncs Zoho inbox to ERPNext Communications |

---

## File Structure

```
zoho_workspace_connector/
├── zoho_workspace_connector/
│   ├── hooks.py                          # Frappe app hooks
│   ├── install.py                        # Post-install custom fields
│   ├── oauth/
│   │   └── token_manager.py              # OAuth flow + token refresh
│   ├── zoho_mail/
│   │   ├── mail_client.py                # Zoho Mail REST client
│   │   └── mail_sync.py                  # Inbox → Communication sync
│   ├── zoho_drive/
│   │   └── drive_client.py               # Zoho WorkDrive REST client
│   ├── api/
│   │   └── zoho_api.py                   # Whitelisted Frappe API methods
│   ├── zoho_settings/
│   │   └── doctype/
│   │       └── zoho_workspace_settings/  # Settings doctype (JSON + py)
│   ├── public/
│   │   └── js/zoho_connector.js          # ERPNext form buttons
│   └── templates/
│       └── pages/zoho_oauth_callback.html
├── setup.py
├── requirements.txt
└── README.md
```

---

## Support
**Meta Cloud Networks Ltd** | info@mcnets.co.uk | https://www.mcnets.co.uk  
*Beyond Cloud*
