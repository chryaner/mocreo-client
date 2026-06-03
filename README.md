# MOCREO API Key Manager

> **Not affiliated with MOCREO.** I'm not connected to MOCREO in any way - just someone who
> wanted to integrate with their API and built this to make it easier.

A single, self-contained HTML page for managing [MOCREO](https://mocreo.com) API keys. No
backend and no build step; it calls the public MOCREO REST API at `https://api.mocreo.com`
directly from the browser.

It's one HTML file on purpose. This is a small tool for quick, occasional use, so there's no
framework, no build, and nothing to install - just a file you can open and read top to bottom.

## TL;DR

MOCREO provides sensors and an API, but no interface for creating the API keys you need to read
your data. This page covers that. Use it live at
[chryaner.github.io/mocreo-client](https://chryaner.github.io/mocreo-client/) or open
`index.html` locally, then log in and manage your keys.

## What it does

- Log in with your MOCREO account. Tokens are kept in memory only, not in localStorage or disk.
- Browse your assets and the sensors under each one.
- Create, list, and delete API keys per asset. A new key is shown only once, with a copy button,
  so you'll want to copy it before closing the dialog.
- Copy the Asset ID and per-sensor Device IDs for use in another tool.
- Test a key by fetching a device's latest values - the temperature reading, plus everything
  else it reports (humidity, battery, signal, and a hub's network info).

## Quick start

No dependencies, no build. Pick whichever is convenient:

- **Open directly** - double-click `index.html` (or open it in a browser).
- **Serve locally** - from the project folder:
  ```sh
  python -m http.server 8000
  # then visit http://localhost:8000
  ```

## How it works

1. **Login** - `POST /v1/users/login` returns a short-lived bearer token (held in memory).
2. **Dashboard** - three cards:
   - **Your Assets** - pick an asset to load its sensors and API keys, and generate new keys.
   - **Integration Values** - copyable Asset ID and Device IDs for an external tool.
   - **Test an API Key** - paste a key, pick an asset/device, and see the latest reading.

## MOCREO API reference

Base URL: `https://api.mocreo.com`. All responses use the envelope
`{ success, result, messages, errors }`; errors arrive as `errors: [{ message, code }]`.

The API uses two different auth schemes depending on what you're doing:

| Context                        | Header                                 |
|--------------------------------|----------------------------------------|
| Account session (after login)  | `Authorization: Bearer <access_token>` |
| API key (data access)          | `X-API-Key: <full_api_key>`            |

| Action            | Request                                                                          |
|-------------------|----------------------------------------------------------------------------------|
| Login             | `POST /v1/users/login` - `{ email, password }` → `result.access_token`           |
| List assets       | `GET /v1/assets`                                                                 |
| List devices      | `GET /v1/assets/{assetId}/devices`                                               |
| List API keys     | `GET /v1/assets/{assetId}/apikeys`                                               |
| Create API key    | `POST /v1/assets/{assetId}/apikeys` - see body below                             |
| Delete API key    | `DELETE /v1/assets/{assetId}/apikeys/{prefix}`                                   |
| Device history    | `GET /v1/assets/{assetId}/devices/{deviceId}/history` - uses `X-API-Key`         |
| Device details    | `GET /v1/assets/{assetId}/devices/{deviceId}` - uses `X-API-Key`                 |

**Create key body** - all three fields are required:

```json
{
  "displayName": "My Integration Key",
  "permissions": ["device.read"],
  "expiresAt": "2027-01-01T00:00:00.000Z"
}
```

The response's `result.key` (prefixed `mok_`) is the **full key, returned only once** - it can
never be retrieved again. Keys are referenced by their `prefix` for deletion.

**Device history** uses the `X-API-Key` header (not bearer auth) and these query params:
`field=temperature`, `from=<unix_ms>`, `to=<unix_ms>`, `limit=1`, `tz=<IANA timezone>`.

**Device details** uses the `X-API-Key` header (not bearer auth). The device's `properties` hold
the latest values: `temperature` is in hundredths of a degree Celsius (e.g. `2300` = 23.0 °C).
"Test a key" reads it, divides by 100, and converts to the asset's display unit
(`config.units.temperature`). For the timestamp it uses `attributes.lastOnline` (when the device
was last seen, in seconds) rather than `updatedAt` (which only changes when the value changes).

## Getting your sensors online

To have data to manage keys for, set up the hardware via the MOCREO Smart app
([mocreo.com/download](https://mocreo.com/download/)): create an account, power on and pair the
H5Pro hub, connect it to Wi-Fi, then activate and pair each temperature sensor. Once sensors
report to the cloud, they show up here under their asset.

This tool was built and tested with a MOCREO H5Pro hub and its temperature sensors. It should
work with any setup the same API exposes, but that's the hardware it's been verified against.

## License

Licensed under the [MIT License](LICENSE).
