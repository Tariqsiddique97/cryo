# CORS Error Fix Guide

## Problem
Getting CORS errors and 405 status on preflight OPTIONS requests when calling Apps Script from localhost.

## Root Cause
Google Apps Script deployment setting "Who has access" is likely set to **"Anyone with the link"** instead of **"Anyone"**.

## Solution: Redeploy Apps Script

### Step 1: Open Your Apps Script
1. Go to your Google Sheet
2. Click **Extensions** → **Apps Script**

### Step 2: Update the Code (Optional)
The code has been updated in `/DEV DOCS/Apps Script Code.txt` with better CORS handling.

If you want to apply the updates:
1. Open the updated file: `DEV DOCS/Apps Script Code.txt`
2. Copy all the code
3. Paste it into Code.gs in Apps Script editor
4. Click **Save** (Ctrl+S or Cmd+S)

### Step 3: Create NEW Deployment (IMPORTANT!)

**Don't just update the existing deployment - create a NEW one:**

1. Click **Deploy** → **New deployment** (NOT "Manage deployments")

2. Click the gear icon ⚙️ next to "Select type"

3. Select **Web app**

4. Fill in settings:
   - **Description**: `Cryo API v2` (or any name)
   - **Execute as**: `Me (your-email@gmail.com)`
   - **Who has access**: **`Anyone`** ⚠️ **NOT "Anyone with the link"**

5. Click **Deploy**

6. If prompted to authorize:
   - Click **Authorize access**
   - Choose your Google account
   - Click **Advanced**
   - Click **Go to [Project Name] (unsafe)**
   - Click **Allow**

7. **Copy the new Web App URL** (ends with `/exec`)

### Step 4: Update Frontend Files

Update the `API_ENDPOINT` in these three files:

```bash
# File 1: index.html (line 114)
cd /Users/xlmb02/Downloads/cryo/cryo-calculator-dev-files
# Find this line:
const API_ENDPOINT = "https://script.google.com/macros/s/OLD_URL/exec";
# Replace with your NEW URL

# File 2: calculator.html (line 2131)
# Same replacement

# File 3: records.html (line 74)
# Same replacement
```

### Step 5: Test the Endpoint

Open a new browser tab and test:

```
https://script.google.com/macros/s/YOUR_NEW_URL/exec?action=ping
```

Should return:
```json
{"ok":true,"service":"cryo-sheets-api","time":"2025-12-19T...","status":200}
```

### Step 6: Test from Frontend

1. Reload your app: http://localhost:8000/cryo-calculator-dev-files/
2. Click "Login"
3. Enter: `test@example.com` / `123456`
4. Should login successfully without CORS errors

### Step 7: Verify in Browser DevTools

Open DevTools (F12) → Network tab:
- Should see successful OPTIONS (204) and POST (200) requests
- No CORS errors
- No 405 errors

---

## Why This Happens

Google Apps Script has two access modes:

### "Anyone with the link"
- Requires authentication for API calls
- **Blocks CORS preflight OPTIONS requests** from unknown origins
- ❌ Causes 405 errors from localhost

### "Anyone"
- Allows unauthenticated OPTIONS requests
- ✅ Properly handles CORS from localhost
- Still requires token authentication for actual data operations (your login system)

---

## Quick Command to Update All Files

```bash
cd /Users/xlmb02/Downloads/cryo/cryo-calculator-dev-files

# Replace OLD_URL with your new deployment ID
OLD_URL="AKfycbwIWol7vhD17iQ1f7fdVtOWrAR0RuFelGMYM6jpooD-rOkgDL1wdtlYZr0IoZmrDg0M"
NEW_URL="YOUR_NEW_DEPLOYMENT_ID_HERE"

# Update all three files at once (macOS/Linux)
sed -i '' "s/$OLD_URL/$NEW_URL/g" index.html
sed -i '' "s/$OLD_URL/$NEW_URL/g" calculator.html
sed -i '' "s/$OLD_URL/$NEW_URL/g" records.html

echo "✓ Updated API endpoints in all files"
```

---

## Still Not Working?

### Check 1: Deployment Access Level
1. Apps Script → Deploy → Manage deployments
2. Click pencil icon on your deployment
3. Verify "Who has access" shows **"Anyone"**
4. If not, change it and click "Deploy"

### Check 2: Wait for Propagation
After deployment changes, wait 30-60 seconds for Google's servers to update.

### Check 3: Clear Browser Cache
1. Open DevTools (F12)
2. Right-click refresh button → Empty Cache and Hard Reload
3. Or use Incognito mode

### Check 4: Test in Postman
If Postman works but browser doesn't:
- Definitely a CORS issue
- Redeploy with "Anyone" access
- Make sure you're using the NEW deployment URL

---

## Security Note

Setting access to "Anyone" is safe because:
- OPTIONS requests don't access data (just CORS preflight)
- All actual data operations require JWT token authentication
- Tokens are validated with HMAC signature
- Users must login with email + password to get a token
- Token expires after 7 days

The "Anyone" setting only allows the CORS preflight to succeed. Your data is still protected by the login system.
