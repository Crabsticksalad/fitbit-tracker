---
name: fitbit-tracker
description: Personal Fitbit integration for daily health tracking with adaptive sleep and activity reporting
version: 0.2.1
metadata:
  clawdbot:
    env:
      FITBIT_CLIENT_ID:
        description: Fitbit OAuth client ID
        required: true
      FITBIT_CLIENT_SECRET:
        description: Fitbit OAuth client secret
        required: true
      FITBIT_REDIRECT_URI:
        description: OAuth redirect URI (typically http://localhost:8080)
        required: true
      FITBIT_TZ:
        description: Timezone for date calculations (e.g. Europe/London)
        default: UTC
    requires:
      bins: [python3]
---

# Fitbit Tracker

Effortless health monitoring powered by the official Fitbit API. No apps, no dashboards — just ask and get your health stats instantly.

## Features

### Smart Sleep Tracking
- Automatically separates naps from main sleep
- Reports duration, efficiency, and all sleep stages (Deep, Light, REM, Wake)
- Nap detection with separate reporting

### Complete Activity Picture
- Steps, calories, distance, active minutes
- Resting heart rate
- Heart rate zones (Out of Range, Fat Burn, Cardio, Peak)

### Adaptive Reporting
Only shows what you ask for — no unnecessary data.

| You say... | Reports... |
|-------------|-----------|
| "how did I sleep" / "sleep" | Duration, efficiency, all stages, nap |
| "just my steps" | Steps only |
| "activity today" | Steps, calories, distance, active mins, HR zones |
| "full report" / "everything" / "summary" | All available data |
| "fitbit" / "health" | Complete daily summary |

### Clean Formatting
- Numbers formatted for readability (e.g., "8,234 steps")
- Stages grouped logically
- No raw data dumps

## Data Available

**Sleep:**
- Duration (total sleep time)
- Sleep efficiency %
- Sleep score (when available)
- Sleep stages: Deep, Light, REM, Wake
- Nap duration (when taken)

**Activity:**
- Steps
- Calories (total + BMR)
- Distance (km)
- Resting heart rate
- Active minutes (Very Active, Fairly, Lightly, Sedentary)
- Heart rate zones

## Setup

### 1. Create Fitbit Developer App

1. Go to [dev.fitbit.com](https://dev.fitbit.com)
2. Log in and click **Register an App**
3. Fill in:
   - **Application Name**: OpenClaw Fitbit (or any name)
   - **Description**: Fitbit integration for OpenClaw
   - **Application Website**: `https://github.com/yourusername/openclaw`
   - **OAuth 2.0 Application Type**: Choose **Personal**
   - **Callback URL**: `http://localhost:8080` (for local) or your redirect URI
4. Accept the terms and register
5. Copy your **Client ID** and **Client Secret**

### 2. Configure Environment Variables

```bash
export FITBIT_CLIENT_ID="your_client_id"
export FITBIT_CLIENT_SECRET="your_client_secret"
export FITBIT_REDIRECT_URI="http://localhost:8080"
export FITBIT_TZ="Europe/London"  # Your timezone
```

Or add to `~/.openclaw/.env`:
```
FITBIT_CLIENT_ID=your_client_id
FITBIT_CLIENT_SECRET=your_client_secret
FITBIT_REDIRECT_URI=http://localhost:8080
FITBIT_TZ=Europe/London
```

### 3. Authenticate

Run the OAuth login script:
```bash
python3 scripts/fitbit_oauth_login.py
```

This will:
- Open Fitbit authorization page in your browser
- Ask you to approve access
- Exchange the code for tokens
- Save tokens to `~/.config/openclaw/fitbit/token.json`

Tokens are automatically refreshed when they expire.

## Usage Examples

**Sleep report:**
```
Fitbit — 2026-03-21
- Sleep: 7h 32m (score 85) | 93% efficiency
  - Stages: Deep: 1h 42m, Light: 3h 20m, REM: 1h 45m, Wake: 45m
- Nap: 1h 6m
```

**Full daily summary:**
```
Fitbit — 2026-03-21
- Sleep: 7h 32m (score 85) | 93% efficiency
  - Stages: Deep: 1h 42m, Light: 3h 20m, REM: 1h 45m, Wake: 45m
- Nap: 1h 6m
- Steps: 8,234
- Calories: 1,892 (1,048 BMR)
- Distance: 6.2 km
- Resting HR: 58 bpm
  - Active mins: V. Active: 45m, Fair: 23m, Light: 1h 24m, Sedentary: 8h 12m
  - HR Zones: Out of Range: 12h, Fat Burn: 1h 30m, Cardio: 32m, Peak: 8m
```

**Steps only:**
```
- Steps: 8,234
```

## Python Dependencies

No third-party dependencies required. Uses Python standard library:
- `urllib.request` - HTTP requests
- `json` - JSON parsing
- `datetime` - Date handling
- `zoneinfo` - Timezone support (Python 3.9+)

## Troubleshooting

### "Missing env var: FITBIT_CLIENT_ID"
Environment variables not loaded. Source your `.env` file or ensure variables are set in the gateway environment:
```bash
source ~/.openclaw/.env
```

### "Token expired" error
Tokens auto-refresh. If you see this error repeatedly:
1. Delete `~/.config/openclaw/fitbit/token.json`
2. Re-run `python3 scripts/fitbit_oauth_login.py`

### "No data found for this day"
- Check `FITBIT_TZ` matches your timezone
- Try `--date yesterday` to confirm data exists
- Verify Fitbit account has data for the date

### Nap not separating from main sleep
This may indicate `isMainSleep` flag isn't set correctly on your Fitbit account. The skill filters records by this flag — some older Fitbit devices don't set it properly.

### Sleep score not showing
Not all Fitbit accounts/devices provide sleep scores. This is a Fitbit API limitation, not the skill. Efficiency and stages are always reported when available.

## API Endpoints Used

- `GET /1/user/-/activities/date/{date}.json` - Daily activity summary
- `GET /1.2/user/-/sleep/date/{date}.json` - Sleep records

## References

- API details: `references/fitbit_api.md`
- Output schema: `references/output_schema.md`
