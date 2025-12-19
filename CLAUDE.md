# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Cryo Tank Calculator** is a Progressive Web App (PWA) for cryogenic transfer operations. It helps drivers and technicians calculate liquid transfer amounts for LIN (Nitrogen), LOX (Oxygen), and LAR (Argon) deliveries with precision, using tank geometry and flow measurements.

The app is designed for:
- Offline-first operation with service worker caching
- Mobile-first UI with iOS support (viewport safe areas, home screen icons)
- Three main calculation modes via horizontal swipe navigation:
  1. **Calculator** - Tank transfer calculations using inches and flow
  2. **Split** - Splitting SCF loads between multiple destination tanks
  3. **Reserve** - Checking if tanks have sufficient reserve for deliveries

## Repository Structure

The app files are in `cryo-calculator-dev-files/` subdirectory; documentation files are at root level.

```
cryo/
├── cryo-calculator-dev-files/         - Main application folder
│   ├── index.html                     - Splash screen + login modal wrapper
│   ├── calculator.html                - Main app with 3 swipeable panes
│   ├── records.html                   - History view for saved records
│   ├── constant.js                    - Shared API endpoint (not yet imported by HTML files)
│   ├── sw.js                          - Service worker for offline/caching
│   ├── manifest.webmanifest.txt       - PWA manifest (rename to .webmanifest for deployment)
│   └── _headers.txt                   - Security headers (rename to _headers for deployment)
└── DEV DOCS/                          - Development documentation and drafts
```

## Architecture

### Application Flow

1. **Entry Point**: `index.html` shows splash screen for 3s, then loads `calculator.html` in an iframe
2. **Authentication**: Modal overlay prompts for email + password
   - Stored in `localStorage.cryo_auth` as `{user_id, email, role, token, exp}`
   - Token expires after 7 days (configurable in backend)
3. **Main App**: `calculator.html` contains all three calculators in a single file with swipe navigation
4. **Data Persistence**:
   - Local-first: All records saved to localStorage immediately
   - Cloud sync: Optional Google Sheets backend via Apps Script Web App
5. **Offline Support**: Service worker caches all assets for offline operation

### Navigation System

The app uses a custom swipe-based navigation with three horizontal panes:
- Transform-based sliding: `translateX(-100vw)` for each pane shift
- Swipe gesture detection: Touch events with minimum 50px threshold
- Page indicators: Dots at bottom show current position (calculator.html:82-97)
- Deep linking: `goToCalc()`, `goToSplit()`, `goToReserve()` functions for programmatic navigation (calculator.html:2892-2894)
- Implementation: Touch handlers at calculator.html:2905+ handle `touchstart`, `touchmove`, `touchend` events

### Key Calculation Logic (in calculator.html)

**Gas Constants (calculator.html:2124-2128):**
```javascript
LIN: { scfPerGal: 93.1,  lbPerGal: 6.74, scfPerLb: 13.8  }  // Nitrogen
LOX: { scfPerGal: 115.0, lbPerGal: 9.52, scfPerLb: 12.06 }  // Oxygen
LAR: { scfPerGal: 87.3,  lbPerGal: 9.4,  scfPerLb: 9.3   }  // Argon
```

**Tank Geometry:**
- Supports horizontal cylindrical and vertical cylindrical tanks
- Converts "full trycock inches" (vertical measurement) to "laydown inches" (horizontal position in cylinder)
- Uses cylinder diameter to map vertical height to volume
- Formulas account for tank orientation and end-cap geometry

**Flow Calculations:**
- Supports gallons/min or pounds/min input
- Timer-based transfer estimation
- Option to use timer as primary (instead of inches) for total volume

**Split Mode:**
- Requires total SCF from truck
- Distributes load across multiple destination tanks
- Validates that sum of splits matches total SCF

## Backend Integration

### Google Sheets Apps Script

The backend is a single `Code.gs` file deployed as a Web App. See `GOOGLE_APPS_SCRIPT_SETUP.md` for complete setup instructions.

**Endpoints:**
- `GET ?action=ping` - Health check
- `POST {action:"login", email, password}` - JWT-style token authentication
- `POST {action:"save", type, token, record}` - Save calc/split/reserve records
- `POST {action:"fetch", type, token, limit}` - Retrieve user's records

**Sheet Structure:**
- `users` - user_id, email, password_hash (SHA-256), role, active, created_at
- `calc_records` - Calculation history with all input parameters + results
- `split_records` - Split delivery records
- `reserve_records` - Reserve check records
- `audit_log` (optional) - Event logging

**Security:**
- HMAC-signed tokens (HS256-style, custom implementation)
- Password hashing with SHA-256
- Token expiry (default 7 days)
- Per-user record isolation

