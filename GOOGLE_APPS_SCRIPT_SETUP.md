# Google Apps Script Backend Setup Guide

Complete step-by-step instructions to set up the Google Sheets backend for the Cryo Tank Calculator.

## Overview

This backend uses Google Sheets as a database and Apps Script as an API server. It provides:

- User authentication with JWT-style tokens
- Cloud storage for calculation records
- Multi-user support with record isolation

---

## Step 1: Create a New Google Sheet

1. Go to **https://sheets.google.com**
2. Click **"+ Blank"** to create a new spreadsheet
3. Rename it to **"Cryo Calculator Database"** (click on "Untitled spreadsheet" at top)

---

## Step 2: Create Required Sheet Tabs

Create 4 tabs at the bottom of the spreadsheet:

1. **Rename "Sheet1"** to `users`
2. Click **"+"** to add new sheets and name them:
   - `calc_records`
   - `split_records`
   - `reserve_records`
3. (Optional) Add `audit_log` for debugging

---

## Step 3: Add Headers to Each Sheet

Copy these headers into **Row 1** of each sheet:

### `users` Sheet

```
A1: user_id
B1: email
C1: password_hash
D1: role
E1: active
F1: created_at
```

### `calc_records` Sheet

```
A1: record_id
B1: user_id
C1: timestamp
D1: tank_type
E1: full_inches
F1: start_inches
G1: end_inches
H1: capacity
I1: capacity_unit
J1: gas
K1: flow_value
L1: timer_elapsed
M1: result_text
N1: source
```

### `split_records` Sheet

```
A1: record_id
B1: user_id
C1: timestamp
D1: source_tank
E1: destination
F1: amount_scf
G1: gas
H1: notes
I1: result_text
J1: source
```

### `reserve_records` Sheet

```
A1: record_id
B1: user_id
C1: timestamp
D1: reserve_amount
E1: gas
F1: remaining
G1: warning_flag
H1: result_text
I1: source
```

### `audit_log` Sheet (Optional)

```
A1: timestamp
B1: event_type
C1: data_json
```

---

## Step 4: Open Apps Script Editor

1. In your Google Sheet, click **Extensions** (top menu)
2. Click **Apps Script**
3. A new tab will open with the Apps Script editor
4. You'll see a file called `Code.gs` with default code

---

## Step 5: Add the Backend Code

1. In the Apps Script editor, **delete all the default code** in `Code.gs`
2. Open this file on your computer: `DEV DOCS\Apps Script Code.txt`
3. **Copy ALL the code** from that file
4. **Paste it** into the `Code.gs` editor in your browser

---

## Step 6: Configure the Secret Key

Find line 22 in the code:

```javascript
TOKEN_SECRET: "CHANGE_ME_TO_A_LONG_RANDOM_SECRET",
```

Change it to a long random string (40+ characters recommended):

```javascript
TOKEN_SECRET: "mySecretKey_2024_xyz789abc123def456ghi789jkl012mno345pqr678stu901vwx234",
```

**Important:**

- Use a unique random string
- Never share this secret
- The longer, the more secure

---

## Step 7: Save the Script

1. Click the **disk icon** (💾) or press `Ctrl+S` to save
2. You might be asked to name the project
3. Name it **"Cryo API"** or similar
4. Click **OK**

---

## Step 8: Deploy as Web App

1. Click **Deploy** button (top right corner)
2. Select **"New deployment"**
3. Click the **gear icon** (⚙️) next to "Select type"
4. Choose **"Web app"**
5. Fill in the deployment settings:
   - **Description:** `Cryo Calculator API v1`
   - **Execute as:** Select **"Me (your@email.com)"**
   - **Who has access:** Select **"Anyone"**
6. Click **"Deploy"**

---

## Step 9: Authorize the Script

First-time authorization flow:

1. A dialog will appear: **"Authorization required"**
2. Click **"Authorize access"**
3. Choose your Google account
4. You'll see a warning: **"Google hasn't verified this app"**
   - Click **"Advanced"** (bottom left)
   - Click **"Go to Cryo API (unsafe)"**
   - ⚠️ Don't worry - this is normal for personal scripts
