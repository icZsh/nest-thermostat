# Nest Thermostat Control

Control Google Nest thermostats via the Smart Device Management (SDM) API with a clean Python client.

## Features

- OAuth 2.0 authentication with automatic token refresh
- List devices and view thermostat status
- Set temperature, mode, and eco settings
- Async HTTP client for efficient API calls
- No external dependencies beyond `httpx`

## Prerequisites

- Google Cloud project with SDM API enabled
- Device Access Console project ([$5 one-time fee](https://console.nest.google.com/device-access))
- OAuth 2.0 Web application credentials

## Setup

### 1. Google Cloud + Device Access

1. Enable the SDM API at [console.cloud.google.com](https://console.cloud.google.com)
2. Create OAuth 2.0 Web credentials (redirect URI: `http://localhost:8000/nest/callback`)
3. Create a Device Access project at [console.nest.google.com/device-access](https://console.nest.google.com/device-access) ($5 fee)
4. Link your OAuth client to the Device Access project

### 2. Install

```bash
pip install httpx
```

### 3. Configure

```bash
mkdir -p ~/.config/nest
touch ~/.config/nest/.env
```

Add to `~/.config/nest/.env`:

```
NEST_CLIENT_ID=your-client-id.apps.googleusercontent.com
NEST_CLIENT_SECRET=your-client-secret
NEST_SDM_PROJECT_ID=your-device-access-project-id
```

### 4. Authorize (one-time)

```bash
python3 nest_google_sdm.py oauth
```

Open the printed URL in your browser. The callback is handled automatically.

## Usage

```bash
# List devices
python3 nest_google_sdm.py devices

# View thermostat status
python3 nest_google_sdm.py status

# Set temperature (auto-detects mode: HEAT or COOL)
python3 nest_google_sdm.py set-temp 72

# Change mode
python3 nest_google_sdm.py set-mode HEAT
python3 nest_google_sdm.py set-mode COOL

# Eco mode
python3 nest_google_sdm.py set-eco on
python3 nest_google_sdm.py set-eco off

# Full help
python3 nest_google_sdm.py help
```

## API Commands Reference

| Command | SDM Command | Notes |
|---|---|---|
| `set-mode HEAT` | `sdm.devices.commands.ThermostatMode.SetMode` | |
| `set-mode COOL` | `sdm.devices.commands.ThermostatMode.SetMode` | |
| `set-mode OFF` | `sdm.devices.commands.ThermostatMode.SetMode` | |
| `set-temp 72` | `SetHeat` / `SetCool` | Auto-detects current mode |
| `set-eco on` | `sdm.devices.commands.ThermostatEco.SetMode` | MANUAL_ECO |

## Project Structure

```
nest_google_sdm.py   # CLI client + OAuth server (single file)
SKILL.md             # Hermes skill documentation
```

## Troubleshooting

**Token refresh fails with `invalid_grant`**
: The refresh token was revoked. Re-run `python3 nest_google_sdm.py oauth` to re-authorize.

**Terminal truncates long tokens**
: Tokens are always saved to `~/.config/nest/.env`. Never copy tokens from terminal output.

**Auth code already used error**
: The authorization code is single-use. Only call the token endpoint once per code.

## License

MIT
