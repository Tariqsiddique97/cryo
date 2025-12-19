# API Test Commands (cURL)

Your deployed Web App URL:
```
https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec
```

---

## 1. Health Check (Ping)

**Test if API is running:**

### Windows (CMD)
```cmd
curl "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec?action=ping"
```

### Windows (PowerShell)
```powershell
curl "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec?action=ping"
```

### Linux/Mac
```bash
curl "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec?action=ping"
```

**Expected Response:**
```json
{
  "ok": true,
  "service": "cryo-sheets-api",
  "time": "2025-12-19T12:00:00.000Z",
  "status": 200
}
```

---

## 2. Login (Get Authentication Token)

**Login with test user:**

### Windows (CMD)
```cmd
curl -X POST ^
  -H "Content-Type: application/json" ^
  -d "{\"action\":\"login\",\"email\":\"test@example.com\",\"password\":\"123456\"}" ^
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

### Windows (PowerShell)
```powershell
curl -Method POST `
  -Headers @{"Content-Type"="application/json"} `
  -Body '{"action":"login","email":"test@example.com","password":"123456"}' `
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

### Linux/Mac
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"action":"login","email":"test@example.com","password":"123456"}' \
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

**Expected Response:**
```json
{
  "ok": true,
  "auth": {
    "user_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "email": "test@example.com",
    "role": "user",
    "token": "eyJhbGc...long-jwt-token...xyz",
    "exp": 1734696000
  },
  "status": 200
}
```

**Important:** Copy the `token` value from the response - you'll need it for the next commands!

---

## 3. Save Calculator Record

**Save a calculation to the database:**

Replace `YOUR_TOKEN_HERE` with the token from the login response.

### Windows (CMD)
```cmd
curl -X POST ^
  -H "Content-Type: application/json" ^
  -d "{\"action\":\"save\",\"type\":\"calc\",\"token\":\"YOUR_TOKEN_HERE\",\"record\":{\"tank_type\":\"horizontal\",\"full_inches\":\"48\",\"start_inches\":\"12\",\"end_inches\":\"36\",\"capacity\":\"500\",\"capacity_unit\":\"gal\",\"gas\":\"LIN\",\"flow_value\":\"25\",\"timer_elapsed\":\"00:15:30\",\"result_text\":\"Transferred 387.5 gallons (2612 SCF)\"}}" ^
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