5. Click **"Allow"** to grant the following permissions:
   - See, edit, create, and delete your spreadsheets
   - Connect to an external service

---

## Step 10: Copy the Web App URL

1. After successful deployment, you'll see **"Deployment successfully created"**
2. You'll see a **"Web app URL"** that looks like:
   ```
   https://script.google.com/macros/s/AKfycbz...abc123.../exec
   ```
3. **Click "Copy"** or manually select and copy the entire URL
4. **Save this URL** - you'll need it for the frontend configuration

---

## Step 11: Test the API

Verify your API is working:

1. Open a new browser tab
2. Paste your Web App URL and add `?action=ping` to the end:
   ```
   https://script.google.com/macros/s/YOUR_ID_HERE/exec?action=ping
   ```
3. Press Enter

**Expected Response:**

```json
{
  "ok": true,
  "service": "cryo-sheets-api",
  "time": "2025-12-19T10:30:00.000Z",
  "status": 200
}
```

✅ **If you see this, your API is working correctly!**

❌ **If you see an error:**

- Check that you copied the full URL including `/exec`
- Verify the script is deployed (Step 8)
- Check authorization was completed (Step 9)

---

## Step 12: Create Your First User

Users must be created manually by running a script function:

1. Back in the Apps Script editor
2. Add this test function at the **bottom of the code** (after line 387):

```javascript
// Test function - run once to create user, then delete
function createTestUser() {
  const result = adminCreateUser("test@example.com", "123456", "user");
  Logger.log(result);
}
```

3. **Save** the script (Ctrl+S)
4. Select **`createTestUser`** from the function dropdown (top of editor)
5. Click **"Run"** (▶️ button)
6. Wait for execution to complete

### Verify User Creation

1. Go back to your Google Sheet
2. Open the `users` tab
3. You should see a new row with:
   - **user_id:** UUID (e.g., `a1b2c3d4-e5f6-...`)
   - **email:** `test@example.com`
   - **password_hash:** Long encrypted string (e.g., `a7B9c2D4e6F8...`)
   - **role:** `user`
   - **active:** `TRUE`
   - **created_at:** Timestamp

### Create Additional Users

To create more users, modify the test function:

```javascript
function createAnotherUser() {
  const result = adminCreateUser("driver@company.com", "password123", "user");
  Logger.log(result);
}
```

**User Roles:**

- `user` - Regular user (default)
- `admin` - Admin user (for future features)

---

## Step 13: Update Frontend Configuration

Now connect your frontend app to the API:

### Update `records.html`

1. Open: `cryo-calculator-dev-files/records.html`
2. Find **line 74**:
   ```javascript
   const SAVE_ENDPOINT = "PASTE_YOUR_APPS_SCRIPT_WEBAPP_URL_HERE";
   ```
3. Replace with your actual Web App URL:
   ```javascript
   const SAVE_ENDPOINT = "https://script.google.com/macros/s/AKfycbxBpADMfCQu4p2p6K8s9LHed_5kx2yucg47zMim3dVUL4YwDmfh8DuCGVw4aqf4yIU9/exec";
   ```
4. **Save** the file

### Update `calculator.html` (Future Integration)

The current `calculator.html` might not have full backend integration. You'll need to add save functionality using the patterns shown in `DEV DOCS\shared JS file.txt`.

---

## Step 14: Test End-to-End

### Start Local Server

```bash
cd "C:\Users\XL 20\Desktop\WorkeSpace\Archive (5)\cryo-calculator-dev-files"
python -m http.server 8000
```

### Open App

1. Open browser to: **http://localhost:8000**
2. You'll see the splash screen
3. Login modal appears

### Test Guest Mode

1. Click **"Continue as Guest"**
2. All calculations work
3. Records save to browser localStorage only

### Test Login (When Frontend is Connected)

1. Enter email: `test@example.com`
2. Enter password: `123456`
3. Click **"Login"**
4. Should authenticate and unlock cloud sync

---

## Complete Setup Checklist

Use this checklist to verify everything is configured:

- [ ] Created Google Sheet named "Cryo Calculator Database"
- [ ] Created 4 sheet tabs: `users`, `calc_records`, `split_records`, `reserve_records`
- [ ] Added header rows to all sheets
- [ ] Opened Apps Script editor
- [ ] Pasted code from `DEV DOCS\Apps Script Code.txt`
- [ ] Changed `TOKEN_SECRET` to a random string (40+ chars)
- [ ] Saved the script
- [ ] Deployed as Web App (Execute as: Me, Access: Anyone)
- [ ] Authorized permissions (Advanced → Go to app)
- [ ] Copied Web App URL
- [ ] Tested API with `?action=ping` (got `"ok": true`)
- [ ] Created test user function
- [ ] Ran test user function
- [ ] Verified user appears in `users` sheet
- [ ] Updated `records.html` with Web App URL
- [ ] Tested app in browser on localhost

---

## Troubleshooting

### "Authorization required" error

**Solution:** Repeat Step 9. Click "Authorize access" and allow all permissions.

### "Exception: Invalid argument" when creating user

**Solution:** Check that the `users` sheet has all required headers in Row 1.

### API returns empty response

**Solution:**

- Verify deployment (Step 8)
- Check "Execute as: Me" is selected
- Ensure "Who has access: Anyone" is selected

### "Script function not found"

**Solution:** Make sure you saved the script (Ctrl+S) before running.

### User not appearing in sheet

**Solution:**

- Check spelling of sheet name (must be exactly `users`)
- Verify headers are in Row 1
- Check Apps Script execution log (View → Logs)

### Frontend can't connect to API

**Solution:**

- Verify Web App URL is correct (includes `/exec`)
- Check browser console for CORS errors
- Ensure deployment is set to "Anyone" access

---

## Security Notes

### Best Practices

1. **Change TOKEN_SECRET immediately** - Never use the default value
2. **Use strong passwords** for user accounts (6+ characters minimum)
3. **Don't share your Web App URL publicly** unless you want public access
4. **Review audit logs** periodically (if enabled)

### Password Security

- Passwords are hashed with SHA-256 before storage
- Original passwords are never stored in the sheet
- Tokens expire after 7 days (configurable in `TOKEN_TTL_SECONDS`)

### Access Control

Current setup allows "Anyone with the link" to access the API. To restrict:

1. Deploy → Manage deployments
2. Edit deployment
3. Change "Who has access" to specific users/groups

---

## Next Steps

After completing this setup:

1. **Test the API** - Use browser or Postman to test endpoints
2. **Connect Frontend** - Update login flow in `index.html`
3. **Add Users** - Create accounts for all drivers/technicians
4. **Test Cloud Sync** - Make calculations and verify they appear in sheets
5. **Configure Email Notifications** - See `DEV DOCS\google email script.txt`

---

## API Endpoints Reference

### GET Endpoints

**Health Check:**

```
GET https://your-web-app-url/exec?action=ping
```

### POST Endpoints

**Login:**

```json
POST https://your-web-app-url/exec
{
  "action": "login",
  "email": "test@example.com",
  "password": "123456"
}
```

**Save Record:**

```json
POST https://your-web-app-url/exec
{
  "action": "save",
  "type": "calc",
  "token": "your-jwt-token",
  "record": {
    "tank_type": "horizontal",
    "gas": "LIN",
    "result_text": "..."
  }
}
```

**Fetch Records:**

```json
POST https://your-web-app-url/exec
{
  "action": "fetch",
  "type": "all",
  "token": "your-jwt-token",
  "limit": 100
}
```

---

## Additional Resources

- **Apps Script Documentation:** https://developers.google.com/apps-script
- **Google Sheets API:** https://developers.google.com/sheets/api
- **Project Files:** `DEV DOCS/` folder

---

## Support

If you encounter issues:

1. Check the troubleshooting section above
2. Review Apps Script execution logs (View → Logs)
3. Check browser console for frontend errors (F12)
4. Verify all checklist items are completed

---

**Setup Complete!** 🎉

Your Google Apps Script backend is now ready to handle authentication and data storage for the Cryo Tank Calculator.
