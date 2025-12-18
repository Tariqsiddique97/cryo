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

## Architecture

### Core Files Structure

```
cryo-calculator-dev-files/
├── index.html           - Splash screen + login modal wrapper (loads calculator.html in iframe)
├── calculator.html      - Main app with 3 swipeable panes (Calculator, Split, Reserve)
├── records.html         - History view for saved records
├── login.html          - Simple launch page (rarely used, redirects to index)
├── sw.js               - Service worker for offline/caching
├── manifest.webmanifest.txt - PWA manifest
├── _headers.txt        - Security headers for deployment
└── icons/              - PWA icons (192, 512, maskable, apple-touch)

DEV DOCS/
├── Apps Script Code.txt       - Google Sheets backend (Code.gs)
├── shared JS file.txt         - Client-side API integration patterns
├── Save Buttons.txt           - Integration examples
└── google email script.txt    - Email notification setup
```

### Application Flow

1. **Entry Point**: `index.html` shows splash screen for 3s, then loads `calculator.html` in an iframe
2. **Authentication**: Modal overlay prompts for email + 6-digit PIN (stored in `localStorage.cryo_auth`)
3. **Main App**: `calculator.html` contains all three calculators in a single file with swipe navigation
4. **Data Persistence**:
   - Local-first: All records saved to localStorage immediately
   - Cloud sync: Optional Google Sheets backend via Apps Script Web App
5. **Offline Support**: Service worker caches all assets for offline operation

### Navigation System

The app uses a custom swipe-based navigation with three horizontal panes:
- Transform-based sliding: `translateX(-100vw)` for each pane
- Swipe gesture detection: Touch events with minimum 50px threshold
- Page indicators: Dots at bottom show current position
- Deep linking: `goToCalc()`, `goToSplit()`, `goToReserve()` functions for programmatic navigation

### Key Calculation Logic (in calculator.html)

**Tank Geometry:**
- Supports horizontal cylindrical and vertical cylindrical tanks
- Converts "full trycock inches" (vertical measurement) to "laydown inches" (horizontal position in cylinder)
- Uses cylinder diameter to map vertical height to volume
- Formulas account for tank orientation and end-cap geometry

**Gas Densities (used for SCF conversions):**
- LIN: 6.75 lb/gal
- LOX: 9.52 lb/gal
- LAR: 11.6 lb/gal

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

The backend is a single `Code.gs` file that provides:

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

**Configuration:**
```javascript
const SAVE_ENDPOINT = "PASTE_YOUR_APPS_SCRIPT_WEBAPP_URL_HERE";
```
This must be updated in both `calculator.html` and `records.html`.

**Auth Storage:**
```javascript
localStorage.getItem("cryo_auth")
// → {email, pin, ts} (simplified client)
// → {user_id, email, role, token, exp} (full backend)
```

**Save Pattern:**
1. Save to localStorage immediately (offline-safe)
2. If online + authenticated, POST to backend
3. Mark record as `source: "sync"` on success

## Development Workflow

### Testing Locally

**Start a local server:**
```bash
# Python
python -m http.server 8000

# Node.js (if http-server installed)
npx http-server -p 8000

# PHP
php -S localhost:8000
```

**Important:** The service worker requires HTTPS in production, but works on `localhost` for development.

**Test offline mode:**
1. Load app in browser
2. Open DevTools → Application → Service Workers
3. Check "Offline" checkbox
4. Reload page - should work from cache

### PWA Installation Testing

**iOS Safari:**
- Open in Safari (not Chrome/Firefox)
- Tap Share → Add to Home Screen
- Icon should appear with correct splash screen on launch

**Android Chrome:**
- Should show "Add to Home Screen" prompt automatically
- Or use Chrome menu → Install App

### Deployment