### Windows (PowerShell)
```powershell
$token = "YOUR_TOKEN_HERE"
curl -Method POST `
  -Headers @{"Content-Type"="application/json"} `
  -Body "{`"action`":`"save`",`"type`":`"calc`",`"token`":`"$token`",`"record`":{`"tank_type`":`"horizontal`",`"full_inches`":`"48`",`"start_inches`":`"12`",`"end_inches`":`"36`",`"capacity`":`"500`",`"capacity_unit`":`"gal`",`"gas`":`"LIN`",`"flow_value`":`"25`",`"timer_elapsed`":`"00:15:30`",`"result_text`":`"Transferred 387.5 gallons (2612 SCF)`"}}" `
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

### Linux/Mac
```bash
TOKEN="YOUR_TOKEN_HERE"

curl -X POST \
  -H "Content-Type: application/json" \
  -d "{\"action\":\"save\",\"type\":\"calc\",\"token\":\"$TOKEN\",\"record\":{\"tank_type\":\"horizontal\",\"full_inches\":\"48\",\"start_inches\":\"12\",\"end_inches\":\"36\",\"capacity\":\"500\",\"capacity_unit\":\"gal\",\"gas\":\"LIN\",\"flow_value\":\"25\",\"timer_elapsed\":\"00:15:30\",\"result_text\":\"Transferred 387.5 gallons (2612 SCF)\"}}" \
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

**Expected Response:**
```json
{
  "ok": true,
  "status": 200
}
```

**Verify:** Check your Google Sheet → `calc_records` tab. You should see a new row.

---

## 4. Save Split Record

**Save a split calculation:**

### Windows (CMD)
```cmd
curl -X POST ^
  -H "Content-Type: application/json" ^
  -d "{\"action\":\"save\",\"type\":\"split\",\"token\":\"YOUR_TOKEN_HERE\",\"record\":{\"source_tank\":\"Truck A\",\"destination\":\"Tank 1, Tank 2\",\"amount_scf\":\"5000\",\"gas\":\"LOX\",\"notes\":\"Split between two storage tanks\",\"result_text\":\"Tank 1: 3000 SCF, Tank 2: 2000 SCF\"}}" ^
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

### Linux/Mac
```bash
TOKEN="YOUR_TOKEN_HERE"

curl -X POST \
  -H "Content-Type: application/json" \
  -d "{\"action\":\"save\",\"type\":\"split\",\"token\":\"$TOKEN\",\"record\":{\"source_tank\":\"Truck A\",\"destination\":\"Tank 1, Tank 2\",\"amount_scf\":\"5000\",\"gas\":\"LOX\",\"notes\":\"Split between two storage tanks\",\"result_text\":\"Tank 1: 3000 SCF, Tank 2: 2000 SCF\"}}" \
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

**Verify:** Check `split_records` tab in Google Sheet.

---

## 5. Save Reserve Record

**Save a reserve check:**

### Windows (CMD)
```cmd
curl -X POST ^
  -H "Content-Type: application/json" ^
  -d "{\"action\":\"save\",\"type\":\"reserve\",\"token\":\"YOUR_TOKEN_HERE\",\"record\":{\"reserve_amount\":\"1000\",\"gas\":\"LAR\",\"remaining\":\"500\",\"warning_flag\":\"true\",\"result_text\":\"WARNING: Reserve too low (500 SCF remaining)\"}}" ^
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

### Linux/Mac
```bash
TOKEN="YOUR_TOKEN_HERE"

curl -X POST \
  -H "Content-Type: application/json" \
  -d "{\"action\":\"save\",\"type\":\"reserve\",\"token\":\"$TOKEN\",\"record\":{\"reserve_amount\":\"1000\",\"gas\":\"LAR\",\"remaining\":\"500\",\"warning_flag\":\"true\",\"result_text\":\"WARNING: Reserve too low (500 SCF remaining)\"}}" \
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

**Verify:** Check `reserve_records` tab in Google Sheet.

---

## 6. Fetch All Records

**Retrieve all records for the logged-in user:**

### Windows (CMD)
```cmd
curl -X POST ^
  -H "Content-Type: application/json" ^
  -d "{\"action\":\"fetch\",\"type\":\"all\",\"token\":\"YOUR_TOKEN_HERE\",\"limit\":100}" ^
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

### Windows (PowerShell)
```powershell
$token = "YOUR_TOKEN_HERE"
curl -Method POST `
  -Headers @{"Content-Type"="application/json"} `
  -Body "{`"action`":`"fetch`",`"type`":`"all`",`"token`":`"$token`",`"limit`":100}" `
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

### Linux/Mac
```bash
TOKEN="YOUR_TOKEN_HERE"

curl -X POST \
  -H "Content-Type: application/json" \
  -d "{\"action\":\"fetch\",\"type\":\"all\",\"token\":\"$TOKEN\",\"limit\":100}" \
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

**Expected Response:**
```json
{
  "ok": true,
  "data": {
    "calc": {
      "ok": true,
      "records": [
        {
          "record_id": "abc123...",
          "user_id": "xyz789...",
          "timestamp": "2025-12-19T12:30:00.000Z",
          "tank_type": "horizontal",
          "gas": "LIN",
          "result_text": "Transferred 387.5 gallons (2612 SCF)"
        }
      ]
    },
    "split": {
      "ok": true,
      "records": [...]
    },
    "reserve": {
      "ok": true,
      "records": [...]
    }
  },
  "status": 200
}
```

---

## 7. Fetch Only Calculator Records

**Retrieve only calc records:**

### Linux/Mac
```bash
TOKEN="YOUR_TOKEN_HERE"

curl -X POST \
  -H "Content-Type: application/json" \
  -d "{\"action\":\"fetch\",\"type\":\"calc\",\"token\":\"$TOKEN\",\"limit\":50}" \
  "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"
```

**Types:** `calc`, `split`, `reserve`, or `all`

---

## Quick Testing Script (PowerShell)

Save this as `test-api.ps1`:

```powershell
# API URL
$API = "https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"

