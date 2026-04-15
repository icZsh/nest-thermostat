# Nest Thermostat Control

A Hermes skill for controlling Google Nest thermostats via the SDM API.

## For Agents

### Installation

This skill lives at `~/.hermes/skills/nest-thermostat/`. If not present, install it:

```bash
mkdir -p ~/.hermes/skills
git clone https://github.com/icZsh/nest-thermostat.git ~/.hermes/skills/nest-thermostat
```

### Loading

Before use, load the skill to get the full documentation and context:

```
/skills view nest-thermostat
```

Or in a session, call `skill_view('nest-thermostat')`.

### Usage

After loading, the agent can execute the following commands via the `nest_google_sdm.py` CLI:

```bash
python3 ~/.hermes/skills/nest-thermostat/nest_google_sdm.py <command>
```

#### Commands

| Command | Description |
|---|---|
| `devices` | List all Nest devices |
| `status [idx]` | Show thermostat status (current temp, humidity, mode, HVAC) |
| `set-temp <F> [idx]` | Set target temperature (auto-detects HEAT/COOL mode) |
| `set-mode HEAT\|COOL\|OFF [idx]` | Change thermostat mode |
| `set-eco on\|off [idx]` | Toggle eco mode |
| `oauth` | Run one-time OAuth authorization (requires browser) |

### Configuration (required before first use)

Create `~/.config/nest/.env` with:

```
NEST_CLIENT_ID=<from Google Cloud OAuth credentials>
NEST_CLIENT_SECRET=<from Google Cloud OAuth credentials>
NEST_SDM_PROJECT_ID=<from Device Access Console>
NEST_ACCESS_TOKEN=<filled automatically after oauth>
NEST_REFRESH_TOKEN=<filled automatically after oauth>
```

#### One-time OAuth

```bash
python3 ~/.hermes/skills/nest-thermostat/nest_google_sdm.py oauth
```

Prints a URL to open in a browser. Callback auto-handled on `http://localhost:8000/nest/callback`.

### Cron Usage

For scheduled control, include these steps in the cron prompt:

```
1. Read access token: grep NEST_ACCESS_TOKEN ~/.hermes/.env
2. Read project ID: grep NEST_PROJECT_ID ~/.hermes/.env
3. Device name: enterprises/<project_id>/devices/<device_id>
```

Common API commands:

- **HEAT 75F**: POST `.../devices/{id}:executeCommand`
  Body: `{"command": "sdm.devices.commands.ThermostatMode.SetMode", "params": {"mode": "HEAT"}}`
  Then: `{"command": "sdm.devices.commands.ThermostatTemperatureSetpoint.SetHeat", "params": {"heatCelsius": 23.89}}`

- **OFF**: `{"command": "sdm.devices.commands.ThermostatMode.SetMode", "params": {"mode": "OFF"}}`

### Critical Pitfalls

1. **Auth code is single-use** — exchange + save in one atomic operation. Never call token endpoint twice with same code.
2. **Terminal truncates long tokens** — always read/write tokens from `.env` file, never from terminal echo.
3. **Refresh token fails with `invalid_grant`** — token was revoked. Re-run `oauth`.
4. **Hermes profile changes $HOME** — if running under a Hermes profile, credentials may be at `~/.hermes/.env` instead of `~/.config/nest/.env`.

## For Humans

See [SKILL.md](./SKILL.md) for full setup guide, API reference, and troubleshooting.