**Static Hosting (Netlify, Cloudflare Pages, etc.):**
1. Upload entire `cryo-calculator-dev-files/` folder
2. Ensure `_headers.txt` is renamed to `_headers` (check host requirements)
3. Manifest file should be renamed from `.txt` to actual `.webmanifest`
4. Update service worker cache version in `sw.js` on each deploy

**Backend Setup:**
1. Create Google Sheet with required tabs (users, calc_records, split_records, reserve_records)
2. Add header rows to each sheet (see Apps Script Code.txt for exact column names)
3. Extensions → Apps Script → paste Code.gs
4. Update `CONFIG.TOKEN_SECRET` to a long random string
5. Deploy as Web App (Execute as: Me, Access: Anyone with link)
6. Copy Web App URL → update `SAVE_ENDPOINT` in client files
7. Run `adminCreateUser(email, password, role)` in script editor to create first user

## Important Patterns

### Authentication Flow
- Guest mode: Can use calculator, but Save/History buttons are disabled
- Logged in: Full access to Save + History features
- Login check: `getAuth()` returns null if no auth or expired
- No server session required - stateless JWT-style tokens

### Record Structure
All saved records include:
- `record_id` (UUID, server-generated)
- `user_id` (from auth token)
- `timestamp` (ISO 8601)
- `source` ("local" or "sync")
- Type-specific fields (tank params, gas, flow, results, etc.)

### Service Worker Updates
When updating `sw.js`:
1. Increment both `CACHE_STATIC` and `CACHE_RUNTIME` version numbers
2. Old caches are automatically cleaned on activate
3. New SW installs immediately (`skipWaiting()`) and claims clients

### Swipe Navigation Implementation
- Uses CSS transforms for 60fps performance
- Panes are 100vw wide, container is 300vw
- Touch events: `touchstart` → `touchmove` → `touchend`
- Threshold: 50px minimum swipe distance
- Velocity detection for flick gestures
- Prevents default to avoid browser back/forward navigation

## Calculation Accuracy Notes

**Inches Measurement:**
- "Full trycock inches" = vertical measurement at full column
- Must be mapped to laydown position for horizontal cylinders
- Requires accurate tank diameter
- Checkbox option: "Estimate diameter from timer" (uses flow + time to back-calculate)

**Timer Mode:**
- Start/Stop timer built into calculator
- Can use as cross-check or as primary measurement
- "Use timer as primary" ignores inches for total, uses flow × time
- Timer format: HH:MM:SS

**Split Calculator Requirements:**
- Must know total SCF from source
- Each destination needs start/end inches + tank specs
- Warns if sum of splits ≠ total SCF
- All fields marked with * are required

## File Naming Conventions

Note: Several files have `.txt` extensions but are actually JSON/JS/manifest files:
- `manifest.webmanifest.txt` → should be `.webmanifest`
- `_headers.txt` → should be `_headers` (for Netlify/Cloudflare)

This appears to be for easy editing/viewing during development.

## Browser Compatibility

**Supported:**
- iOS Safari 12+ (primary target)
- Chrome/Edge 80+ (Desktop + Android)
- Firefox 75+

**Service Worker:**
- All modern browsers except iOS < 11.3

**Required Features:**
- CSS env() for safe-area-inset (notch support)
- localStorage (no fallback)
- Fetch API
- ES6+ (arrow functions, const/let, template literals)

## Common Tasks

**Add a new gas type:**
1. Add to `<select id="gas">` in calculator.html
2. Add density to `GAS_DENSITY` object
3. Update Apps Script sheet headers if storing gas-specific fields

**Change cache strategy:**
- Edit `sw.js` fetch handler (currently: stale-while-revalidate for assets, network-first for HTML)

**Add new calculation page:**
1. Add new pane to `.panes` div in calculator.html
2. Update swipe navigation max index
3. Add dot indicator
4. Create `goToNewPage()` helper function

**Modify login requirements:**
- Client validation in `index.html` login modal
- Backend logic in `handleLogin_()` in Apps Script
- Current: email + password (though UI shows PIN for simplicity)