# 1. Test ping
Write-Host "`n=== Testing Ping ===" -ForegroundColor Cyan
curl "$API?action=ping" | ConvertFrom-Json | ConvertTo-Json

# 2. Login
Write-Host "`n=== Testing Login ===" -ForegroundColor Cyan
$loginBody = @{
    action = "login"
    email = "test@example.com"
    password = "123456"
} | ConvertTo-Json

$loginResponse = curl -Method POST -Headers @{"Content-Type"="application/json"} -Body $loginBody $API | ConvertFrom-Json
$loginResponse | ConvertTo-Json
$token = $loginResponse.auth.token

# 3. Save calc record
Write-Host "`n=== Saving Calc Record ===" -ForegroundColor Cyan
$saveBody = @{
    action = "save"
    type = "calc"
    token = $token
    record = @{
        tank_type = "horizontal"
        gas = "LIN"
        capacity = "500"
        result_text = "Test calculation from PowerShell"
    }
} | ConvertTo-Json

curl -Method POST -Headers @{"Content-Type"="application/json"} -Body $saveBody $API | ConvertFrom-Json | ConvertTo-Json

# 4. Fetch records
Write-Host "`n=== Fetching All Records ===" -ForegroundColor Cyan
$fetchBody = @{
    action = "fetch"
    type = "all"
    token = $token
    limit = 10
} | ConvertTo-Json

curl -Method POST -Headers @{"Content-Type"="application/json"} -Body $fetchBody $API | ConvertFrom-Json | ConvertTo-Json

Write-Host "`n=== Tests Complete ===" -ForegroundColor Green
```

**Run with:** `powershell -File test-api.ps1`

---

## Quick Testing Script (Bash)

Save this as `test-api.sh`:

```bash
#!/bin/bash

# API URL
API="https://script.google.com/macros/s/AKfycbwbIBbIlxnjfL4sZAJx_RxkjyNDa_QwXxTEfuKKXhUiHneKeVU3K8P50GWrsZqewS9Z/exec"

echo -e "\n=== Testing Ping ==="
curl -s "$API?action=ping" | jq

echo -e "\n=== Testing Login ==="
LOGIN_RESPONSE=$(curl -s -X POST \
  -H "Content-Type: application/json" \
  -d '{"action":"login","email":"test@example.com","password":"123456"}' \
  "$API")
echo "$LOGIN_RESPONSE" | jq

TOKEN=$(echo "$LOGIN_RESPONSE" | jq -r '.auth.token')

echo -e "\n=== Saving Calc Record ==="
curl -s -X POST \
  -H "Content-Type: application/json" \
  -d "{\"action\":\"save\",\"type\":\"calc\",\"token\":\"$TOKEN\",\"record\":{\"tank_type\":\"horizontal\",\"gas\":\"LIN\",\"result_text\":\"Test from bash\"}}" \
  "$API" | jq

echo -e "\n=== Fetching All Records ==="
curl -s -X POST \
  -H "Content-Type: application/json" \
  -d "{\"action\":\"fetch\",\"type\":\"all\",\"token\":\"$TOKEN\",\"limit\":10}" \
  "$API" | jq

echo -e "\n=== Tests Complete ==="
```

**Run with:** `bash test-api.sh`

---

## Error Responses

### Invalid Credentials
```json
{
  "ok": false,
  "error": "Invalid credentials",
  "status": 401
}
```

### Missing Token
```json
{
  "ok": false,
  "error": "Missing token",
  "status": 401
}
```

### Token Expired
```json
{
  "ok": false,
  "error": "Token expired",
  "status": 401
}
```

### Invalid Action
```json
{
  "ok": false,
  "error": "Invalid action",
  "status": 400
}
```

---

## Testing Checklist

- [ ] Ping endpoint returns `"ok": true`
- [ ] Login returns a token
- [ ] Save calc record succeeds (check Google Sheet)
- [ ] Save split record succeeds
- [ ] Save reserve record succeeds
- [ ] Fetch returns saved records
- [ ] Invalid credentials return error
- [ ] Missing token returns error

---

## Next Steps

After confirming all endpoints work:

1. **Update `records.html`** with your API URL
2. **Test in browser** - http://localhost:8000
3. **Connect frontend login** to use real authentication
4. **Deploy to production** hosting

Your API is ready! 🚀
