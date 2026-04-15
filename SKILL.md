---
name: nest-thermostat
description: Control Google Nest thermostats via SDM API with a clean Python client and OAuth flow.
version: 1.0.0
author: hermes-minimax
license: MIT
metadata:
  hermes:
    tags: [smart-home, nest, thermostat, google-sdm]
prerequisites:
  commands: [python3.11]
  setup:
    - Google Cloud project with SDM API enabled
    - Device Access Console project ($5 one-time fee)
    - OAuth 2.0 Web credentials
---

# Nest Thermostat Control

Control Google Nest thermostats via the Smart Device Management (SDM) API.

## Setup

### 1. Google Cloud + Device Access (one-time)

1. Enable SDM API at console.cloud.google.com
2. Create OAuth 2.0 Web credentials (redirect URI: `http://localhost:8000/nest/callback`)
3. Create Device Access project at console.nest.google.com/device-access ($5 fee)
4. Link OAuth client ID to Device Access project

### 2. Install dependencies

```bash
# Only httpx is required — no FastAPI, no dotenv required
pip install httpx
```

### 3. Configure

```bash
mkdir -p ~/.config/nest
# Edit: ~/.config/nest/.env
# Required keys:
#   NEST_CLIENT_ID=your-client-id.apps.googleusercontent.com
#   NEST_CLIENT_SECRET=your-client-secret
#   NEST_SDM_PROJECT_ID=your-device-access-project-id
```

### 4. Authorize (one-time, browser required)

```bash
python3 nest_google_sdm.py oauth
# The script will print the URL — open it in your browser manually
# Callback is caught automatically on the same port
# Tokens auto-saved to ~/.config/nest/.env
```

---

## Usage

```bash
# List devices
python3 nest_google_sdm.py devices

# View thermostat status
python3 nest_google_sdm.py status
python3 nest_google_sdm.py status 0    # specific thermostat

# Set temperature (auto-detects mode)
python3 nest_google_sdm.py set-temp 72
python3 nest_google_sdm.py set-temp 68 1

# Change mode
python3 nest_google_sdm.py set-mode HEAT
python3 nest_google_sdm.py set-mode COOL 0

# Eco mode
python3 nest_google_sdm.py set-eco on
python3 nest_google_sdm.py set-eco off 0
```

---

## Isaac's Setup (San Jose, PDT)

- Credentials: `/Users/isaaczhu/.hermes/.env` (NOT ~/.config/nest/.env — Hermes profile changes $HOME)
- Python: `/Users/isaaczhu/.hermes/hermes-agent/venv/bin/python3`
- Token manager: `/Users/isaaczhu/.hermes/hermes-agent/tools/nest_token_manager.py`
- Device ID (勺儿): `AVPHwEub9_C3M1NyDipFEQs18tGCr4cDjxCcwyF0nW3y3DURFh3CQ24mulF4opCBiJZ3lR6P72DCq_FmormMnvgW5oJZPg`
- Project ID: `1c07e101-6b09-4c4d-93b0-67a2577b41b9`
- Thermostat name: 勺儿 (room: 勺儿, structure: Abreviated structure ID)

## Critical Pitfalls

1. **Auth code is single-use**: When user provides a `code=`, immediately exchange + save in one atomic operation. Never call the token endpoint twice with the same code.
2. **Terminal truncates long tokens**: Always read/write tokens from files, never rely on terminal echo.
3. **Profile $HOME override**: Hermes profile changes `$HOME` to `~/.hermes/profiles/<profile>/home`. Always use absolute paths for credentials and scripts.
4. **Refresh token fails with `invalid_grant`**: Usually means the token was revoked. Requires re-authorization via browser.

## Cron Usage (for scheduled commands)

For cron jobs that control the thermostat, include these steps in the prompt:

```
1. Read access token: grep NEST_ACCESS_TOKEN /Users/isaaczhu/.hermes/.env
2. Read project ID: grep NEST_PROJECT_ID /Users/isaaczhu/.hermes/.env
3. Device name: enterprises/1c07e101-6b09-4c4d-93b0-67a2577b41b9/devices/AVPHwEub9_C3M1NyDipFEQs18tGCr4cDjxCcwyF0nW3y3DURFh3CQ24mulF4opCBiJZ3lR6P72DCq_FmormMnvgW5oJZPg
```

Common commands:
- **HEAT mode**: POST .../devices/{id}:executeCommand  Body: {"command": "sdm.devices.commands.ThermostatMode.SetMode", "params": {"mode": "HEAT"}}
- **OFF**: {"command": "sdm.devices.commands.ThermostatMode.SetMode", "params": {"mode": "OFF"}}
- **Set 75°F**: {"command": "sdm.devices.commands.ThermostatTemperatureSetpoint.SetHeat", "params": {"heatCelsius": 23.89}}

## Architecture

```
~/.config/nest/nest_google_sdm.py  # Full-featured CLI with OAuth flow
scripts/nest_token_manager.py        # Auto token refresh (for cron jobs)

Key improvements over upstream `nest-sdm-control`:
- Single self-contained file (no duplicate code)
- httpx.AsyncClient reused per-process (not rebuilt per request)
- Parallel device info fetching with asyncio.gather
- Lazy token initialization (works with refresh_token only on first run)
- All null-safety handled gracefully
- No FastAPI/uvicorn dependency (plain http.server + threading)
- Config stored in ~/.config/nest/ (not cwd)