### Client-Side Integration

**API Endpoint Configuration:**

The API endpoint is hardcoded in three files (and also in `constant.js` which is not yet imported):

- `index.html:114` - `API_ENDPOINT` constant
- `calculator.html:2131` - `API_ENDPOINT` constant
- `records.html:153` - `API_ENDPOINT` constant
- `constant.js:1` - Shared constant file (created but not yet used)

**Auth Storage:**
```javascript
localStorage.getItem("cryo_auth")
// → {user_id, email, role, token, exp}
```

**Save Pattern:**
1. Save to localStorage immediately (offline-safe)
2. If online + authenticated, POST to backend
3. Mark record as `source: "sync"` on success

## Development Workflow

### Local Development
```bash
cd /Users/xlmb02/Downloads/cryo
python -m http.server 8000
# Open http://localhost:8000/cryo-calculator-dev-files/
```

The service worker requires HTTPS in production but works on `localhost`.

### Offline Testing
DevTools → Application → Service Workers → check "Offline" → reload page

### Deployment

**Before deploying:**
1. Rename `manifest.webmanifest.txt` → `manifest.webmanifest`
2. Rename `_headers.txt` → `_headers` (for Netlify/Cloudflare)
3. Increment cache versions in sw.js:2-3
4. Update `API_ENDPOINT` in all files

**Backend Setup:** See `GOOGLE_APPS_SCRIPT_SETUP.md`

## Important Patterns

### Authentication Flow
- **Guest mode**: Can use calculator, but Save/History buttons are disabled
- **Logged in**: Full access to Save + History features
- **Login check**: `getAuth()` returns null if no auth or expired
- **No server session required**: Stateless JWT-style tokens

### Record Structure
All saved records include:
- `record_id` (UUID, server-generated)
- `user_id` (from auth token)
- `timestamp` (ISO 8601)
- `source` ("local" or "sync")
- Type-specific fields (tank params, gas, flow, results, etc.)

### Service Worker Updates

When updating `sw.js`:
1. Increment **both** `CACHE_STATIC` and `CACHE_RUNTIME` version numbers (sw.js:2-3)
2. Old caches are automatically cleaned on activate (sw.js:31-42)
3. New SW installs immediately via `skipWaiting()` and claims clients (sw.js:27, 41)

Cache strategy (sw.js:48-91):
- **HTML navigations**: Network-first, fallback to cached index for offline (sw.js:52-64)
- **Same-origin static assets**: Stale-while-revalidate (sw.js:67-86)
- **Cross-origin**: Pass through (sw.js:90)

### Swipe Navigation Implementation
- Uses CSS transforms for 60fps performance
- Panes are 100vw wide, container is 300vw (calculator.html:34-44)
- Touch events: `touchstart` → `touchmove` → `touchend`
- Threshold: 50px minimum swipe distance
- Velocity detection for flick gestures
- Prevents default to avoid browser back/forward navigation

## Common Development Tasks

### Add a new gas type
1. Update gas constants object in calculator.html:2124-2128
2. Add new `<option>` to `<select id="gas">` dropdown
3. Update Apps Script backend if storing gas-specific fields

### Change cache strategy
Edit `sw.js` fetch handler (sw.js:48-91):
- Currently: stale-while-revalidate for assets, network-first for HTML
- Modify `event.respondWith()` logic for different strategies

### Add new calculation page
1. Add new pane to `.panes` div in calculator.html
2. Update swipe navigation max index check
3. Add dot indicator to hint section
4. Create `goToNewPage()` helper function (follow pattern at calculator.html:2892-2894)

### Update API endpoint (for new backend deployment)
Search and replace in all files with the endpoint:
```bash
grep -rn "API_ENDPOINT" cryo-calculator-dev-files/
# Update: index.html:114, calculator.html:2131, records.html:153, constant.js:1
```

### Modify login requirements
- Client validation: `index.html:175-182` (login modal)
- Backend logic: `handleLogin_()` in Apps Script Code.gs
- Current: email + password validation

### Increment service worker version (for cache busting)
Edit sw.js:2-3 and increment the version number (currently v6).

## File References

Key file locations for common edits:

- **Gas constants**: calculator.html:2124-2128
- **API endpoints**: index.html:114, calculator.html:2131, records.html:153, constant.js:1
- **Save function**: calculator.html:2142+
- **Swipe navigation**: calculator.html:2892-2894 (helpers), 2905+ (touch handlers)
- **Service worker cache versions**: sw.js:2-3
- **Login modal**: index.html
- **Auth storage**: index.html:125-130, calculator.html:2133-2139
